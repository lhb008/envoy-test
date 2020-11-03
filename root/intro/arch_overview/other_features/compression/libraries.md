Compression Libraries {#arch_overview_compression_libraries}
=====================

Underlying implementation
-------------------------

Currently Envoy uses [zlib](http://zlib.net) as a compression library.

::: {.note}
::: {.title}
Note
:::

[zlib-ng](https://github.com/zlib-ng/zlib-ng) is a fork that hosts
several 3rd-party contributions containing new optimizations. Those
optimizations are considered useful for [improving compression
performance](https://github.com/envoyproxy/envoy/issues/8448#issuecomment-667152013).
Envoy can be built to use [zlib-ng](https://github.com/zlib-ng/zlib-ng)
instead of regular [zlib](http://zlib.net) by using `--define zlib=ng`
Bazel option. The relevant build options used to build
[zlib-ng](https://github.com/zlib-ng/zlib-ng) can be evaluated in `here
<bazel/foreign_cc/BUILD>`{.interpreted-text role="repo"}. Currently,
this option is only available on Linux.
:::
