---
title: "Specfile"
linkTitle: "Specfile"
weight: 2
description: >
  Luet specfile syntax
---

## Specfiles

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

### Build specs

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

#### Keywords

Global:

- `env`: List of environment variables ( in `NAME=value` format ) that are expanded in `step` and in `prelude`. ( e.g. `${NAME}` ).
- `step`: List of commands to perform in the build container.
- `prelude`: List of commands to perform in the build container before building.
- `unpack`: Boolean which indicates if the package content **is** the whole container content.
- `includes`: List of strings which are encoded in logical AND, they denote the content to filter from the container image to be packed. Wildcards and golang regular expressions are supported. If specified, files not matching any of the regular expressions in the list won't be included in the final package.

Source from external image (e.g. Docker):

- `image`: List of packages which it provides.

From a tree dependency:

- `requires`: List of packages which it depends on.
- `conflicts`: List of packages which it conflicts with.

### Rutime specs

Runtime specification are denoted in a `definition.yaml` sibiling file. It identifies the package and the runtime contraints.

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
#### Keywords

Global:

- `name`: Name of the package **required**
- `version`: Version of the package in semver notation. Selectors here are not supported. You can use selectors (`>=`,`<`,`>`,`<=`) only in dependency lists.
- `category`: Category of the package.
- `provides`: List of packages which it replaces.

Runtime dependency list:

- `requires`: List of packages which it depends on, in runtime.
- `conflicts`: List of packages which it conflicts with, in runtime.

### Finalizers

Finalizers are denoted in a `finalize.yaml` file, which is a sibiling of `definition.yaml` and `build.yaml` file. It contains a list of commands that finalize the package when it is installed in the machine.

*finalize.yaml*:
```yaml
install:
- rc-update add docker default || true
```

#### Keywords

- `install`: List of commands to run in the host machine. They are all fatal by default.
