# coolports

A *simple*, *source based*, *hermetic* package system, that is *really fast* for cached packages... cool!

## Getting started

You need [bwrap](https://github.com/containers/bubblewrap) to run the build sandbox and [redo](https://github.com/apenwarr/redo) to execute the build rules.

## Running packages in a venv

We support running packages in a container called a venv:

```
$ ./bin/venv ./pkg/{make,oksh,gnu-base}
$ ./venv/bin/venv-run make --version
GNU Make 4.2
```

The requested process is run in a linux user container with top level directories substituted for those
in the requested packages, this allows very lightweight use of package environments.

## Building a package

```
$ redo-ifchange ./pkg/make/.pkg.tar.gz
...

# View package runtime dependencies
$ cat ./pkg/make/.closure
./pkg/libc-rt/.pkg.tar.gz
```

## How it works

- The package tree is a redo based build system.
- Each package has a few files:
  - ./pkg/$name/build-deps
    - A list of build dependencies.
  - ./pkg/$name/run-deps
    - A list of runtime dependencies.
  - ./pkg/$name/build
    - The build/install script executed in the build sandbox.
  - ./pkg/$name/fetch
    - A curl script of files to download.
  - ./pkg/$name/sha256sums
    - Validation sums for the download.
  - ./pkg/$name/files
    - An optional directory of files added to the build directory.
- Each package has has a few computed targets:
  - ./pkg/$name/.pkghash
    - A cryptographic hash representing this package, computed by hashing the *full* dependency graph.
  - ./pkg/$name/.closure
    - A computed list containing the transitive runtime dependencies of this package.
  - ./pkg/$name/.bclosure
    - A computed list of all the transitive build time dependencies of this package.
  - ./pkg/$name/.pkg.tar.gz
    - The actual package contents once build.

So what is the point?

The .pkghash represents each package as a merkle tree encapsulating all dependencies, we can use this as a cache tag and immutable id for that
package.

We gain:

- Transparent build caching, by simply checking https://$cache/$pkghash.tar.gz before performing a build we can skip huge amounts redundant package builds.
- Easy distributed builds by shipping package closures to remote servers.
- Easy access to the source code of our entire system.
- The ability to quickly build all our packages from a tiny set of host dependencies.

## ./bin/do

Instead of redo, you can use the bootstrap 'do' script, which is a pure sh
implementation of redo, it does not support incremental builds, but should
be able to build one off packages.

