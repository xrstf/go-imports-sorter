# gimps, the Go IMPort Sorter

This is a fork of [goimports-reviser](https://github.com/incu6us/goimports-reviser). The purpose of
forking was to replace the fixed import classes ("std", "local", "project", ...) with a configurable
set of classes, so that the order can be configured much more fine-grained.

## Example

In [Kubermatic](https://kubermatic.com/) imports are grouped in 4 groups:

- standard library
- external packages that are not in the following two groups
- kubermatic packages
- kubernetes packages

`kubermatic` here means both `k8c.io/*` as well as `github.com/kubermatic/*`, whereas `kubernetes`
includes `k8s.io/*` and `*.k8s.io`.

This setup can be configured in gimps like so:

```yaml
importOrder: [std, external, kubermatic, kubernetes]
sets:
  - name: kubermatic
    patterns:
      - 'k8c.io/**'
      - 'github.com/kubermatic/**'
  - name: kubernetes
    patterns:
      - 'k8s.io/**'
      - '*.k8s.io/**'
```

Note that `std` and `external` are pre-defined by gimps and cannot be configured explicitly.

Then running `gimps -config configfile.yaml .` will automatically fix all Go files, except for
the `vendor` folder and generated files.

## Installation

```
go get go.xrstf.de/gimps
```

## Configuration

gimps uses a `.gimps.yaml` file that can either be given explicitly via `-config FILE.yaml` or
it can be placed in the Go module root (where your `go.mod` lives) and must then be named
`.gimps.yaml`.

The configuration is rather simple:

```yaml
# By default, gimps detects the project name based on the go.mod file.
# If this fails or you don't have a go.mod file, you can configure the
# name here.
projectName: github.com/example/repo

# This list is the order of import sets in the output of each file.
#
#   - `std` is predefined and represents all Go standard library packages
#   - `external` is predefined and represents all packages that do not
#     match any of the other sets.
#   - `project` is predefined and presents packages in the same project
#     (i.e. have the project name as their prefix)
#
# The default order is shown below. If you define more sets (see below),
# add them to this list in the spot where the matching imports should be
# placed.
#
# Important: If you define a set and not use it in the importOrder, the
#            imports that match the set's patterns will be dropped!
importOrder: [std, project, external]

# Define additional groups of imports. Their names are then used in the
# importOrder above.
sets:
  - # a unique name
    name: kubermatic
    # a list of glob-expressions, with the addition that double star
    # expressions are allowed (`foo/**` matches `foo/bar/bar`)
    patterns:
      - 'k8c.io/**'
      - 'github.com/kubermatic/**'
  - name: kubernetes
    patterns:
      - 'k8s.io/**'
      - '*.k8s.io/**'

# whether or not to remove unused imports; usually this is not needed,
# as your editor's Go integration, like gopls, takes care of that already.
removeUnusedImports: false

# transform each import with a version suffix into a more readable import,
# i.e. `"github.com/bmatcuk/doublestar/v4"` => `doublestar "github.com/bmatcuk/doublestar/v4"`
setVersionAlias: true
```

## Running

Put the configuration either in a `.gimps.yaml` in the module root (recommended) or configure
it explicitly via `-config`.

Provide one or more arguments, each being either a file or a directory. Directories are
automatically traversed recursively.

**Important:** The first argument controls the Go module path for the entire operation. It's not
recommended to make gimps work across multiple modules at the same time. Usually you want to
either run it from your module root directory or give it a single file explicitly to facilitate
editor integration when needed.

For the editor integration, you can specify `-stdout` to print the formatted file to stdout. This
only makes sense if you provide exactly one file, otherwise separating the output is difficult.

```bash
$ cd ~/myproject
$ gimps .
```

## License

The original reviser code is MIT licensed and (c) 2020 Vyacheslav Pryimak.

The new parts in this fork are MIT licensed and (c) 2021 Christoph Mewes.
