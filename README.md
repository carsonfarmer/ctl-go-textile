# ctl-go-textile

> Spawn Textile daemons using JavaScript!

## Lead Maintainer

[Carson Farmer](https://github.com/carsonfarmer)

## Table of Contents

## Install

```sh
yarn add @textile/ctl-go-textile
```

## Usage

**Spawn a Textile daemon from Node.js**

```js
// Start a disposable node, and get access to the api
// print the node id, and stop the temporary daemon

const Factory = require('ctl-go-textile')
const f = Factory.create()

f.spawn(async (err, daemon) => {
  if (err) { throw err }

  const id = await daemon.api.profile.id()
  console.log(id)
  daemon.stop()
  })
})
```

**Spawn a Textile daemon from the Browser using the provided remote endpoint**

```js
// Start a remote disposable node, and get access to the api
// print the node id, and stop the temporary daemon

const Factory = require('ctl-go-textile')

const port = 40600
const server = Factory.createServer(port)
const f = Factory.create({ remote: true, port: port })

server.start((err) => {
  if (err) { throw err }

  f.spawn((err, daemon) => {
    if (err) { throw err }

    daemon.api.id(async (err, id) => {
      const id = await daemon.api.profile.id()
      console.log(id)
      daemon.stop(server.stop)
    })
  })
})
```

## Disposable vs non Disposable nodes

`ctl-go-textile` can spawn `disposable` and `non-disposable` daemons.

- `disposable`- Creates on a temporary repo which will be optionally initialized and started (the default), as well cleaned up on process exit. Great for tests.
- `non-disposable` - Requires the user to initialize and start the node, as well as stop and cleanup after wards. Additionally, a non-disposable will allow you to pass a custom repo using the `repoPath` option, if the `repoPath` is not defined, it will use the default repo (`$HOME/.textile`). The `repoPath` parameter is ignored for disposable nodes, as there is a risk of deleting a live repo.

## Batteries not included. Bring your own Textile executable.

Install a `go-textile` binary via `yarn add get-go-textile`.

## API

### `Factory` - `const f = Factory.create([options])`

`Factory.create([options])` returns an object that will expose the `df.spawn` method

- `options` - optional object with:
  - `remote` bool - use remote endpoint to spawn the nodes.
  - `port` number - remote endpoint port. Defaults to 40600.
  - `exec` - Textile executable path. `ctl-go-textile` will attempt to locate it by default.
  - `Client` - A custom Textile API constructor (`js-http-client`) to use instead of the packaged one.

**example:** See [Usage](#usage)

#### Spawn a daemon with `f.spawn([options], callback)`

Spawn the daemon

- `options` is an optional object the following properties:
  - `init` bool (default true) or Object - should the node be initialized
  - `initOptions` object - ...
  - `start` bool (default true) - should the node be started
  - `repoPath` string - the repository path to use for this node, ignored if node is disposable
  - `disposable` bool (default true) - a new repo is created and initialized for each invocation, as well as cleaned up automatically once the process exits
  - `daemonOptions` - array of cmd line arguments to be passed to the textile daemon

- `callback` - is a function with the signature `function (err, daemon)` where:
  - `err` - is the error set if spawning the node is unsuccessful
  - `daemon` - is the daemon controller instance:
    - `api` - a property of `daemon`, an instance of  [js-http-client](https://github.com/textileio/js-http-client) attached to the newly created textile node

**example:** See [Usage](#usage)

#### Get daemon version with `f.version(callback)`

Get the version without spawning a daemon

- `callback` - is a function with the signature `function(err, version)`.

### Remote endpoint - `const server = Factory.createServer([options])`

`Factory.createServer` starts a Factory endpoint.

- `options` is an optional object with the following properties:
  - `port` - the port to start the server on

**example:**
```js
const Factory = require('ctl-go-textile')

const server = Factory.createServer({ port: 40600 })

server.start((err) => {
  if (err) { throw err }

  console.log('endpoint is running')

  server.stop((err) => {
    if (err) { throw err }

    console.log('endpoint has stopped')
  })
})
```

### Daemon Controller - `daemon`

The daemon controller (`daemon`) allows you to interact with the spawned Textile daemon.

#### `daemon.apiAddr` (getter)

Get the address of the running Textile REST API.

#### `daemon.gatewayAddr` (getter)

Get the address of the running Textile HTTP Gateway.

#### `daemon.repoPath` (getter)

Get the current repo path. Returns a string.

#### `daemon.started` (getter)

Is the node started. Returns a boolean.

#### `init([initOpts], callback)`

Initialize a repo.

`initOpts` (optional) is an object with the following properties:
...

`callback` is a function with the signature `function (err, daemon)` where `err` is an Error in case something goes wrong and `daemon` is the daemon controller instance.

#### `daemon.cleanup(callback)`

Delete the repo that was being used. If the node was marked as `disposable` this will be called automatically when the process is exited.

`callback` is a function with the signature `function(err)`.

#### `daemon.start(flags, callback)`

Start the daemon.

`flags` - Flags array to be passed to the `textile daemon` command.

`callback` is a function with the signature `function(err, apiClient)` that receives an instance of `Error` on failure or an instance of `js-http-client` on success.


#### `daemon.stop([timeout, callback])`

Stop the daemon.

`callback` is a function with the signature `function(err)` callback - function that receives an instance of `Error` on failure. Use timeout to specify the grace period in ms before hard stopping the daemon. Otherwise, a grace period of `10500` ms will be used for disposable nodes and `10500 * 3` ms for non disposable nodes.

#### `daemon.killProcess([timeout, callback])`

Kill the `textile daemon` process. Use timeout to specify the grace period in ms before hard stopping the daemon. Otherwise, a grace period of `10500` ms will be used for disposable nodes and `10500 * 3` ms for non disposable nodes.

First a `SIGTERM` is sent, after 10.5 seconds `SIGKILL` is sent if the process hasn't exited yet.

`callback` is a function with the signature `function()` called once the process is killed

#### `daemon.pid(callback)`

Get the pid of the `textile daemon` process. Returns the pid number

`callback` is a function with the signature `function(err, pid)` that receives the `pid` of the running daemon or an `Error` instance on failure

#### `daemon.version(callback)`

Get the version of textile

`callback` is a function with the signature `function(err, version)`

### HTTP Client  - `daemon.api`

An instance of [js-http-client](https://github.com/textileio/js-http-client#api) that is used to interact with the daemon.

This instance is returned for each successfully started Textile daemon, when either `df.spawn({start: true})` (the default) is called, or `daemon.start()` is invoked in the case of nodes that were spawned with `df.spawn({start: false})`.

## ctl-go-textile environment variables

In addition to the API described in previous sections, `ctl-go-textile` also supports several environment variables. This are often very useful when running in different environments, such as CI or when doing integration/interop testing.

_Environment variables precedence order is as follows. Top to bottom, top entry has highest precedence:_

- command line options/method arguments
- env variables
- default values

Meaning that, environment variables override defaults in the configuration file but are superseded by options to `df.spawn({...})`

#### TEXTILE_EXEC

An alternative way of specifying the executable path for the `go-textile` executable.

## Packaging

`ctl-go-textile` can be packaged in Electron applications, but the textile binary _has_ to be excluded from asar (Electron Archives).
[read more about unpack files from asar](https://electron.atom.io/docs/tutorial/application-packaging/#adding-unpacked-files-in-asar-archive).

`ctl-go-textile` will try to detect if used from within an `app.asar` archive and tries to resolve textile from `app.asar.unpacked`. The textile binary is part of the `get-go-textile` module.

```bash
electron-packager ./ --asar.unpackDir=node_modules/get-go-textile
```

See [electron asar example](https://github.com/ipfs/js-ipfsd-ctl/tree/master/examples/electron-asar/)

## Development

Project structure:

```
src
├── defaults
│   ├── config.json
│   └── options.json
├── endpoint
│   ├── routes.js
│   └── server.js
├── factory-client.js
├── factory-daemon.js
├── index.js
├── ipfsd-client.js
├── ipfsd-daemon.js
└── utils
    ├── configure-node.js
    ├── exec.js
    ├── find-executable.js
    ├── flatten.js
    ├── parse-config.js
    ├── repo
    │   ├── create-browser.js
    │   └── create-nodejs.js
    ├── run.js
    ├── set-config-value.js
    └── tmp-dir.js
```

## Contribute

Feel free to join in. All welcome. Open an [issue](https://github.com/textileio/ctl-go-textile/issues)!

## Disclaimer

This project is a direct fork of [`js-ipfsd-ctl`](https://github.com/ipfs/js-ipfsd-ctl), but any issues, omissions, errors, or silly mistakes are the responsability of @carsonfarmer.

## License

[MIT](LICENSE)
