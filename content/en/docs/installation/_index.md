---
title: "Installation"
weight: 30
description: How to install SOPS, or build it from source
---

## Stable release

Binaries and packages of the latest stable release are available at
<https://github.com/getsops/sops/releases>.

## Development branch

For the adventurous, unstable features are available in the
[main](https://github.com/getsops/sops/commits/main/) branch, which you can install from source:

``` bash
$ mkdir -p $GOPATH/src/github.com/getsops/sops/
$ git clone https://github.com/getsops/sops.git $GOPATH/src/github.com/getsops/sops/
$ cd $GOPATH/src/github.com/getsops/sops/
$ make install
```

(requires Go >= 1.25)

If you don\'t have Go installed, set it up with:

``` bash
$ {apt,yum,brew} install golang
$ echo 'export GOPATH=~/go' >> ~/.bashrc
$ source ~/.bashrc
$ mkdir $GOPATH
```

Or whatever variation of the above fits your system and shell.

To use **SOPS** as a library, take a look at the [decrypt
package](https://pkg.go.dev/github.com/getsops/sops/v3/decrypt).
