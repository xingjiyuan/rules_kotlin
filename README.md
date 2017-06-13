<table><tr>
<td><img src="https://github.com/pubref/rules_protobuf/blob/master/images/bazel.png" width="120"/></td>
<td><img src="https://kotlinlang.org/assets/images/open-graph/kotlin_250x250.png" width="120"/></td>
</tr><tr>
<td>Bazel</td>
<td>Kotlin</td>
</tr></table>

# Kotlin Rules for Bazel
[![Build Status](https://travis-ci.org/pubref/rules_kotlin.svg?branch=master)](https://travis-ci.org/pubref/rules_kotlin)

> Note: **These rules require Bazel 0.4.5 or higher**.

These rules are for building [Kotlin][kotlin] source with with
[Bazel][bazel].

1. [kotlin_repositories](#kotlin_repositories)
1. [kotlin_library](#kotlin_library)
1. [kotlin_binary](#kotlin_binary)
1. [kotlin_test](#kotlin_test)

## Workspace rules

Add the following to your `WORKSPACE` file:

```python
git_repository(
    name = "org_pubref_rules_kotlin",
    remote = "https://github.com/pubref/rules_kotlin.git",
    tag = "v0.3.1", # update as needed
)

load("@org_pubref_rules_kotlin//kotlin:rules.bzl", "kotlin_repositories")

kotlin_repositories()
```

This will fetch a
[kotlin release](https://github.com/JetBrains/kotlin/releases)
(currently 1.1.2-2) and load a number of dependencies related to
dagger (used to build the `KotlinCompiler` bazel worker).

> You can override kotlin release via the `com_github_jetbrains_kotlin_url`, `com_github_jetbrains_kotlin_sha256` options
> and various dependencies loaded in the `kotlin_repositories` rule via the `omit_*` options; 
> see the source file for details.

## BUILD rules

Add the following to your BUILD file:

```python
load("@org_pubref_rules_kotlin//kotlin:rules.bzl", "kotlin_library")
```

### kotlin_library

Example:

```python
kotlin_library(
    name = "my_kotlin_lib",
    srcs = ["kotlin_source_file.kt"],
    deps = [":some_other_kotlin_library_rule"],
    java_deps = [":some_other_java_library_rule", "@another_maven_jar//jar"],
)
```

Use the `deps` attribute to name other `kotlin_library` targets as jar
providers for this rule.  Use the `java_deps` attribute to name other
`java_library` or `java_import` targets (to expose traditional java
classes in your kotlin source).

To compile a set of kotlin sources files with the `kotlinc` tool and
emit the corresponding jar file, use:

```sh
$ bazel build :my_kotlin_lib
Target :my_kotlin_lib up-to-date:
  bazel-bin/.../my_kotlin_lib.jar
```

To use the output of a `kotlin_library` as input to a `java_library`
rule (and make your compiled kotlin classes available to your
traditional java source files), name it as a dependency using the
`deps` attribute, just as you would any other `java_library` target.
The name of this target must be of the form
`{kotlin_library_target_name}_kt`.  For example, to use the
`:my_kotlin_lib` in an `android_binary` target, the name would be
`:my_kotlin_lib_kt`, such as:

```python
android_binary(
   name = "foo",
   deps = [
       ":my_kotlin_lib_kt`,
   ]
)
```

### kotlin_library attributes

| Name | Type | Description |
| --- | --- | --- |
| `srcs` | `label_list` | Kotlin source files `*.kt` |
| `deps` | `label_list` | List of `kotlin_library` targets |
| `java_deps` | `label_list` | List of java provider targets (`java_library`, `java_import`, `...`) |
| `android_deps` | `label_list` | List of android provider targets (`android_library`) |
| `jars` | `label_list` | List of jar file targets (`*.jar`) |
| `x_opts` | `string_list` | List of additional `-X` options to `kotlinc` |
| `plugin_opts` | `string_dict` | List of additional `-P` options to `kotlinc` |


### kotlin_binary

A `kotlin_binary` macro takes the same arguments as a
`kotlin_library`, plus a required `main_class` argument (the name of
the compiled kotlin class to run, in java package notation).  This
class should have a `fun main(...)` entrypoint.  Example:

```python
kotlin_binary(
    name = "main_kt",
    main_class = "my.project.MainKt",
    srcs = ["main.kt"],
    deps = [":my_kotlin_lib"]
    java_deps = [":javalib"]
)
```

To create a self-contained executable jar, invoke the implicit
`_deploy.jar` target. For example:

```sh
$ bazel build :main_kt_deploy.jar
Target :main_kt_deploy.jar up-to-date:
  bazel-bin/.../main_kt_deploy.jar
$ java -jar ./bazel-bin/.../main_kt_deploy.jar
```

> The `kotlin-runtime.jar` is implicitly included by the `kotlin_binary` rule.

#### kotlin_binary attributes

Includes all `kotlin_library` attributes as well as:

| Name | Type | Description |
| --- | --- | --- |
| `main_class` | `string` | Main class to run with the `kotlin_binary` rule |


### kotlin_test

The `kotlin_test` rule is nearly identical the `kotlin_binary` rule
(other than calling `java_test` internally rather than `java_binary`).


```python
kotlin_test(
    name = "main_kt_test",
    test_class = "examples.helloworld.MainKtTest",
    srcs = ["MainKtTest.kt"],
    size = "small",
    deps = [
        ":rules",
    ],
    java_deps = [
        "@junit4//jar",
    ],
)
```

```sh
$ bazel test :main_kt_test.jar
```

> The `kotlin-test.jar` is implicitly included by the `kotlin_test` rule.

### kotlin_compile

> TL;DR; You most likely do not need to interact with the
> `kotlin_compile` rule directly.

The `kotlin_compile` rule runs the kotlin compiler to generate a
`.jar` file from a list of kotlin source files.  The `kotlin_library`
rule calls this internally and then makes the jarfile available to
other java rules via a `java_import` rule.

# Summary

That's it!  Hopefully these rules with make it easy to mix kotlin and
traditional java code in your projects and take advantage of bazel's
approach to fast, repeatable, and reliable builds.

> Note: Consider [rules_maven](https://github.com/pubref/rules_maven)
> for handling transitive maven dependencies with your java/kotlin
> projects.

## Examples

To run the examples in this repository, clone the repo:

```sh
$ git clone https://github.com/pubref/rules_kotlin
$ cd rules_kotlin
$ bazel query //... --output label_kind
$ bazel run examples/helloworld:main_kt
$ bazel run examples/helloworld:main_java
$ bazel test examples/helloworld:main_test
$ bazel test examples/helloworld:main_kt_test
```

## TODO

1. Proper `data` and runfiles support.
2. Proper android support.
4. kapt support.
3. Incremental compilation.

[bazel]: http://www.bazel.io
[kotlin]: http://www.kotlinlang.org
