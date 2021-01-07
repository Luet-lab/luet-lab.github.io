---
title: "Building packages"
linkTitle: "Building packages"
weight: 1
date: 2017-01-05
description: >
  How to build packages with Luet
---


## Prerequisistes

Luet currently supports [Docker](https://www.docker.com/) and [Img](https://github.com/genuinetools/img) as backends to build packages. Both of them can be used and switched in runtime with the ```--backend``` option, so either one of them must be present in the host system.

### Docker

Docker is the (less) experimental Luet engine supported. Be sure to have Docker installed and the daemon running. The user running `luet` commands needs the corresponding permissions to run the `docker` executable, and to connect to a `docker` daemon. The only feature needed by the daemon is the ability to build images, so it fully supports remote daemon as well (this can be specified with the `DOCKER_HOST` environment variable, that is respected by `luet`)

### Img

Luet supports [Img](https://github.com/genuinetools/img). To use it, simply install it in your system, and while running `luet build`, you can switch the backend by providing it as a parameter: `luet build --backend img`. For small packages it is particularly powerful, as it doesn't require any docker daemon running in the host.

### Building packages on Kubernetes

Luet and img can be used together to orchestrate package builds on kubernetes. There is available an experimental [Kubernetes CRD for Luet](https://github.com/mudler/luet-k8s) which allows to build packages seamelessly in Kubernetes and push package artifacts to an S3 Compatible object storage (e.g. Minio).


![Build packages](/tree.jpg)

Building a package with Luet requires only a [definition](/docs/docs/concepts/packages/specfile). This definition can be self-contained and be only composed of one [specfile](/docs/docs/concepts/packages/specfile), or a group of them, forming a Luet tree.

Run `luet build --help` to get more help for each parameter.


Build accepts a list of packages to build, which syntax is in the `category/name-version` notation (following Gentoo syntax). 

## Environmental variables

Luet builds passes its environment variable at the engine which is called during build, so for example the environment variable `DOCKER_HOST` or `DOCKER_BUILDKIT` can be setted.

Every argument from the CLI can be setted via environment variable too with a `LUET_` prefix, for instance the flag `--clean`, can be setted via environment with `LUET_CLEAN`, `--privileged` can be enabled with `LUET_PRIVILEGED` and so on.

Additionally, you can set:
- `DOCKER_SQUASH`: Set to `true` to squash each image being created (Docker backend only)

## Supported compression format

At the moment, `luet` can compress packages and tree with `zstd` and `gzip`. For example: 

```bash
luet build --compression zstd ...
```

Will output package compressed in the zstd format.

See the `--help` of `create-repo` and `build` to learn all the available options.

## Example

A [package definition](/docs/docs/concepts/packages/specfile) is composed of a `build.yaml` and a `definition.yaml`:

```bash
$> mkdir package

$> cat <<EOF > package/build.yaml
image: busybox
steps:
- echo "foo" > /foo
EOF

$> cat <<EOF > package/definition.yaml
name: "foo"
version: "0.1"
category: "bar" # optional!
EOF

```

To build it, simply run `luet build bar/foo` or `luet build --all` to build all the packages in the current directory:

```bash
$> luet build --all

📦  Selecting  foo 0.1
📦  Compiling foo version 0.1 .... ☕
🐋  Downloading image luet/cache-foo-bar-0.1-builder
🐋  Downloading image luet/cache-foo-bar-0.1
📦   foo Generating 🐋  definition for builder image from busybox
🐋  Building image luet/cache-foo-bar-0.1-builder
🐋  Building image luet/cache-foo-bar-0.1-builder done
 Sending build context to Docker daemon  4.096kB
 ...

```

## Notes

If you notice errors about disk space, mind to set the `TMPDIR` env variable to a different folder. By default luet respects the O.S. default (which in the majority of system is `/tmp`).