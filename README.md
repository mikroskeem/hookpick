# hookpick

hookpick is a tool to manage some operational concepts of Hashicorp Vault, specifically related to the painful process of unsealing, initialising and rekeying Vault.

You provide a configuration file with a map of "datacenters". Each datacenter has a key and an array of hosts. hookpick will perform actions against each of those hosts as you need.

The name comes from a a Hook Pick, a tool used to pick locks. This tool is meant to to "unlock" the administration of Vault. Originally the tool was called locksmith, but it conflicted with [locksmith](https://github.com/coreos/locksmith).

It is currently considered Alpha, and may change drastically over time.

# Why?

Originally, I wrote [unseal](https://github.com/jaxxstorm/unseal) which was specifically for unsealing a large number of Vault servers. However, it became apparent that operating on large numbers of Vaults is painful, especially when it comes to rekeying.

This tool is aimed at bridging the gap when it comes to administration and operation of large numbers of Vault servers.

# Features

Some of the advantages you might gain over using the Vault HTTP API or the standard Vault binary

- Zero touch interaction. Once you've written your yaml config, you can simply invoke the command and it'll operate on the Vault servers you need to.
- Parallel execution. Each unseal command runs in a goroutine, meaning you can unseal multiple servers in a matter of seconds

Currently Unseal has the capability to:

- Query the status of all Vault servers configured
- Unseal all Vault servers configured, with a key specified.

# Usage

You'll need a configuration file. Unseal uses [viper](https://github.com/spf13/viper) which means it supports JSON, yaml and hcl syntax.

The app will look for the config file in the following directories, in order:

 - `$HOME/.hookpick.yaml`
 - `.hookpick.yaml` (in the directory you're running the binary from)

An example configuration file in yaml looks like this:

```yml
gpg: true
datacenters:
- hosts:
  - name: consulserver-1.example.dc1.com
    port: 8200
  - name: consulserver-2.example.dc1.com
    port: 8200
  keys:
  - key: <key1>
  - key: <key2>
  name: dc1
- hosts:
  - name: consulserver-1.example.dc2.com
    port: 8200
  - name: consulserver-2.example.dc2.com
    port: 8200
  keys:
  - key: <key1>
  - key: <key2>
  name: dc2
```

This can be converted to JSON or HCL as needed. Configuration options available are:

 - `gpg` - Boolean - Set to true if you init'd Vault with [GPG support](https://www.vaultproject.io/docs/concepts/pgp-gpg-keybase.html) enabled
 - `capath` - String - The path to a directory containing CA certificates for all Vaults
 - `protocol` - String - The HTTP protocol to use when connecting to vaults (default: `https`)
 - `datacenters` - Array of maps - an array of datacenters with nested options
   - `name` - String - The name of the datacenters
   - `keys` - Array - contains keys:
     - `key` - String - The unseal key for that datacenter. Should be base64 encoded if the `gpg` flag is set to true
   - `hosts` - Array - contains two config options:
     - `name` - String - Hostname of a Vault server
     - `port` - Int - The port that Vault server listens on

## Environment Variables

By default, hookpick will read some environment variables for your configuration. You can find them [here](https://www.vaultproject.io/docs/commands/environment.html)

You can use _some_ of these environment variables if you wish when using hookpick.

 - `VAULT_CACERT`: Set this to the path of a CA Cert you wish to use to verify the Vault connection. Note, this will use the same CA cert for all Vaults
 - `VAULT_CAPATH`: An alternative to the above CA Path config option.
 - `VAULT_CLIENT_CERT`: An SSL client cert to use when connecting to your Vaults. Note, this will use the same cert for all Vaults
 - `VAULT_CLIENT_KEY`: An SSL client key to use when connecting to your Vaults. Note, this will use the same key for all Vaults
 - `VAULT_SKIP_VERIFY`: Skip SSL verification. This is not recommended in production use.

# Building

If you want to contribute, we use [Go Modules](https://github.com/golang/go/wiki/Modules) for dependency management, so it should be as simple as:

 - cloning this repo into `$GOPATH/src/github.com/jaxxstorm/hookpick`
 - run `go get -u` from the directory
 - run `go mod tidy` from the directory
 - run `go build -o hookpick main.go`

## Building Docker Image

If you want to build the Docker image:

 - cloning this repo into `$GOPATH/src/github.com/jaxxstorm/hookpick`
 - run `docker build -t hookpick .` from the directory

You should have a tiny image `hookpick` which is less than 5 Mb.

For using it :

 - Create you configfile `.hookpick.yaml`
 - Run docker command `docker run -v $(pwd)/.hookpick.yaml:/.hookpick.yaml:ro hookpick status`
 - 
__Nota__: you can change `status` by one of the program command. (`unseal` if omited)