---
title: "Managing packages"
linkTitle: "Managing packages"
weight: 3
date: 2019-12-14
description: >
  How to install packages, manage repositories, ...
---


## Installing a package

To install a package with `luet`, simply run:

```bash

$ luet install <package_name>

```

## Uninstalling a package

To uninstall a package with `luet`, simply run:

```bash

$ luet uninstall <package_name>

```

## Upgrading the system

To upgrade your system, simply run:

```bash

$ luet upgrade

```

## Searching a package

To search a package:

```bash

$ luet search <regex>

```

To search a package and display results in a table:

```bash

$ luet search --table <regex>

```

To look into the installed packages:

```bash

$ luet search --installed <regex>

```

Note: the regex argument is optional


### Search output

Search can return results in the terminal in different ways: as terminal output, as json or as yaml.

#### JSON

```bash

$ luet search --json <regex>

```

#### YAML

```bash

$ luet search --yaml <regex>

```

#### Tabular


```bash

$ luet search --table <regex>

```
