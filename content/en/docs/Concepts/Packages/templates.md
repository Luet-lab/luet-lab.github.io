---
title: "Templated packages"
linkTitle: "Templated packages"
weight: 3
description: >
  Use templates to fine tune build specs
---

Luet supports the [`helm` rendering engine template](https://helm.sh/docs/chart_template_guide/). It's being used to interpolate `build.yaml` and `finalize.yaml` files before their execution.

The `build.yaml` and `finalize.yaml` files are rendered during build time, and it's possible to use the `helm` templating syntax inside such files.

Given the following `definition.yaml`:

```yaml
name: "test"
category: "foo"
version: "1.1"

additional_field: "baz"
```

A `build.yaml` can look like the following, and interpolates it's values during build time:

```yaml
image: ...

steps:
- echo {{.Values.name}} > /package_name
- echo {{.Values.additional_field}} > /extra

```

Which would be for example automatically rendered by luet like the following:

```yaml

image: ...

steps:
- echo test > /package_name
- echo baz > /extra

```

This mechanism can be used in collections as well, and each stanza in `packages` is used to interpolate each single package. 

## References

- [Helm Template syntax guide](https://helm.sh/docs/chart_template_guide/)
- [Helm Templating functions](https://helm.sh/docs/chart_template_guide/function_list/)
- [Helm Templating variable](https://helm.sh/docs/chart_template_guide/variables/)

## Examples

- https://github.com/mocaccinoOS/mocaccino-musl-universe/tree/master/multi-arch/packages/tar

