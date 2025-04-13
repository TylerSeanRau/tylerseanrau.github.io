---
layout: post
title: Do it with bazel
---

Hello community! Over the past few years [bazel](https://bazel.build/)
has become my preferred build tool and I figured I'd do a post on how
to get started with Bazel. This is more of a "Hello World" article than
a discussion of the trade offs. Covering the tradeoffs of between bazel
and another tool like cmake would be a much more detailed article. Instead
my hope with this article is just to inspire curiosity by giving a light
intro.

---

## So what is bazel?

bazel is a feature rich tool. The most pratical feature for me is being
able to compile and build C++ code into binaries and libaries. (bazel
can do so much more than just that though, one of my favorite other
features is using python libraries and dependency management via bazel)

## How do we get it?

For fresh users who've never used bazel before then you're probably best
served by using [bazelisk](https://bazel.build/install/bazelisk)

I personally install bazel using the steps
[here](https://bazel.build/install/ubuntu#install-on-ubuntu) since I'm
usually using some flavor of Debian and am a huge fan of the
[APT](https://en.wikipedia.org/wiki/APT_%28software%29) system but I
would not recommend this unless you are very comfortable with Debian
and APT.

## How do we use it with C++ ?

For the rest of this article I'm going to use the code I published
in my
[Benchmarking and Testing](https://tylerseanrau.github.io/BenchAndTest/)
post from a few years ago. Only this time we'll use bazel instead of
cmake.

Here's the files we'll use as our faux library, let's get them in
a directory we can use them

`source_code.h`:

```cpp
#ifndef SOURCECODE_H
#define SOURCECODE_H

#include <vector>

extern std::vector<int> do_something(void);

#endif
```

`source_code.cpp`:

```cpp
#include "source_code.h"

#include <vector>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

std::vector<int> do_something(void)
{
  int fd = open("/dev/urandom", O_RDONLY);
  if(fd < 0)
    throw "failed to open file";
  int v = 0;
  read(fd, &v, 1);
  close(fd);
  std::vector<int> ret{v};
  return ret;
}
```

With these source files how do we actually build them?

In modern versions of bazel we'll first need to add a
`MODULE.bazel` file. (There is an older `WORKSPACE` system
that is [deprecated and is going to be removed in bazel 9](
https://bazel.build/external/migration#:~:text=Due%20to%20the%20shortcomings%20of,Bazel%209%20%28late%202025%29))
This file can be empty for now we'll come back to it later

Next we'll create a `BUILD.bazel` file for us to setup our
library. That looks like this:

```
cc_library(
  name = "source_code",
  hdrs = ["source_code.h"],
  srcs = ["source_code.cpp"]
)
```

And from here -with just these 4 files- we can do:
```sh
bazel build //...
```

and see our library get built!

```sh
$ bazel-7.3.1 build //...
INFO: Analyzed target //:source_code (86 packages loaded, 394 targets configured).
INFO: Found 1 target...
Target //:source_code up-to-date:
  bazel-bin/libsource_code.a
  bazel-bin/libsource_code.so
INFO: Elapsed time: 2.588s, Critical Path: 0.23s
INFO: 6 processes: 3 internal, 3 linux-sandbox.
INFO: Build completed successfully, 6 total actions
```

And that's it for a basic setup!

To implement the rest of the stuff from the bench and test
we'll just bring those files over and then modify our `BUILD.bazel`
to include them. To refresh here's what those files looked like:

`source_code.test.cpp`:

```cpp
#include "source_code.h"
#include <gtest/gtest.h>

TEST(do_something, CorrectSize)
{
  const std::vector<int> ret = do_something();
  EXPECT_EQ(ret.size(), 1);
}
```

`source_code.benchmark.cpp`:

```cpp
#include "source_code.h"
#include <benchmark/benchmark.h>

static void BM_do_something(benchmark::State & state)
{
  for(auto _ : state)
  {
    std::vector<int> foo;
    benchmark::DoNotOptimize(foo = do_something());
    benchmark::ClobberMemory();
  }
}

BENCHMARK(BM_do_something);
```

(Note this code is not a real benchmark that does anything
meaningful. Check documentation before using
`benchmark::DoNotOptimize` and `benchmark::ClobberMemory`.
At time of writing, the documentation for these is
[here](https://github.com/google/benchmark/blob/main/docs/user_guide.md#:~:text=not%20set%20explicitly.-,Preventing%20Optimization,-To%20prevent%20a))

These files both have an extra dependency we need to bring in.
The test file needs google test and the benchmark needs google
benchmark. How do we bring those in? This is where we need that
`MODULE.bazel` file that we left empty. So what do we put in
there?

Bazel's module system is exceptionally flexible so a lot of
things are possible. The simplest method for dependencies with
bazel's module system is to look the dependency up on the
[Bazel Central Registry](https://registry.bazel.build/). In
this simple case both google test and google benchmark are on
the central registry so we can use the sample hooks that they
mention.

This makes our `MODULE.bazel` look like this now:

```
bazel_dep(name = "googletest", version = "1.15.2")
bazel_dep(name = "google_benchmark", version = "1.9.1")
```

Now let's go back to our `BUILD.bazel` and tie everything back
together:

```
cc_library(
  name = "source_code",
  hdrs = ["source_code.h"],
  srcs = ["source_code.cpp"]
)

cc_test(
  name = "source_code_test",
  srcs = ["source_code.test.cpp"],
  deps = [
    ":source_code",
    "@googletest//:gtest",
    "@googletest//:gtest_main"
  ]
)

cc_binary(
  name = "source_code_benchmark",
  srcs = ["source_code.benchmark.cpp"],
  deps = [
    ":source_code",
    "@google_benchmark//:benchmark",
    "@google_benchmark//:benchmark_main"
  ]
)
```

Now we can do things like:

```sh
bazel-7.3.1 test //...
```

Giving us:

```sh
$ bazel-7.3.1 test //...
Starting local Bazel server and connecting to it...
INFO: Analyzed 3 targets (100 packages loaded, 648 targets configured).
INFO: Found 2 targets and 1 test target...
INFO: Elapsed time: 6.001s, Critical Path: 2.77s
INFO: 61 processes: 16 internal, 45 linux-sandbox.
INFO: Build completed successfully, 61 total actions
//:source_code_test                                                      PASSED in 0.0s

Executed 1 out of 1 test: 1 test passes.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command line option to see which ones these are.
```

We can also build again and get our benchmark binary:
```
$ bazel-7.3.1 build //...
Starting local Bazel server and connecting to it...
INFO: Analyzed 3 targets (100 packages loaded, 648 targets configured).
INFO: Found 3 targets...
INFO: Elapsed time: 5.809s, Critical Path: 2.78s
INFO: 60 processes: 16 internal, 44 linux-sandbox.
INFO: Build completed successfully, 60 total actions
```

We can find our output binary for the benchmark under the
`bazel-out` symlink. We can run that and see:

```sh
bazel-out/k8-fastbuild/bin/source_code_benchmark |& tail --lines=+2
Running bazel-out/k8-fastbuild/bin/source_code_benchmark
Run on (16 X 6843.75 MHz CPU s)
CPU Caches:
  L1 Data 32 KiB (x8)
  L1 Instruction 32 KiB (x8)
  L2 Unified 1024 KiB (x8)
  L3 Unified 16384 KiB (x1)
Load Average: 1.74, 1.79, 1.25
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
***WARNING*** Library was built as DEBUG. Timings may be affected.
----------------------------------------------------------
Benchmark                Time             CPU   Iterations
----------------------------------------------------------
BM_do_something       3298 ns         3298 ns       214927
```

(If we actually were doing a benchmark then we would need
to adjust for those warnings but for now I'll leve that
as an exercise for the reader)

Note the dir `k8-fastbuild` is a combination of our CPU
architecture and the build type we ran. `k8` is another
name for
[x86-64](https://en.wikipedia.org/wiki/AMD_K8#:~:text=The%20K8%20was%20the%20first%20implementation%20of%20the%20AMD64%2064%2Dbit%20extension)
and `fastbuild` is the default build type for bazel.

And that's it! We can now run our tests and benchmarks
with bazel! Congrats! Now you can start exploring more
about bazel and experimenting with it!

## A few more things

### Toolchain selection

A very frustrating thing when setting up build systems on
multiple machines can be ensuring the same toolchain
setup and that it is being used correctly. In comes [toolchains_llvm](https://github.com/bazel-contrib/toolchains_llvm)
as the perfect answer to this. By following details on
their releases you can hook llvm into your `MODULE.bazel`
and get a consistent toolchain on all machines.

Following the [v1.2.0](https://github.com/bazel-contrib/toolchains_llvm/releases/tag/v1.2.0)
release example, If I want to use llvm `18.1.8` I just
need to add this to my `MODULE.bazel` file:

```
bazel_dep(name = "toolchains_llvm", version = "1.2.0")

# Configure and register the toolchain.
llvm = use_extension("@toolchains_llvm//toolchain/extensions:llvm.bzl", "llvm")
llvm.toolchain(
   llvm_version = "18.1.8",
)

use_repo(llvm, "llvm_toolchain")

register_toolchains("@llvm_toolchain//:all")
```

rebuild and now we're using llvm!


### `compile_commands.json`

I love [LSP](https://en.wikipedia.org/wiki/Language_Server_Protocol)
functionality, particularly `clangd`. `clangd` works
best for me when I can generate a `compile_commands.json`
for my codebases. Thankfully there is a really nice way to
do this with bazel with: [bazel-compile-commands-extractor](https://github.com/hedronvision/bazel-compile-commands-extractor)

Following the instructions from the README.md we just need
to do:

```
# Hedron's Compile Commands Extractor for Bazel
# https://github.com/hedronvision/bazel-compile-commands-extractor
bazel_dep(name = "hedron_compile_commands", dev_dependency = True)
git_override(
    module_name = "hedron_compile_commands",
    remote = "https://github.com/hedronvision/bazel-compile-commands-extractor.git",
    commit = "4f28899228fb3ad0126897876f147ca15026151e",
    # Replace the commit hash (above) with the latest (https://github.com/hedronvision/bazel-compile-commands-extractor/commits/main).
    # Even better, set up Renovate and let it do the work for you (see "Suggestion: Updates" in the README).
)
```

Then we just need to do `bazel run @hedron_compile_commands//:refresh_all`
to get `compile_commands.json` for us to use!

## fin

I hope you've enjoyed this quick bazel post and happy coding!
- Tyler Sean Rau
