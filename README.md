# AVR Bazel build rules

This directory contains some special Bazel build rules to produce joint x86 and AVR
build targets. This allows defining a single target definition per BUILD file to
produce libraries for both targets, to avoid having to double-declare them.

An AVR library can be defined in the same way a C++ library is defined:
```bazel
avr_library(
  name = "my_library",
  hdrs = ["my_lib.h"],
  srcs = ["my_lib.cpp"],
  deps = [":my_other_library"],
)
```

This produces a traditional x86 cc_library target called `my_library`, and an AVR
target called `my_library_avr`. An AVR-only library can be produced with
`avr_pure_library`:
```bazel
avr_pure_library(
  name = "my_pure_library",
  hdrs = ["my_pure_lib.h"],
  srcs = ["my_pure_lib.cpp"],
  deps = [":my_other_library"],
)
```

This will only produce an AVR target called `my_pure_library_avr`. See the full
list of rules below for more rules and info.

## Using these rules
To use these rules, add the following to your `WORKSPACE` file:

```bazel
load(
  "@bazel_tools//tools/build_defs/repo:git.bzl",
  "git_repository",
)

git_repository(
  name = "avr-bazel",
  branch = "master",
  remote = "https://github.com/taylorconor/avr-bazel",
)

# initialise the avr-basel rules.
load(
  "@avr-bazel//:avr.bzl",
  "avr_tools_repository",
)
avr_tools_repository()
```

## List of rules
`avr.bzl` contains a number of rule definitions to work with AVR libraries and
binaries:
- `avr_library`: similar to `cc_library`, produces a regular `cc_library` target
along with an AVR target, with name original_target_name*_avr*.
- `avr_pure_library`: similar to `avr_library` except does not produce a
`cc_library` target.
- `avr_binary`: similar to `cc_binary`, produces an AVR binary file.
- `avr_hex`: produces a hex file to be used for flashing AVR microcontrollers.

Any `cc_library` or `cc_test` targeting non-AVR targets can depend on
`avr_library` targets and will automatically depend on the non-AVR version of that
target. This is because `avr_library` is a rule that produces two targets. One
regular `cc_library` target, and one `avr_pure_library` target, with a name ending
in `_avr`.

Please note that this is not a complete toolchain, but rather a rule that generates
other rules. This has a number of limitations, including being less portable than
a complete toolchain, and producing hidden target names (ending in _avr).