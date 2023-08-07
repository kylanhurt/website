---
title: LoginPlugin
description: change_me
category: SessionKit
published: true
---

# LoginPlugin

The `LoginPlugin` interface and `AbstractLoginPlugin` abstract class are tools for developers to create plugins for the [SessionKit](#). These plugins register custom logic through the use of hooks which are performed at specific points during the [Login](#) call.

The [login-plugin-template](https://github.com/wharfkit/login-plugin-template) is available on Github to help developers get started.

## Usage

For application developers that wish to use a `LoginPlugin` in their application, the plugin code needs to be included in the project and then passed to either the [SessionKit](#) factory or included as part of a [Login](#) call.

### SessionKit

Passing a `LoginPlugin` to the options parameter of the [SessionKit](#) during instantiation causes every call to the [Login](#) method to trigger its custom logic.

```ts
const sessionKit = new SessionKit(
  {
    // ...arguments
  },
  {
    loginPlugins: [new ExampleLoginPlugin()],
  }
)
```

### Login

During an individual [Login](#) call, a `LoginPlugin` can also be passed to only execute during that call.

```ts
const result = await sessionKit.login({
  loginPlugins: [new ExampleLoginPlugin()],
})
```

## Development

The `LoginPlugin` interface and `AbstractLoginPlugin` abstract class are tools for developers to create plugins for the [SessionKit](#). These plugins register custom logic through the use of hooks which are performed at specific points during the [Login](#) call.

The [login-plugin-template](https://github.com/wharfkit/login-plugin-template) is available as a template on Github to help developers get started.

### Structure

When building a `LoginPlugin`, the shape of the plugin can either be a Javascript class or a simple object that conforms to the interface.

```ts
interface LoginPlugin {
  /** A URL friendly (lower case, no spaces, etc) ID for this plugin */
  get id(): string
  /** A function that registers hooks into the transaction flow */
  register: (context: TransactContext) => void
}
```

Examples implementing this pattern can be seen below as both a class and as an object.

#### Class

```ts
// Import the hook types
import { LoginHookTypes } from "@wharfkit/session"

// Create a class that extends AbstractLoginPlugin
export class LoginPluginTemplate extends AbstractLoginPlugin {
  id = "example-plugin"

  register(context) {
    context.addHook(LoginHookTypes.beforeLogin, this.exampleHook)
  }

  // Establish the logic that needs to be performed as a hook
  async exampleHook(request) {
    console.log("Executing custom logic!")
  }
}

// Perform a login and watch the hook execute
const result = session.login({
  loginPlugins: [new LoginPluginTemplate()],
})
```

#### Object

```ts
// Import the hook types
import { LoginHookTypes } from "@wharfkit/session"

// Establish the logic that needs to be performed as a hook
const exampleHook = async (request) => {
  console.log("Executing custom logic!")
}

// Create an object shaped like a LoginPlugin
const examplePlugin = {
  id: "example-plugin",
  register(context) {
    // Register the hook to the appropriate point in the transaction lifecycle
    context.addHook(LoginHookTypes.beforeLogin, exampleHook)
  },
}

// Perform a transaction and watch the hook execute
const result = sessionKit.login({
  loginPlugins: [examplePlugin],
})
```

### Register

At the core of a `LoginPlugin` is the `register` method, which is responsible for registering custom logic at specific points in the transaction lifecycle through the use of hooks. This `register` call is made available to the plugin through the [LoginContext](#) provided as `context`.

```ts
register(context) {
  context.addHook(LoginHookTypes.beforeLogin, exampleHook)
}
```

### Hooks

The login lifecycle currently has 3 points which hooks can be established.

- `beforeLogin`: Occurs before the login request is processed by the [WalletPlugin](#).
- `afterLogin`: Occurs after the login request is completed.

These types are provided by the exported `LoginHookTypes` enumeration. Each hook type is either a mutable hook or an immutable hook, based on where in the lifecycle the hook is executed.

#### Hook Functions

All hooks associated with a `LoginPlugin` follow the same design pattern.

```ts
type LoginHook = (context: LoginContext) => Promise<void>
```

The only parameter passed to these functions is an instance of the [LoginContext](#) that was established to represent the state of the request. This function can perform any required logic to assist or validate the login request, and should return nothing when completed.

An example of a function hook is as follows:

```ts
async function beforeLoginHook(context) {
  // perform logic
}
```