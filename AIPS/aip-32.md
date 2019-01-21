---
  AIP: 32
  Title: Plugin System Improvements
  Authors: *Brian Faust*
  Status: *Draft*
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: *Standards Track*
  Category: Core
  Created: *2019-01-21*
  Last Update: *2019-01-21*
---

## Abstract

This AIP proposes improvements to the structure and expandability of plugin within Core to solve issues that are present because the initial implementation of the system was done in JavaScript which doesn't enforce any types or contracts.

## Motivation

The current implementation of the plugin system has various issues that have been caused by it being written in JavaScript. Those issues were known but became more apparent after the switch to TypeScript and need resolving in the next major release to provide developers an easier and self documented way of developing plugins.

## Specification

Right now plugins simply export an object named `plugin` that contains a name, register and deregister method without having any expectations or providing helpers through a base class.

```ts
export const plugin: Container.PluginDescriptor = {
    pkg: require("../package.json"),
    defaults,
    alias: "api",
    async register(container: Container.IContainer, options) {
        const server = new Server(options);
        await server.start();

        return server;
    },
    async deregister(container: Container.IContainer, options) {
        if (options.enabled) {
            container.resolvePlugin<Logger.ILogger>("logger").info(`Stopping Public API`);

            await container.resolvePlugin<Server>("api").stop();
        }
    },
};
```

This implementation works completely fine but is fairly verbose and we can't have contracts and doing things like logging or resolving plugins is also fairly tedious. Overall it is a very simple implementation that serves its purpose but causes a lot of duplication and leaves room for user errors as the TypeScript compiler won't complain if nothing or something wrong is returned that the container doesn't know how to handle.

The solution to that problem is to provide an abstract class called `AbstractPlugin` that will enforce an implementation contract and clearly show in an IDE what methods are required and what they are expected to return.

```ts
import Joi from 'joi';

abstract class AbstractPlugin {
    // A plugin receives an instance of the container and it's options after they have been merged with its defaults.
    public constructor (readonly container: Container.IContainer, options: PluginOptions) {}

    // This will be called before "register" is fired.
    abstract async public boot(): void;

    // This will be called when Core starts.
    abstract async public register(): Promise<any>;

    // This will be called when Core shuts down.
    abstract async public deregister(): void;

    // This will be the name used in the container of Core to access the instance of the plugin.
    abstract public getName(): string;

    // This will be the version of the plugin used in Core to allow other plugins to perform checks on it.
    abstract public getVersion(): string;

    // This will be the default configuration provided by the plugin in the form of a Joi schema for validation.
    abstract public getDefaults(): Joi.object;

    // Those methods provide quick access to the container and other plugins without exposing how it is done.
    protected resolvePlugin(key string): Container.IContainer {
      return this.container.resolvePlugin<any>(key);
    }

    protected logger(): Logger.ILogger {
      return this.resolvePlugin();
    }
}
```

An example implementation of the `AbstractPlugin` for `@arkecosystem/core-api` could look like the following. _Keep in mind this is all just pseudo code to illustrate the idea._

```ts
import Joi from 'joi';

export class Plugin extends AbstractPlugin {
    abstract async public boot(): void {
        if (this.options.get('protocol') === 'https') {
            this.options.set('port', 8443);
        }
    }

    abstract async public register(): Promise<any>  {
        return (new Server(options)).start();
    }

    abstract async public deregister(): void {
        if (this.options.get('enabled')) {
            this.logger().info(`Stopping Public API`);

            await this.resolvePlugin("api").stop();
        }
    }

    abstract public getName(): string {
        return 'api';
    }

    abstract public getVersion(): string {
        return '3.0.0';
    }

    abstract public getDefaults(): Joi.object {
        return Joi.object({
            protocol: Joi.string().default('http'),
            port: Joi.number().port().default(4003),
        });
    }
}
```

Now that we expect a plugin to provide us certain information and types we can ensure that the data inside the container is what we want it to be.

The `getDefaults` method will provide a Joi schema which we can use to validate the user configuration and if it is valid we get an object that has all properties casted to be correct types of strings, booleans and integers.
