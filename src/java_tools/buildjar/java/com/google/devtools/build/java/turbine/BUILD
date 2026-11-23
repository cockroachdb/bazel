load("@bazel_skylib//rules:common_settings.bzl", "bool_flag")
load("@rules_graalvm//graalvm:defs.bzl", "native_image")
load("@rules_java//java:defs.bzl", "java_binary", "java_library")
load("@rules_shell//shell:sh_binary.bzl", "sh_binary")

package(
    default_applicable_licenses = ["//:license"],
    default_visibility = ["//src/java_tools/buildjar:buildjar_package_group"],
)

licenses(["notice"])

filegroup(
    name = "srcs",
    srcs = glob(["**"]),
    visibility = ["//src:__subpackages__"],
)

_TURBINE_MAIN_CLASS = "com.google.turbine.main.Main"

java_library(
    name = "turbine_deps",
    runtime_deps = [
        "//src/main/protobuf:deps_java_proto",
        "//third_party:guava",
        "//third_party:jsr305",
        "//third_party:turbine",
    ],
)

java_binary(
    name = "turbine_direct_binary",
    main_class = _TURBINE_MAIN_CLASS,
    runtime_deps = [":turbine_deps"],
)

# If enabled, build Turbine with Oracle GraalVM rather than GraalVM Community.
# See https://www.graalvm.org/downloads/ for licensing restrictions.
bool_flag(
    name = "use_oracle_graalvm",
    build_setting_default = False,
)

config_setting(
    name = "do_use_oracle_graalvm",
    flag_values = {"use_oracle_graalvm": str(True)},
)

# If enabled, build Turbine with PGO instrumentation and without a PGO profile.
# This is used by :update_turbine_pgo. Requires the use of Oracle GraalVM.
bool_flag(
    name = "pgo_instrument",
    build_setting_default = False,
)

config_setting(
    name = "do_pgo_instrument",
    flag_values = {
        "pgo_instrument": str(True),
        "use_oracle_graalvm": str(True),
    },
)

alias(
    name = "turbine_direct_graal",
    actual = select({
        "@platforms//os:windows": ":turbine_direct_graal_with_app_manifest",
        "//conditions:default": ":turbine_direct_graal_unpatched",
    }),
)

# On Windows, add an app manifest to the binary to force it to run with a UTF-8
# code page. It is built with one, but without the app manifest it will not be
# able to use UTF-8 for filesystem operations.
# https://github.com/oracle/graal/issues/10237
genrule(
    name = "turbine_direct_graal_with_app_manifest",
    srcs = [
        ":turbine_direct_graal_unpatched",
        "turbine_direct_graal.manifest",
    ],
    outs = ["turbine_direct_graal.exe"],
    cmd = """\
cp $(location :turbine_direct_graal_unpatched) $@
chmod +w $@
cat $(location turbine_direct_graal.manifest) | $(location //src:write_manifest) $@
""",
    target_compatible_with = ["@platforms//os:windows"],
    tools = ["//src:write_manifest"],
)

# Run with -c opt.
sh_binary(
    name = "turbine_benchmark",
    srcs = ["turbine_benchmark.sh"],
    args = ["$(rlocationpath turbine_direct_graal)"],
    data = [":turbine_direct_graal"],
    deps = ["@bazel_tools//tools/bash/runfiles"],
)

# Run this target to update profile.iprof.
sh_binary(
    name = "update_turbine_pgo",
    srcs = ["update_turbine_pgo.sh"],
)
