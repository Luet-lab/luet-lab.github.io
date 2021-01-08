---
title: "Specfile"
linkTitle: "Specfile"
weight: 2
description: >
  Luet specfile syntax
---

# Specfiles

Luet [packages](/docs/docs/concepts/packages/) are defined by specfiles. Specfiles define the runtime and builtime requirements of a package.  There is an hard distinction between runtime and buildtime. A spec is composed at least by the runtime (`definition.yaml`) and the buildtime specification (`build.yaml`).

Luet identifies the package definition by looking at directories that contains a `build.yaml` and a `definition.yaml` files. A Luet tree is merely a composition of directories that follows this convention. There is no constriction on either folder naming or hierarchy.

*Example of a [tree folder hierarchy](https://github.com/Luet-lab/luet-embedded/tree/master/distro)*
```bash
tree distro                                                      
distro
├── funtoo              
│   ├── 1.4
│   │   ├── build.sh        
│   │   ├── build.yaml                                             
│   │   ├── definition.yaml
│   │   └── finalize.yaml
│   ├── docker
│   │   ├── build.yaml
│   │   ├── definition.yaml
│   │   └── finalize.yaml
│   └── meta
│       └── rpi
│           └── 0.1
│               ├── build.yaml
│               └── definition.yaml
├── packages
│   ├── container-diff
│   │   └── 0.15.0
│   │       ├── build.yaml
│   │       └── definition.yaml
│   └── luet
│       ├── build.yaml
│       └── definition.yaml
├── raspbian
│   ├── buster
│   │   ├── build.sh
│   │   ├── build.yaml
│   │   ├── definition.yaml
│   │   └── finalize.yaml
│   ├── buster-boot
│   │   ├── build.sh
│   │   ├── build.yaml
│   │   ├── definition.yaml
│   │   └── finalize.yaml
```

## Build specs

Build specs are defined in `build.yaml` files. They denote the build-time `dependencies` and `conflicts`, together with a definition of the content of the package.

*Example of a `build.yaml` file*:
```yaml
steps:
- echo "Luet is awesome" > /awesome
prelude:
- echo "nooops!"
requires:
- name: "echo"
  version: ">=1.0"
conflicts:
- name: "foo"
  version: ">=1.0"
provides:
- name: "bar"
  version: ">=1.0"
env:
- FOO=bar
includes:
- /awesome

unpack: true
```

### Building strategies

Luet can create packages with different strategies: 

- by delta. Luet will analyze the containers differencies to find out which files got **added**. 
  You can use the `prelude` section to exclude certains file during analysis.
- by taking a whole container content
- by considering a single directory in the build container.

#### Package by delta

By default Luet will analyze the container content and extract any file that gets **added** to it. The difference is calculated by using the container which is depending on, or either by the container which is created by running the steps in the `prelude` section of the package build spec:

```yaml
prelude:
- # do something...
steps:
- # real work that should be calculated delta over
```

By omitting the `prelude` keyword, the delta will be calculated from the parent container where the build will start from.

#### Package by container content

Luet can also generate a package content from a container. This is really useful when creating packages that are entire versioned `rootfs`. To enable this behavior, simply add `unpack: true` to the `build.yaml`. This enables the Luet unpacking features, which will extract all the files contained in the container which is built from the `prelude` and `steps` fields.

To include/exclude single files from it, use the `includes` and `excludes` directives.

#### Package by a folder in the final container

Similarly, you can tell Luet to create a package from a folder in the build container. To enable this behavior, simply add `package_dir: "/path/to/final/dir"`.
The directory must represent exactly how the files will be ultimately installed from clients, and they will show up in the same layout in the final archive.

So for example, to create a package which ships `/usr/bin/mybin`, we could write:
```yaml
package_dir: "/output"
steps:
- mkdir -p /output/usr/bin/
- echo "fancy stuff" > /output/usr/bin/mybin && chmod +x /output/usr/bin/mybin
```

### Build time dependencies

A package build spec defines how a package is built. In order to do this, Luet needs to know where to start. Hence a package must declare at least either one of the following:

- an `image` keyword which tells which Docker image to use as base, or
- a list of `requires`, which are references to other packages available in the tree.

They can't be both present in the same specfile. 

To note, it's not possible to mix package build definitions from different `image` sources. They must form a unique sub-graph in the build dependency tree. 

On the other hand it's possible to have multiple packages depending on a combination of different `requires`, given they are coming from the same `image` parent.

### Excluding/including files explictly

Luet can also *exclude* and *include* single files or folders from a package by using the `excludes` and `includes` keyword respecitvely. 

Both of them are parsed as a list of Golang regex expressions, and they can be combined together to fine-grainly decide which files should be inside the final artifact. You can refer to the files as they were in the resulting package. So if a package produces a `/foo` file,  and you want to exclude it, you can add it to `excludes` as `/foo`.

### Keywords

Global:

- `env`: List of environment variables ( in `NAME=value` format ) that are expanded in `step` and in `prelude`. ( e.g. `${NAME}` ).
- `step`: List of commands to perform in the build container.
- `prelude`: List of commands to perform in the build container before building.
- `unpack`: Boolean which indicates if the package content **is** the whole container content.
- `includes`: List of strings which are encoded in logical AND, they denote the content to filter from the container image to be packed. Wildcards and golang regular expressions are supported. If specified, files not matching any of the regular expressions in the list won't be included in the final package.
- `package_dir`: Directory from within the build container which contains the final artefacts of your package
- `excludes`: List of golang regexes. They are in full path form (e.g. `^/usr/bin/foo` ) and indicates that the files listed shouldn't be part of the final artifact
- `includes`: List of golang regexes. They are in full path form (e.g. `^/usr/bin/foo` ) and indicates that the files listed needs to be included as part of the final artifact

By combining `excludes` with `includes`, it's possible to include certain files while excluding explicitly some others (`excludes` takes precedence over `includes`).

Source from external image (e.g. Docker):
- `image`: docker image to be used to build the package (might be omitted)

From a tree dependency:

- `requires`: List of packages which it depends on.
- `conflicts`: List of packages which it conflicts with.

## Rutime specs

Runtime specification are denoted in a `definition.yaml` sibiling file. It identifies the package and the runtime contraints attached to it.

*definition.yaml*:
```yaml
name: "awesome"
version: "0.1"
category: "foo"
requires:
- name: "echo"
  version: ">=1.0"
  category: "bar"
conflicts:
- name: "foo"
  version: "1.0"
provides:
- name: "bar"
  version: "<1.0"
```

### Keywords

Global:

- `name`: Name of the package **required**
- `version`: Version of the package in semver notation. Selectors (`>=`,`<`,`>`,`<=`) here are not supported. You can use selectors only in dependency lists.
- `category`: Category of the package.
- `provides`: List of packages which it replaces.
- `hidden`: Boolean that indicates that the package should be hidden from `luet search`. Note packages can be still installed, they are just omitted to the user. You can inspect any time hidden packages with `luet search --hidden`

Runtime dependency list:

- `requires`: List of packages which it depends on, in runtime.
- `conflicts`: List of packages which it conflicts with, in runtime.

## Refering to packages from the CLI

All the `luet` commands which takes a package as argument, respect the following syntax notation:

- `cat/name`: will default to selecting any available package
- `=cat/name`: will default to gentoo parsing with regexp so also `=cat/name-1.1` works
- `cat/name@version`: will select the specific version wanted ( e.g. `cat/name@1.1` ) but can also include ranges as well `cat/name@>=1.1`
- `name`: just name, category is omitted and considered empty

## Finalizers

Finalizers are denoted in a `finalize.yaml` file, which is a sibiling of `definition.yaml` and `build.yaml` file. It contains a list of commands that finalize the package when it is installed in the machine.

*finalize.yaml*:
```yaml
install:
- rc-update add docker default
```

### Keywords

- `install`: List of commands to run in the host machine. Failures are eventually ignored, but will be reported and luet will exit non-zero in such case.
