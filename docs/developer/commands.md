In an [Software Delivery Machine (SDM)][sdm], a command is an action
that can be triggered on demand, either from the command line in
[local mode][local], from chat in [team mode][team], or in the Atomist
web interface.  This can be any action that you write in a TypeScript
function.

What would you like your team to do more frequently or more
consistently?

This page will show you how to:

* create a command
* test your command
* respond to the person who invoked the command
* [wrap a shell script](#wrap-a-shell-script) in a command
* define [parameters](#command-parameters) for your command

This page starts after you have [created][create] an SDM.

## Create a command

First, we'll write the function for our command.  For this example, we
will write the classic "hello, world" function.  To be able to respond
regardless of whether the command was invoked on the command line, in
chat, or the web interface, we need to learn a little about what a
command function looks like.

We call these command functions "listeners", because they are always
listening for someone to call them.  All command listeners take a
single argument, a `CommandListenerInvocation`, which provides some
useful information to the command, like how to respond to the person
who invoked the command.

Command listeners are asynchronous functions, returning a `Promise`.
The `Promise` contains an object with at least a `code`
property, whose value is a `number`.  A code of zero means success.
  If the code is non-zero, the
command execution is considered unsuccessful.

Armed with this information, we can write our command listener
function.

```typescript
import { HandlerResult, NoParameters } from "@atomist/automation-client";
import { CommandListenerInvocation } from "@atomist/sdm";
export async function helloWorldListener(ci: CommandListenerInvocation<NoParameters>): Promise<HandlerResult> {
    return ci.addressChannels("Hello, world");
}
```

You can see the `CommandListenerInvocation` has an `addressChannels` property,
which sends a message to the appropriate places -- in this case, to wherever the
command was invoked.
Like much of the API, this is an asynchronous function.

### Register your command

The next thing to do is register your command in your SDM.  First, we
create a `CommandHandlerRegistration`.

```typescript
import { CommandHandlerRegistration } from "@atomist/sdm";
const helloWorldCommand: CommandHandlerRegistration = {
    name: "HelloWorld",
    description: "Responds with a friendly greeting to everyone",
    intent: "hello",
    listener: async ci => {
        await ci.addressChannels("Hello, world");
        return { code: 0 };
    },
};
```

We provide a unique name and description in the registration.  The
value of the `intent` property defines the command you enter to invoke
the command listener function.  Here we have defined our function
inline.  In the test, we would also have to update the invocation of
the command to `helloWorldCommand.listener(ci)`.

Once we have the registration, we can add the command to our SDM
object.

```typescript
import { Configuration } from "@atomist/automation-client";
import {
    SoftwareDeliveryMachine,
    SoftwareDeliveryMachineConfiguration,
} from "@atomist/sdm";
import {
    createSoftwareDeliveryMachine,
    configureSdm,
} from "@atomist/sdm-core";
function machine(configuration: SoftwareDeliveryMachineConfiguration): SoftwareDeliveryMachine {
    const sdm = createSoftwareDeliveryMachine({
        name: "My SDM",
        configuration,
    });
    sdm.addCommand(helloWorldCommand);
    return sdm;
}
export const configuration: Configuration = {
    postProcessors: [configureSdm(machine)],
};
```

The `configuration` object should be exported from the `index.ts` of
your SDM.

### Run your command

If you are running your SDM in local mode, start the SDM as you
normally would

```
atomist start --local
```

and then in another terminal run the `hello` command:

```
atomist hello
```

and the SDM should respond in the same terminal.

In team mode, once you have started your SDM you can send the intent
to the Atomist bot

```
@atomist hello
```

and it will respond back to you in the channel.

In chat, you can send `@atomist describe skill "hello"`
to see details of your command and its parameters.

## Wrap a shell script

If you want to turn a shell script into a bot command, just call that
shell script from a command listener.  You can use the [`execPromise`][apidoc-execpromise]
helper to capture the output and respond back with it.

[apidoc-execpromise]: https://atomist.github.io/automation-client/modules/_lib_util_child_process_.html#execpromise (API doc for execPromise)

```typescript
import { execPromise } from "@atomist/automation-client";
import { CommandHandlerRegistration } from "@atomist/sdm";
const myScriptCommand: CommandHandlerRegistration = {
    name: "MyScript",
    description: "Run my-script and respond with its stdout",
    intent: "my script",
    listener: async ci => {
        const result = await execPromise("my-script", ["its", "args"]);
        return ci.addressChannels(result.stdout);
    },
};
```

See also: [Running external commands](spawn.md)

## Command parameters

Command parameters give you extra information that can be different each time
the command runs.

By default, commands not require parameters. However, you can specify parameters to be gathered in chat by the Atomist bot
or via the web interface or CLI. These parameters can be accessed in a typesafe manner.

To specify commands,  use the optional `parameters` property when creating a command. Let's make our
hello command welcome us by name to wherever we are.

### Defining parameters

Start by defining the parameters. A parameter definition object maps the name of each
parameter to optional details. Pass it to the command registration so that it will know
what parameters to gather.

```typescript
const helloWorldParametersDefinition = {
      name: { description: "name",
              required: true,
              pattern: /.*/, },
      location: {},
    };

const helloWorldCommand: CommandHandlerRegistration<{ name: string, location: string }> = {
    name: "HelloWorld",
    description: "Responds with a friendly greeting to everyone",
    intent: "hello",
    parameters: helloWorldParametersDefinition,
    listener: async ci => {
        return ci.addressChannels(`Welcome to ${ci.parameters.location}, ${ci.parameters.name}`);
    },
};
```

Atomist will now prompt anyone who invokes this command for `name` and `location`. In this case
any value will be accepted,
but we can use regular expressions to ensure that valid values are submitted.

The `helloWorldParametersDefinition` object has one property for each parameter. Each property's name is the
name of the parameter, and its value describes the parameter. These options are available in the [parameter definition][apidoc-parameterdef]
(all optional):

| attribute    |  type  | description | default |
| -------------| ------ | ----------- | ------- |
| description  | string | short description | same as name |
| pattern | RegExp | regular expression that the parameter's value must match | match any single line string (`^.*$`) |
| required | boolean | is the parameter required? | false |
| displayName | string | name to display; may contain spaces | same as name |
| validInput | string | describe what makes a valid parameter value | blank |
| displayable | boolean | whether to show a parameter value after it's been entered | true |
| maxLength | number | maximum number of characters to accept | no maximum |
| minLength | number | minmum number of characters needed | 0 |
| type | ParameterType | string, boolean, number, or [Chooser][chooser-apidoc] | string |
| order | number | when prompting, ask for smaller 'order' parameters first | order doesn't matter |
| group | Group | when prompting, put parameters in the same Group together | none |
| control | "input" or "textarea" | input type for string parameters in dialogues | "input" |

[chooser-apidoc]: https://atomist.github.io/automation-client/interfaces/_lib_metadata_automationmetadata_.chooser.html (API doc for Chooser)
[apidoc-parameterdef]: https://atomist.github.io/automation-client/interfaces/_lib_internal_metadata_decoratorsupport_.parameter.html (API doc for Parameter definition)

We can combine parameter definitions using spreads. For example, this will also bring in some common parameters
defined in another object constant:

```typescript
parameters: {
    ...CommonParameters,
    name: { description: "name", required: true, pattern: /.*/, },
    body: { description: "multi line body text", required: true, pattern: /[\s\S]*/, control: "textarea" }
    location: {},
  },
```

The property definition mechanism applies to all commands, so you can apply it to code inspections and code transform commands.

### Accessing parameters in the command

The `CommandHandlerRegistration` type is parameterized by the type of the properties object.
In this case it's the anonymous type `{ name: string, location: string }`, but in more complex scenarios we'd use an interface.
By default, no parameters are exposed.

We access the parameter values in the listener via the `parameters` property, as in `ci.parameters.name`:

```typescript
    listener: async ci => {
        return ci.addressChannels(`Welcome to ${ci.parameters.location}, ${ci.parameters.name}`);
    },
```

### Test your command

Testing your command listener requires that you mock the parts of
`CommandListenerInvocation` you use and then call your function.
Using the [Mocha][mocha] testing framework, it would look something
like this.

```typescript
import { NoParameters } from "@atomist/automation-client";
import { CommandListenerInvocation } from "@atomist/sdm";
import * as assert from "assert";
describe("helloWorldListener", () => {
    it("should respond successfully", async () => {
        let response: string;
        const ci: CommandListenerInvocation<NoParameters> = {
            addressChannels: async (m: string) => response = m,
        };
        const result = await helloWorldListener(ci);
        assert(result);
        assert(result.code === 0);
        assert(response === "Hello, world");
    });
});
```

We assign the message sent to the `response` property so we can later
confirm it was the message we expected.

[mocha]: https://mochajs.org/ (Mocha Test Framework)

## What else would you like to do?

What is missing from this page? Please tell me! There's a #docs
channel in [Atomist community Slack][join], or you can create an issue
on [this repository][this-repo].

[sdm]: sdm.md (Atomist Software Delivery Machine)
[local]: local.md (Atomist SDM Local Mode)
[team]: team.md (Atomist SDM Team Mode)
[join]: https://join.atomist.com/ (Atomist community Slack)
[this-repo]: https://github.com/atomist/docs (Atomist Docs repository)
[create]: sdm.md#creating-an-sdm-project (Create an SDM)
