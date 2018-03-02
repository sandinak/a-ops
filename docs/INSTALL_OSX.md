# MacOS
The OSX install uses [Homebrew][1] .. a common package manager.  There are a few caveats to be aware of when using on OSX

- the version of openssl .. which the default python links .. doesn't support the CYPHERS which are configured and installed on our zabbix server. This means for many tools you'll have to use 'brew' based python.

## Installation Instructions
1. Install [GPGTools][1]
    - generate a key pair if you haven't yet.. this should pop up

```bash
gpg --gen-key
```
1. Install [Homebrew][2] - this is a package manager for OSX that supplies many Unix standards, and tends to keep things more up-to-date than the native OS.
    - Specifically right now the native OSX openssl libs do NOT support current and required ciphers for websites.
1. Install the following brew packages

```bash
  $ brew install git pass
```

1. Now follow the rest of the instructions in [INSTALLATION](INSTALLATION.md)

[1]: http://gpgtools.org
[2]: http://brew.sh
