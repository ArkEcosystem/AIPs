---
  AIP: 29
  Title: Generic Transaction Interface
  Authors: *Brian Faust, Kristjan Kosic, Joshua Noack*
  Status: *Draft*
  Discussions-To: https://github.com/ArkEcosystem/AIPs/issues/64
  Type: *Standards Track*
  Category: Core
  Created: *2019-01-21*
  Last Update: *2019-08-07*
---

## Abstract

This AIP proposes improvements to the structure and expandability of transaction types within Core to solve issues that are present because the initial implementation of the system that make it tedious to implement new transaction types.

## Motivation

At the moment all logic that relates to how transactions are applied, serialised and deserialised is scattered all over the place.

The problem with that is that it makes it more tedious to implement new transaction types and leaves a lot of room for mistakes as logic is not shared and more has to be known about implementation specifics. Another issue is that the `transaction type` is stored as a 1 byte integer. Core already takes the types 0 - 10 for it’s own transaction types. As more and more custom transaction types are written we are bound to run out of numbers. Additionally, there’s no way to prevent collisions since two different plugins could register a completely different transaction type with the same number. Therefore there needs to be 1) more room for transaction types and 2) a way to group transaction types to make collisions less likely to occur.

## Specification

In order to reduce the amount of duplication and complexity that currently plagues anything transaction related we will implement a `Transaction` class that will make it possible to implement de/serialization logic in a single place while the actual transaction logic for modifying wallets, etc. will be provided by `TransactionHandlers`.

To increase the number of maximum transaction types and decrease the likelihood of collisions between plugins that provide transactions, Core will register it’s own transaction types inside a reserved `type group` which cannot be used by any plugin and thus no collision can occur here. However, we cannot completely prevent different plugins from choosing the same group and same type numbering. But it will be a lot less likely.

In other words, we will introduce a 4 bytes transaction type group and increase the transaction type from 1 byte to 2 bytes. Then Core will use both `typeGroup` and `type` to address transactions internally. All V2 transactions will be signed with `typeGroup` and `type`. Non-core transactions will have to provide the `typeGroup` otherwise Core will fall back to `typeGroup: 1`, which is the default Core group.

### Transaction

The `Transaction` abstract on a very basic level would look like the following.

```ts
export abstract class Transaction {
    public static type: number = undefined;
    public static typeGroup: number = undefined;
    public static key: string = undefined;

    protected static defaultStaticFee: BigNumber = BigNumber.ZERO;

    public abstract serialize(): ByteBuffer;
    public abstract deserialize(buf: ByteBuffer): void;

    public hasVendorField(): boolean {
        return false;
    }

    // This has to return an AJV schema to validate the transaction format
    public static getSchema(): TransactionSchema {
        throw new NotImplementedError();
    }
}
```

The `serialize` and `deserialize` methods would be called by another entity or handler that calls it when needed like the other Crypto SDKs do where we have handlers for specific transaction types. The Crypto SDK way of doing it doesn't fit into the JS SDK as it is tightly coupled to core and not designed purely for end-users users.

### TransactionHandler
The actual transaction logic is not defined in the `Transaction` class. Instead we are going to introduce `TransactionHandlers` in order to separate the `crypto` part from transaction logic.

A very basic `TransactionHandler` could look like the following:

```ts
export interface ITransactionHandler {
    getConstructor(): TransactionConstructor;

    dependencies(): ReadonlyArray<TransactionHandlerConstructor>;
    walletAttributes(): ReadonlyArray<string>;
    bootstrap(connection: Database.IConnection, walletManager: State.IWalletManager): Promise<void>;
    isActivated(): Promise<boolean>;

    dynamicFee(transaction: Interfaces.ITransaction, addonBytes: number, satoshiPerByte: number): Utils.BigNumber;

    throwIfCannotBeApplied(
        transaction: Interfaces.ITransaction,
        wallet: State.IWallet,
        databaseWalletManager: State.IWalletManager,
    ): Promise<void>;
    
    applyToSender(transaction: Interfaces.Transaction, wallet: Wallet): Promise<void>;
    applyToRecipient(transaction: Interfaces.Transaction, wallet: Wallet): Promise<void>;
    revertForSender(transaction: Interfaces.Transaction, wallet: Wallet): Promise<void>;
    revertForRecipient(transaction: Interfaces.Transaction, wallet: Wallet): Promise<void>;
    apply(transaction: Interfaces.Transaction, wallet: Wallet): Promise<void>;
    revert(transaction: Interfaces.Transaction, wallet: Wallet): Promise<void>;

    canEnterTransactionPool(data: Interfaces.ITransactionData, pool: TransactionPool.IConnection, processor: TransactionPool.IProcessor): Promise<boolean>;
    emitEvents(transaction: Interfaces.Transaction, emitter: EventEmitter): void;
}
```
A `TransactionHandler` will tell Core for example how a transaction type operates on wallets and what requirements must be met in order to enter the `TransactionPool`.

### Implementing Transaction Types
Implementing transaction types will be easier as their logic is isolated and boilerplate will be greatly reduced. A simple Transfer of type 0 could look like the following.

```ts
export class TransferTransaction extends Transaction {
    public static typeGroup: number = TransactionTypeGroup.Core;
    public static type: number = TransactionType.Transfer;
    public static key: string = "transfer";

    protected static defaultStaticFee: BigNumber = BigNumber.make("10000000");

    public static getSchema(): schemas.TransactionSchema {
        // This represents an AJV schema (https://github.com/epoberezkin/ajv)
        return schemas.transfer;
    }

    // Serializing only the type 0 specific parts
    public serialize(): ByteBuffer {
        const { data } = this;
        const buffer = new ByteBuffer(24, true);
        buffer.writeUint64(+new Bignum(data.amount).toFixed());
        buffer.writeUint32(data.expiration || 0);
        buffer.append(bs58check.decode(data.recipientId));

        return buffer;
    }

    // Deserializing only the type 0 specific parts
    public deserialize(buf: ByteBuffer): void {
        const { data } = this;
        data.amount = new Bignum(buf.readUint64().toString());
        data.expiration = buf.readUint32();
        data.recipientId = bs58check.encode(buf.readBytes(21).toBuffer());
    }
    public hasVendorField(): boolean {
        return true;
    }
}
```

To reduce boilerplate and ease the implementation of custom transaction handlers, a base `TransactionHandler` implementing the aforementioned interface will be provided. In it's most basic form an abstract `TransactionHandler` could look like the following:

```ts
export abstract class TransactionHandler implements ITransactionHandler {
    public abstract getConstructor(): Transactions.TransactionConstructor;

    public abstract dependencies(): ReadonlyArray<TransactionHandlerConstructor>;

    public abstract walletAttributes(): ReadonlyArray<string>;

    public abstract async bootstrap(
        connection: Database.IConnection,
        walletManager: State.IWalletManager,
    ): Promise<void>;

    public abstract async isActivated(): Promise<boolean>;

    public dynamicFee(
        transaction: Interfaces.ITransaction,
        addonBytes: number,
        satoshiPerByte: number,
    ): Utils.BigNumber {
        addonBytes = addonBytes || 0;

        if (satoshiPerByte <= 0) {
            satoshiPerByte = 1;
        }

        const transactionSizeInBytes: number = transaction.serialized.length / 2;
        return Utils.BigNumber.make(addonBytes + transactionSizeInBytes).times(satoshiPerByte);
    }

    // Common wallet logic for all transaction types
     public async throwIfCannotBeApplied(
        transaction: Interfaces.ITransaction,
        sender: State.IWallet,
        databaseWalletManager: State.IWalletManager,
    ): Promise<void> {
        const data: Interfaces.ITransactionData = transaction.data;

        if (Utils.isException(data)) {
            return;
        }

        if (data.version > 1 && data.nonce.isLessThanOrEqualTo(sender.nonce)) {
            throw new UnexpectedNonceError(data.nonce, sender.nonce, false);
        }

        if (
            sender.balance
                .minus(data.amount)
                .minus(data.fee)
                .isNegative()
        ) {
            throw new InsufficientBalanceError();
        }

        if (data.senderPublicKey !== sender.publicKey) {
            throw new SenderWalletMismatchError();
        }

        if (sender.hasSecondSignature()) {
            // Ensure the database wallet already has a 2nd signature, in case we checked a pool wallet.
            const dbSender: State.IWallet = databaseWalletManager.findByPublicKey(data.senderPublicKey);

            if (!dbSender.hasSecondSignature()) {
                throw new UnexpectedSecondSignatureError();
            }

            const secondPublicKey: string = dbSender.getAttribute("secondPublicKey");
            if (!Transactions.Verifier.verifySecondSignature(data, secondPublicKey)) {
                throw new InvalidSecondSignatureError();
            }
        } else if (data.secondSignature || data.signSignature) {
            const isException =
                Managers.configManager.get("network.name") === "devnet" &&
                Managers.configManager.getMilestone().ignoreInvalidSecondSignatureField;
            if (!isException) {
                throw new UnexpectedSecondSignatureError();
            }
        }

        if (sender.hasMultiSignature()) {
            // Ensure the database wallet already has a multi signature, in case we checked a pool wallet.
            const dbSender: State.IWallet = databaseWalletManager.findByPublicKey(transaction.data.senderPublicKey);

            if (!dbSender.hasMultiSignature()) {
                throw new UnexpectedMultiSignatureError();
            }

            if (!dbSender.verifySignatures(data, dbSender.getAttribute("multiSignature"))) {
                throw new InvalidMultiSignatureError();
            }
        } else if (transaction.type !== Enums.TransactionType.MultiSignature && transaction.data.signatures) {
            throw new UnexpectedMultiSignatureError();
        }
    }

    public async apply(transaction: Interfaces.ITransaction, walletManager: State.IWalletManager): Promise<void> {
        await this.applyToSender(transaction, walletManager);
        await this.applyToRecipient(transaction, walletManager);
    }

    public async revert(transaction: Interfaces.ITransaction, walletManager: State.IWalletManager): Promise<void> {
        await this.revertForSender(transaction, walletManager);
        await this.revertForRecipient(transaction, walletManager);
    }

    public async applyToSender(
        transaction: Interfaces.ITransaction,
        walletManager: State.IWalletManager,
    ): Promise<void> {
        const sender: State.IWallet = walletManager.findByPublicKey(transaction.data.senderPublicKey);
        const data: Interfaces.ITransactionData = transaction.data;

        if (Utils.isException(data)) {
            walletManager.logger.warn(`Transaction forcibly applied as an exception: ${transaction.id}.`);
        }

        await this.throwIfCannotBeApplied(transaction, sender, walletManager);

        if (data.version > 1) {
            if (!sender.nonce.plus(1).isEqualTo(data.nonce)) {
                throw new UnexpectedNonceError(data.nonce, sender.nonce, false);
            }

            sender.nonce = data.nonce;
        }

        const newBalance: Utils.BigNumber = sender.balance.minus(data.amount).minus(data.fee);

        if (process.env.CORE_ENV === "test") {
            assert(Utils.isException(transaction.data) || !newBalance.isNegative());
        } else {
            assert(!newBalance.isNegative());
        }

        sender.balance = newBalance;
    }

    public async revertForSender(
        transaction: Interfaces.ITransaction,
        walletManager: State.IWalletManager,
    ): Promise<void> {
        const sender: State.IWallet = walletManager.findByPublicKey(transaction.data.senderPublicKey);
        const data: Interfaces.ITransactionData = transaction.data;

        sender.balance = sender.balance.plus(data.amount).plus(data.fee);

        if (data.version > 1) {
            if (!sender.nonce.isEqualTo(data.nonce)) {
                throw new UnexpectedNonceError(data.nonce, sender.nonce, true);
            }

            sender.nonce = sender.nonce.minus(1);
        }
    }

    public abstract async applyToRecipient(
        transaction: Interfaces.ITransaction,
        walletManager: State.IWalletManager,
    ): Promise<void>;

    public abstract async revertForRecipient(
        transaction: Interfaces.ITransaction,
        walletManager: State.IWalletManager,
    ): Promise<void>;

    /**
     * Transaction Pool logic
     */
    public async canEnterTransactionPool(
        data: Interfaces.ITransactionData,
        pool: TransactionPool.IConnection,
        processor: TransactionPool.IProcessor,
    ): Promise<boolean> {
        processor.pushError(
            data,
            "ERR_UNSUPPORTED",
            `Invalidating transaction of unsupported type '${Enums.TransactionType[data.type]}'`,
        );

        return false;
    }

    protected async typeFromSenderAlreadyInPool(
        data: Interfaces.ITransactionData,
        pool: TransactionPool.IConnection,
        processor: TransactionPool.IProcessor,
    ): Promise<boolean> {
        const { senderPublicKey, type }: Interfaces.ITransactionData = data;

        if (await pool.senderHasTransactionsOfType(senderPublicKey, type)) {
            processor.pushError(
                data,
                "ERR_PENDING",
                `Sender ${senderPublicKey} already has a transaction of type '${Enums.TransactionType[type]}' in the pool`,
            );

            return true;
        }

        return false;
    }

    public emitEvents(transaction: Interfaces.ITransaction, emitter: EventEmitter.EventEmitter): void {}
}

```

Now, the implementation for a simple `TransferHandler` of type 0 based on the abstract `TransactionHandler` class could look like the following:

```ts
export class TransferTransactionHandler extends TransactionHandler {
    public getConstructor(): Transactions.TransactionConstructor {
        return Transactions.TransferTransaction;
    }

    public dependencies(): ReadonlyArray<TransactionHandlerConstructor> {
        return [];
    }

    public walletAttributes(): ReadonlyArray<string> {
        return [];
    }

    public async bootstrap(connection: Database.IConnection, walletManager: State.IWalletManager): Promise<void> {
        const transactions = await connection.transactionsRepository.getReceivedTransactions();

        for (const transaction of transactions) {
            const wallet = walletManager.findByAddress(transaction.recipientId);
            wallet.balance = wallet.balance.plus(transaction.amount);
        }
    }

    public async isActivated(): Promise<boolean> {
        return true;
    }

    // Type 0 specific wallet logic
    public async throwIfCannotBeApplied(
        transaction: Interfaces.ITransaction,
        sender: State.IWallet,
        databaseWalletManager: State.IWalletManager,
    ): Promise<void> {
        return super.throwIfCannotBeApplied(transaction, sender, databaseWalletManager);
    }

    public async applyToRecipient(
        transaction: Interfaces.ITransaction,
        walletManager: State.IWalletManager,
    ): Promise<void> {
        const recipient: State.IWallet = walletManager.findByAddress(transaction.data.recipientId);
        recipient.balance = recipient.balance.plus(transaction.data.amount);
    }

    public async revertForRecipient(
        transaction: Interfaces.ITransaction,
        walletManager: State.IWalletManager,
    ): Promise<void> {
        const recipient: State.IWallet = walletManager.findByAddress(transaction.data.recipientId);
        recipient.balance = recipient.balance.minus(transaction.data.amount);
    }

    public async canEnterTransactionPool(
        data: Interfaces.ITransactionData,
        pool: TransactionPool.IConnection,
        processor: TransactionPool.IProcessor,
    ): Promise<boolean> {
        if (!isRecipientOnActiveNetwork(data)) {
            processor.pushError(
                data,
                "ERR_INVALID_RECIPIENT",
                `Recipient ${data.recipientId} is not on the same network: ${Managers.configManager.get(
                    "network.pubKeyHash",
                )}`,
            );
            return false;
        }

        return true;
    }
}

```

If a custom type requires more flexibility it can implement the `ITransactionHandler` interface directly at the cost of more overhead.


### Registering Transaction Types

After we have created our custom transaction type and the accompaying transaction handler we need a way of exposing it to the crypto package and core. To do this we will introduce a `TransactionRegistry` class inside the `crypto` package and a similar class for transaction handlers called `TransactionHandlerRegistry` which both hold a few methods to guard against overwrites of core transaction types or already registered custom types.

```ts
type TransactionConstructor = typeof Transaction;

class TransactionRegistry {
    private readonly transactionTypes: Map<InternalTransactionType, TransactionConstructor> = new Map();

    constructor() {
        this.registerTransactionType(TransferTransaction);
        this.registerTransactionType(SecondSignatureRegistrationTransaction);
        this.registerTransactionType(DelegateRegistrationTransaction);
        this.registerTransactionType(VoteTransaction);
        this.registerTransactionType(MultiSignatureRegistrationTransaction);
        this.registerTransactionType(IpfsTransaction);
        this.registerTransactionType(MultiPaymentTransaction);
        this.registerTransactionType(DelegateResignationTransaction);
        this.registerTransactionType(HtlcLockTransaction);
        this.registerTransactionType(HtlcClaimTransaction);
        this.registerTransactionType(HtlcRefundTransaction);
    }

    public registerTransactionType(constructor: TransactionConstructor): void {
        const { typeGroup, type } = constructor;
        const internalType: InternalTransactionType = InternalTransactionType.from(type, typeGroup);
        if (this.transactionTypes.has(internalType)) {
            throw new TransactionAlreadyRegisteredError(constructor.name);
        }

        this.transactionTypes.set(internalType, constructor);
        this.updateSchemas(constructor);
    }

    public deregisterTransactionType(constructor: TransactionConstructor): void {
        const { typeGroup, type } = constructor;
        const internalType: InternalTransactionType = InternalTransactionType.from(type, typeGroup);

        if (!this.transactionTypes.has(internalType)) {
            throw new UnkownTransactionError(internalType.toString());
        }

        if (typeGroup === TransactionTypeGroup.Core) {
            throw new CoreTransactionTypeGroupImmutableError();
        }

        const schema = this.transactionTypes.get(internalType);
        this.updateSchemas(schema, true);
        this.transactionTypes.delete(internalType);
    }

    private updateSchemas(transaction: TransactionConstructor, remove?: boolean): void {
        validator.extendTransaction(transaction.getSchema(), remove);
    }
}
```

The accompanying registry for transaction handlers:
```ts
import { transactionHandlers } from "./handlers";
export type TransactionHandlerConstructor = new () => TransactionHandler;

class TransactionHandlerRegistry {
    private readonly registeredTransactionHandlers: Map<
        Transactions.InternalTransactionType,
        TransactionHandler
    > = new Map();

    private readonly knownWalletAttributes: Map<string, boolean> = new Map();

    constructor() {
        this.registerTransactionHandler(TransferTransactionHandler);
        this.registerTransactionHandler(SecondSignatureTransactionHandler);
        this.registerTransactionHandler(DelegateRegistrationTransactionHandler);
        this.registerTransactionHandler(VoteTransactionHandler);
        this.registerTransactionHandler(MultiSignatureTransactionHandler);
        this.registerTransactionHandler(IpfsTransactionHandler);
        this.registerTransactionHandler(MultiPaymentTransactionHandler);
        this.registerTransactionHandler(DelegateResignationTransactionHandler);
        this.registerTransactionHandler(HtlcLockTransactionHandler);
        this.registerTransactionHandler(HtlcClaimTransactionHandler);
        this.registerTransactionHandler(HtlcRefundTransactionHandler);
    }

    public get(type: number, typeGroup?: number): TransactionHandler {
        const internalType: Transactions.InternalTransactionType = Transactions.InternalTransactionType.from(
            type,
            typeGroup,
        );
        if (this.registeredTransactionHandlers.has(internalType)) {
            return this.registeredTransactionHandlers.get(internalType);
        }

        throw new InvalidTransactionTypeError(internalType.toString());
    }

    public async getActivatedTransactions(): Promise<TransactionHandler[]> {
        const activatedTransactions: TransactionHandler[] = [];

        for (const handler of this.registeredTransactionHandlers.values()) {
            if (await handler.isActivated()) {
                activatedTransactions.push(handler);
            }
        }

        return activatedTransactions;
    }

    public registerTransactionHandler(constructor: TransactionHandlerConstructor) {
        const service: TransactionHandler = new constructor();
        const transactionConstructor = service.getConstructor();
        const { typeGroup, type } = transactionConstructor;

        for (const dependency of service.dependencies()) {
            this.registerTransactionHandler(dependency);
        }

        const internalType: Transactions.InternalTransactionType = Transactions.InternalTransactionType.from(
            type,
            typeGroup,
        );
        if (this.registeredTransactionHandlers.has(internalType)) {
            return;
        }

        if (!(type in Enums.TransactionType)) {
            Transactions.TransactionRegistry.registerTransactionType(transactionConstructor);
        }

        const walletAttributes: ReadonlyArray<string> = service.walletAttributes();
        for (const attribute of walletAttributes) {
            assert(!this.knownWalletAttributes.has(attribute), `Wallet attribute is already known: ${attribute}`);
            this.knownWalletAttributes.set(attribute, true);
        }

        this.registeredTransactionHandlers.set(internalType, service);
    }

    public deregisterTransactionHandler(constructor: TransactionHandlerConstructor): void {
        const service: TransactionHandler = new constructor();
        const transactionConstructor = service.getConstructor();
        const { typeGroup, type } = transactionConstructor;

        if (typeGroup === Enums.TransactionTypeGroup.Core || typeGroup === undefined) {
            throw new Errors.CoreTransactionTypeGroupImmutableError();
        }

        const internalType: Transactions.InternalTransactionType = Transactions.InternalTransactionType.from(
            type,
            typeGroup,
        );
        if (!this.registeredTransactionHandlers.has(internalType)) {
            throw new InvalidTransactionTypeError(internalType.toString());
        }

        const walletAttributes: ReadonlyArray<string> = service.walletAttributes();
        for (const attribute of walletAttributes) {
            this.knownWalletAttributes.delete(attribute);
        }

        Transactions.TransactionRegistry.deregisterTransactionType(transactionConstructor);
        this.registeredTransactionHandlers.delete(internalType);
    }

    public isKnownWalletAttribute(attribute: string): boolean {
        return this.knownWalletAttributes.has(attribute);
    }
}
```

The usage is fairly simple and the custom types would be easiest bootstrapped during the start up of the node.

```ts
import { Transaction } from "@arkecosystem/crypto";
import { TransactionHandler, TransactionHandlerRegistry } from "@arkecosystem/core-transactions";

// This is our custom transaction that we want to register for core
class CustomTransaction extends Transaction {}
class CustomTransactionHandler extends TransactionHandler {}

// This will register a new transaction handler with the core-transactions package which core will be able to pick up
// NOTE: The `TransactionHandlerRegistry` will call `registerType` on the `TransactionRegistry` for us
TransactionHandlerRegistry.registerTransactionHandler(CustomTransactionHandler);
```

### Reserved Type Groups
Core will reserve the first 1000 type groups . Everything beyond is freely available to other developers.

```ts
enum TransactionTypeGroup {
    Test: 0,
    Core: 1,
    Reserved: 1000,
}
```

### Database

In order to support new transaction types without having to do major changes to the database migrations a new `asset` column will be introduced.

This column will be used to store the type specific information of a transaction as a JSON blob to allow easy use of it in SQL queries.

```sql
SELECT * FROM transactions WHERE type = 5 AND asset @> '{"ipfs_id":1}';
```

Queries like this will allow us to do searches on the `asset` information of a transaction without having to add real columns through migrations.

### Fees
All transaction types specify default fees in the respective handler, but if a `fee` is defined inside  `milestones.json` and/or `addonBytes` of the `core-transaction-pool` options that will be taken.