---
  AIP: 29
  Title: Generic Transaction Interface
  Authors: *Brian Faust, Kristjan Kosic, Joshua Noack*
  Status: *Draft*
  Discussions-To: https://github.com/ArkEcosystem/AIPs/issues/64
  Type: *Standards Track*
  Category: Core
  Created: *2019-01-21*
  Last Update: *2019-01-21*
---

## Abstract

This AIP proposes improvements to the structure and expandability of transaction types within Core to solve issues that are present because the initial implementation of the system that make it tedious to implement new transaction types.

## Motivation

At the moment all logic that relates to how transactions are applied, serialised and deserialised is scattered all over the place.

The problem with that is that it makes it more tedious to implement new transaction types and leaves a lot of room for mistakes as logic is not shared and more has to be known about implementation specifics.

## Specification

In order to reduce the amount of duplication and complexity that currently plagues anything transaction related we will implement a `Transaction` class that will make it possible to implement de/serialization logic in a single place while the actual transaction logic for modifying wallets, etc. will be provided by `TransactionHandlers`.

### Transaction

The `Transaction` abstract on a very basic level would look like the following.

```ts
export abstract class Transaction {
    public static type: TransactionTypes = null;

    public static fromHex(hex: string): Transactions;
    public static fromData(data: ITransactionData): Transactions;
    public static toBytes(data: ITransactionData): Buffer;

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

    canBeApplied(transaction: Transaction, wallet: Wallet, walletManager?: IWalletManager): boolean;
    applyToSender(transaction: Transaction, wallet: Wallet): void;
    applyToRecipient(transaction: Transaction, wallet: Wallet): void;
    revertForSender(transaction: Transaction, wallet: Wallet): void;
    revertForRecipient(transaction: Transaction, wallet: Wallet): void;
    apply(transaction: Transaction, wallet: Wallet): void;
    revert(transaction: Transaction, wallet: Wallet): void;

    canEnterTransactionPool(data: ITransactionData, guard: ITransactionGuard): boolean;
    emitEvents(transaction: Transaction, emitter: EventEmitter): void;
}
```
A `TransactionHandler` will tell Core for example how a transaction type operates on wallets and what requirements must be met in order to enter the `TransactionPool`.

### Implementing Transaction Types
Implementing transaction types will be easier as their logic is isolated and boilerplate will be greatly reduced. A simple Transfer of type 0 could look like the following.

```ts
export class TransferTransaction extends Transaction {
    public static type: TransactionTypes = TransactionTypes.Transfer;

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
    public abstract getConstructor(): TransactionConstructor;

    // Common wallet logic for all transaction types
    public canBeApplied(transaction: Transaction, wallet: Wallet, walletManager?: IWalletManager): boolean {
        const { data } = transaction;
        if (wallet.multisignature) {
            throw new UnexpectedMultiSignatureError();
        }

        if (
            wallet.balance
                .minus(data.amount)
                .minus(data.fee)
                .isLessThan(0)
        ) {
            throw new InsufficientBalanceError();
        }

        if (data.senderPublicKey !== wallet.publicKey) {
            throw new SenderWalletMismatchError();
        }

        if (wallet.secondPublicKey) {
            if (!crypto.verifySecondSignature(data, wallet.secondPublicKey)) {
                throw new InvalidSecondSignatureError();
            }
        } else {
            if (data.secondSignature || data.signSignature) {
                if (!configManager.getMilestone().ignoreInvalidSecondSignatureField) {
                    throw new UnexpectedSecondSignatureError();
                }
            }
        }

        return true;
    }

    public applyToSender(transaction: Transaction, wallet: models.Wallet): void {
        const { data } = transaction;
        if (data.senderPublicKey === wallet.publicKey || crypto.getAddress(data.senderPublicKey) === wallet.address) {
            wallet.balance = wallet.balance.minus(data.amount).minus(data.fee);

            this.apply(transaction, wallet);

            wallet.dirty = true;
        }
    }

    public applyToRecipient(transaction: Transaction, wallet: models.Wallet): void {
        const { data } = transaction;
        if (data.recipientId === wallet.address) {
            wallet.balance = wallet.balance.plus(data.amount);
            wallet.dirty = true;
        }
    }

    public revertForSender(transaction: Transaction, wallet: models.Wallet): void {
        const { data } = transaction;
        if (data.senderPublicKey === wallet.publicKey || crypto.getAddress(data.senderPublicKey) === wallet.address) {
            wallet.balance = wallet.balance.plus(data.amount).plus(data.fee);

            this.revert(transaction, wallet);

            wallet.dirty = true;
        }
    }

    public revertForRecipient(transaction: Transaction, wallet: models.Wallet): void {
        const { data } = transaction;
        if (data.recipientId === wallet.address) {
            wallet.balance = wallet.balance.minus(data.amount);
            wallet.dirty = true;
        }
    }

    // Transaction type specific wallet logic is implemented in subclasses
    public abstract apply(transaction: Transaction, wallet: models.Wallet): void;
    public abstract revert(transaction: Transaction, wallet: models.Wallet): void;

    /**
     * Transaction Pool logic
     */
    public canEnterTransactionPool(data: ITransactionData, guard: TransactionPool.ITransactionGuard): boolean {
        guard.pushError(
            data,
            "ERR_UNSUPPORTED",
            `Invalidating transaction of unsupported type '${TransactionTypes[data.type]}'`,
        );
        return false;
    }

    protected typeFromSenderAlreadyInPool(data: ITransactionData, guard: TransactionPool.ITransactionGuard): boolean {
        const { senderPublicKey, type } = data;
        if (guard.pool.senderHasTransactionsOfType(senderPublicKey, type)) {
            guard.pushError(
                data,
                "ERR_PENDING",
                `Sender ${senderPublicKey} already has a transaction of type '${TransactionTypes[type]}' in the pool`,
            );

            return true;
        }

        return false;
    }
}

```

Now, the implementation for a simple `TransferHandler` of type 0 based on the abstract `TransactionHandler` class could look like the following:

```ts
export class TransferTransactionHandler extends TransactionHandler {
    public getConstructor(): TransactionConstructor {
        return TransferTransaction;
    }

    // Type 0 specific wallet logic
    public canBeApplied(transaction: Transaction, wallet: models.Wallet, walletManager?: Database.IWalletManager): boolean {
        return super.canBeApplied(transaction, wallet, walletManager);
    }

    public apply(transaction: Transaction, wallet: models.Wallet): void {
        return;
    }

    public revert(transaction: Transaction, wallet: models.Wallet): void {
        return;
    }

    // Type 0 specific transaction pool logic
    public canEnterTransactionPool(data: ITransactionData, guard: TransactionPool.ITransactionGuard): boolean {
        if (!isRecipientOnActiveNetwork(data)) {
            guard.pushError(
                data,
                "ERR_INVALID_RECIPIENT",
                `Recipient ${data.recipientId} is not on the same network: ${configManager.get("pubKeyHash")}`,
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
    private readonly coreTypes = new Map<TransactionTypes, TransactionConstructor>();
    private readonly customTypes = new Map<number, TransactionConstructor>();

    constructor() {
        Object.values(coreTypes).forEach(coreType => this.registerCoreType(coreType));
    }

    public create(data: ITransactionData): Transaction {
        const instance = new (this.get(data.type) as any)() as Transaction;
        instance.data = data;

        return instance;
    }

    public get(type: TransactionTypes): TransactionConstructor {
        if (this.coreTypes.has(type)) {
            return this.coreTypes.get(type);
        }

        if (this.customTypes.has(type)) {
            return this.customTypes.get(type);
        }


        throw new UnkownTransactionError(type);
    }

    public registerCustomType(constructor: TransactionConstructor): void {
        const { type } = constructor;
        if (this.customTypes.has(type)) {
            throw new TransactionAlreadyRegisteredError(constructor.name);
        }

        // TODO: subject to change, but for now the first 100 types are reserved by Core.
        if (type < 100) {
            throw new TransactionTypeInvalidRangeError(type);
        }

        this.customTypes.set(type, constructor);
        this.updateSchemas(constructor);
        this.updateStaticFees();
    }

    public deregisterCustomType(type: number): void {
        if (this.customTypes.has(type)) {
            const schema = this.customTypes.get(type);
            this.updateSchemas(schema, true);
            this.customTypes.delete(type);
        }
    }

    public updateStaticFees(height?: number): void {
        const customConstructors = Array.from(this.customTypes.values());
        const milestone = configManager.getMilestone(height);
        const { staticFees } = milestone.fees;
        for (const constructor of customConstructors) {
            const { type, name } = constructor;
            if (milestone.fees && milestone.fees.staticFees) {
                const value = staticFees[camelCase(name.replace("Transaction", ""))];
                if (!value) {
                    throw new MissingMilestoneFeeError(name);
                }
                feeManager.set(type, value);
            }
        }
    }

    private registerCoreType(constructor: TransactionConstructor) {
        const { type } = constructor;
        if (this.coreTypes.has(type)) {
            throw new TransactionAlreadyRegisteredError(constructor.name);
        }

        this.coreTypes.set(type, constructor);
        this.updateSchemas(constructor);
    }

    private updateSchemas(transaction: TransactionConstructor) {
        AjvWrapper.extendTransaction(transaction.getSchema());
    }
}
```

The accompanying registry for transaction handlers:
```ts
import { transactionHandlers } from "./handlers";
export type TransactionHandlerConstructor = new () => TransactionHandler;

class TransactionHandlerRegistry {
    private readonly coreTransactionHandlers = new Map<constants.TransactionTypes, TransactionHandler>();
    private readonly customTransactionHandlers = new Map<number, TransactionHandler>();

    constructor() {
        transactionHandlers.forEach((handler: TransactionHandlerConstructor) => {
            this.registerCoreTransactionHandler(handler);
        });
    }

    public get(type: constants.TransactionTypes): TransactionHandler {
        if (this.coreTransactionHandlers.has(type)) {
            return this.coreTransactionHandlers.get(type);
        }

        if (this.customTransactionHandlers.has(type)) {
            return this.customTransactionHandlers.get(type);
        }

        throw new InvalidTransactionTypeError(type);
    }

    public registerCustomTransactionHandler(constructor: TransactionHandlerConstructor): void {
        const handler = new constructor();
        const transactionConstructor = handler.getConstructor();
        const { type } = transactionConstructor;

        if (this.customTransactionHandlers.has(type)) {
            throw new TransactionHandlerAlreadyRegisteredError(type);
        }

        TransactionRegistry.registerCustomType(transactionConstructor);

        this.customTransactionHandlers.set(type, handler);
    }

    public deregisterCustomTransactionHandler(constructor: TransactionHandlerConstructor): void {
        const handler = new constructor();
        const transactionConstructor = handler.getConstructor();
        const { type } = transactionConstructor;

        if (this.customTransactionHandlers.has(type)) {
            TransactionRegistry.deregisterCustomType(type);
            this.customTransactionHandlers.delete(type);
        }
    }

    private registerCoreTransactionHandler(constructor: TransactionHandlerConstructor) {
        const handler = new constructor();
        const transactionConstructor = handler.getConstructor();
        const { type } = transactionConstructor;

        if (this.coreTransactionHandlers.has(type)) {
            throw new TransactionHandlerAlreadyRegisteredError(type);
        }

        this.coreTransactionHandlers.set(type, handler);
    }
}
```

As you can see we are only able to add and get transaction types, this is to prevent that transaction types accidentally disappear at runtime which core needs to handle.

The usage is fairly simple and the custom types would be easiest bootstrapped during the start up of the node.

```ts
import { Transaction } from "@arkecosystem/crypto";
import { TransactionHandler, TransactionHandlerRegistry } from "@arkecosystem/core-transactions";

// This is our custom transaction that we want to register for core
class CustomTransaction extends Transaction {}
class CustomTransactionHandler extends TransactionHandler {}

// This will register a new transaction handler with the core-transactions package which core will be able to pick up
// NOTE: The `TransactionHandlerRegistry` will call `registerType` on the `TransactionRegistry` for us
TransactionHandlerRegistry.registerCustomHandler(CustomTransactionHandler);
```

### Database

In order to support new transaction types without having to do major changes to the database migrations a new `asset` column will be introduced.

This column will be used to store the type specific information of a transaction as a JSON blob to allow easy use of it in SQL queries.

```sql
SELECT * FROM transactions WHERE type = 5 AND asset @> '{"ipfs_id":1}';
```

Queries like this will allow us to do searches on the `asset` information of a transaction without having to add real columns through migrations.

### Fees
All transaction types are required to define a `fee` in the `milestones.json` and `addonBytes` of
the `core-transaction-pool` options.