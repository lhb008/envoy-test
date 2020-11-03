Building
========

The Envoy build system uses Bazel. In order to ease initial building and
for a quick start, we provide an Ubuntu 16 based docker container that
has everything needed inside of it to build and *statically link* Envoy,
see `ci/README.md`{.interpreted-text role="repo"}.

In order to build manually, follow the instructions at
`bazel/README.md`{.interpreted-text role="repo"}.

Requirements {#install_requirements}
------------

Envoy was initially developed and deployed on Ubuntu 14.04 LTS. It
should work on any reasonably recent Linux including Ubuntu 18.04 LTS.

Building Envoy has the following requirements:

-   GCC 7+ or Clang/LLVM 7+ (for C++14 support). Clang/LLVM 9+ preferred
    where Clang is used (see below).
-   These
    `Bazel native <bazel/repository_locations.bzl>`{.interpreted-text
    role="repo"} dependencies.

Please see the linked `CI <ci/README.md>`{.interpreted-text role="repo"}
and `Bazel <bazel/README.md>`{.interpreted-text role="repo"}
documentation for more information on performing manual builds. Please
note that for Clang/LLVM 8 and lower, Envoy may need to be built with
[\--define tcmalloc=gperftools]{.title-ref} as the new tcmalloc code is
not guaranteed to compile with lower versions of Clang.

Pre-built binaries {#install_binaries}
------------------

We build and tag Docker images with release versions when we do official
releases. These images can be found in the following repositories:

-   [envoyproxy/envoy](https://hub.docker.com/r/envoyproxy/envoy/tags/):
    Release binary with symbols stripped on top of an Ubuntu Bionic
    base.
-   [envoyproxy/envoy-debug](https://hub.docker.com/r/envoyproxy/envoy-debug/tags/):
    Release binary with debug symbols on top of an Ubuntu Bionic base.
-   [envoyproxy/envoy-alpine](https://hub.docker.com/r/envoyproxy/envoy-alpine/tags/):
    Release binary with symbols stripped on top of a **glibc** alpine
    base.
-   [envoyproxy/envoy-alpine-debug](https://hub.docker.com/r/envoyproxy/envoy-alpine-debug/tags/):
    *Deprecated in favor of envoyproxy/envoy-debug.* Release binary with
    debug symbols on top of a Release binary with debug symbols on top
    of a **glibc** alpine base.

::: {.note}
::: {.title}
Note
:::

In the above repositories, we tag a *vX.Y-latest* image for each
security/stable release line.
:::

On every master commit we additionally create a set of development
Docker images. These images can be found in the following repositories:

-   [envoyproxy/envoy-dev](https://hub.docker.com/r/envoyproxy/envoy-dev/tags/):
    Release binary with symbols stripped on top of an Ubuntu Bionic
    base.
-   [envoyproxy/envoy-debug-dev](https://hub.docker.com/r/envoyproxy/envoy-debug-dev/tags/):
    Release binary with debug symbols on top of an Ubuntu Bionic base.
-   [envoyproxy/envoy-alpine-dev](https://hub.docker.com/r/envoyproxy/envoy-alpine-dev/tags/):
    Release binary with symbols stripped on top of a **glibc** alpine
    base.
-   [envoyproxy/envoy-alpine-debug-dev](https://hub.docker.com/r/envoyproxy/envoy-alpine-debug-dev/tags/):
    *Deprecated in favor of envoyproxy/envoy-debug-dev.* Release binary
    with debug symbols on top of a **glibc** alpine base.

In the above *dev* repositories, the *latest* tag points to the last
Envoy SHA in master that passed tests.

::: {.note}
::: {.title}
Note
:::

The Envoy project considers master to be release candidate quality at
all times, and many organizations track and deploy master in production.
We encourage you to do the same so that issues can be reported as early
as possible in the development process.
:::

Packaged Envoy pre-built binaries for a variety of platforms are
available via [GetEnvoy.io](https://www.getenvoy.io/).

We will consider producing additional binary types depending on
community interest in helping with CI, packaging, etc. Please open an
[issue in GetEnvoy](https://github.com/tetratelabs/getenvoy/issues) for
pre-built binaries for different platforms.

### ARM64 binaries {#arm_binaries}

[envoyproxy/envoy](https://hub.docker.com/r/envoyproxy/envoy/tags/),
[envoyproxy/envoy-debug](https://hub.docker.com/r/envoyproxy/envoy-debug/tags/),
[envoyproxy/envoy-dev](https://hub.docker.com/r/envoyproxy/envoy-dev/tags/)
and
[envoyproxy/envoy-debug-dev](https://hub.docker.com/r/envoyproxy/envoy-debug-dev/tags/)
are Docker
[multi-arch](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/)
images and should run transparently on compatible ARM64 hosts.

Modifying Envoy
---------------

If you\'re interested in modifying Envoy and testing your changes, one
approach is to use Docker. This guide will walk through the process of
building your own Envoy binary, and putting the binary in an Ubuntu
container.

::: {.toctree maxdepth="2"}
sandboxes/local_docker_build
:::
