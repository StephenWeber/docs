Atomist is a programmable platform for event-driven automation of
software delivery, with native chatops support.  Developing on Atomist
means building on a Software Delivery Machine, so let’s get started
creating one! The easiest way to get started with Atomist is to work
on your laptop.

## Install the Atomist Command-Line Interface

!!! note "Note"
    You’ll need Git and Node.js (this comes with npm) installed.

```
$ npm install -g @atomist/cli
```

or on a mac, if you prefer [Homebrew][brew]:

```
$ brew install atomist-cli
```

Now you have the atomist command installed which is used for creating
and running an SDM.

[brew]: https://brew.sh/ (Homebrew)

## Create an Atomist Workspace

A workspace is the hub for your delivery activity. It is where you
configure which git repositories or organizations SDMs work on. It’s
also where you connect a workspace to chat, and invite and manage
other users of the workspace. You’ll need a workspace to connect your
own SDM to.

### Configure your CLI:

```
$ atomist config
```
Follow the prompts to authenticate with GitHub and choose your Atomist workspace

```
? Enter your api key from https://app.atomist.com/apikeys
    or hit <ENTER> to select an authentication provider
    to login with Atomist:
Select one of the following authentication providers
available to login with Atomist:
? Authentication Provider GitHub
Waiting for login flow to finish in your browser ⠸
...
Logged in as jrday-fc using GitHub
No workspaces available. Run atomist workspace create
Successfully wrote configuration ~/.atomist/client.config.json
```

### Create a workspace

```
$ atomist workspace create
```
Give your workspace a name and authenticate with GitHub.

```
Logged in as jrday-fc using GitHub
Create a new workspace:
? Workspace Name ghostbusters
Successfully created new workspace ghostbusters with id AC0PLZOCA
Select an SCM provider type to config:
? SCM provider GitHub.com
Please select organizations you want to enable:
? Organizations jrday-fc
Successfully configured SCM provider GitHub.com
```

### Create a Software Delivery Machine

```
$ atomist create sdm
```

```
? Type of SDM to create
  blank
❯ uhura
…
  Successfully created new project uhura at file:~/atomist/projects/user/uhura
$ cd  ~/atomist/projects/user/uhura
$ atomist start
```

Running `atomist start` will install and start the [Uhura
SDM][uhura]. Uhura provides the same delivery goals that are provided
to every workspace by the Atomist service. By installing your own
instance of Uhura, you can extend and customize its behavior.

When the [Uhura][uhura] SDM starts up, it will connect to your Atomist
workspace. Test it out by [creating a new project in the Web
app][create-project] and see the delivery goals execute.

## Next steps

Find many things you can do with an SDM in the
[Developer Guide][developer-guide].

[uhura]: https://github.com/atomist/uhura/ (Uhura)
[developer-guide]: developer/index.md (Atomist Developer Guide)
[setup]: user/index.md (Atomist Setup)
[create-project]: https://app.atomist.com/workspace/project/project (Project Creation)
