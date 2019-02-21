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

In order to reduce the amount of duplication and complexity that currently plagues anything transaction related we will implement a `Transaction` that will make it possible to implement a handler or entity for each transaction type and have all logic in a single place.

### Transaction

The `Transaction` abstract on a very basic level would look like the following.

```ts
export abstract class Transaction {
    public static type: TransactionTypes = null;

    public abstract serialize(): ByteBuffer;
    public abstract deserialize(buf: ByteBuffer): void;

    protected abstract apply(wallet: Wallet): void;
    protected abstract revert(wallet: Wallet): void;

    public hasVendorField(): boolean {
        return false;
    }

    // This has to return an AJV schema to validate the transaction format
    public static getSchema(): TransactionSchema {
        throw new NotImplementedError();
    }
}
```

The `serialize` and `deserialize` methods would be called by another entity or service that calls it when needed like the other Crypto SDKs do where we have handlers for specific transaction types. The Crypto SDK way of doing it doesn't fit into the JS SDK as it is tightly coupled to core and not designed purely for end-users users.

### Implementing Transaction Types

Implementing transaction types will be easier as their logic is isolated and boilerplate will be greatly reduced. A simple Transfer of type 0 could look like the following.

```ts
export class TransferTransaction extends Transaction {
    public static type: TransactionTypes = TransactionTypes.Transfer;

    public static getSchema(): schemas.TransactionSchema {
        // This represents an AJV schema (https://github.com/epoberezkin/ajv)
        return schemas.transfer;
    }

    public serialize(): ByteBuffer {
        const { data } = this;
        const buffer = new ByteBuffer(24, true);
        buffer.writeUint64(+new Bignum(data.amount).toFixed());
        buffer.writeUint32(data.expiration || 0);
        buffer.append(bs58check.decode(data.recipientId));

        return buffer;
    }

    public deserialize(buf: ByteBuffer): void {
        const { data } = this;
        data.amount = new Bignum(buf.readUint64().toString());
        data.expiration = buf.readUint32();
        data.recipientId = bs58check.encode(buf.readBytes(21).toBuffer());
    }

    public canBeApplied(wallet: Wallet): boolean {
        return super.canBeApplied(wallet);
    }

    public hasVendorField(): boolean {
        return true;
    }

    protected apply(wallet: Wallet): boolean {
        wallet.balance.plus(this.data.totalAmount);

        return true;
    }

    protected revert(wallet: Wallet): boolean {
        wallet.balance.minus(this.data.totalAmount);

        return true;
    }
}
```

### Registering Transaction Types

After we have created our custom transaction type we need a way of exposing it to the crypto package and core. To do this we will add a `TransactionRepository` class which will hold a few methods to guard against overwrites of core transaction types or already registered custom types.

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

        throw new UnkownTransactionError(type);
    }

    public registerCustomType(constructor: TransactionConstructor): void {
        throw new NotImplementedError();
    }

    public deregisterCustomType(constructor: TransactionConstructor): void {
        throw new NotImplementedError();
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

As you can see we are only able to add and get transaction types, this is to prevent that transaction types accidentally disappear at runtime which core needs to handle.

The usage is fairly simple and the custom types would be easiest bootstrapped during the start up of the node.

```ts
import { TransactionRegistry } from "@arkecosystem/crypto";

// This is our custom transaction that we want to register for core
class CustomTransaction extends Transaction {}

// This will register a new transaction type with the crypto package which core will be able to pick up
TransactionRegistry.registerCustomType(CustomTransaction);
```

### Database

In order to support new transaction types without having to do major changes to the database migrations a new `meta` column will be introduced, the name is subject to change.

This column will be used to store the type specific information of a transaction as a JSON blob to allow easy use of it in SQL queries.

```sql
SELECT * FROM transactions WHERE type = 5 AND meta @> '{"ipfs_id":1}';
```

Queries like this will allow us to do searches on the `meta` information of a transaction without having to add real columns through migrations.
