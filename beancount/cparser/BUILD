load("@rules_cc//cc:defs.bzl", "cc_proto_library")
load("@rules_proto//proto:defs.bzl", "proto_library")
load("@com_google_protobuf//:protobuf.bzl", "py_proto_library")
load("//third_party/python:py_extension.bzl", "py_extension")
#load("@pybind11_bazel//:build_defs.bzl", "pybind_extension")
load("//third_party/parser:bison.bzl", "genyacc")
load("//third_party/parser:reflex.bzl", "genreflex")

package(default_visibility = ["//visibility:public"])

#-------------------------------------------------------------------------------
# Scanner and grammar rules.

genreflex(
    name = "scanner",
    src = "scanner.lxx",
    out = "scanner.cc",
    # Set this to true to debug the scanner.
    debug = False,
    header_out = "scanner.h",
    includes = [
        "parser.h",
    ],
)

genyacc(
    name = "parser_generated",
    src = "parser.yxx",
    # Uncomment this to debug reduce/reduce conflicts.
    #extra_options = ["-Wcounterexamples"],
    location_out = "location.h",
    header_out = "parser.h",
    source_out = "parser.cc",
)

cc_library(
    name = "parser_headers",
    hdrs = [
        "scanner.h",
        "parser.h",
        "location.h",
    ],
    deps = [
        "//beancount/ccore:number",
        "//beancount/ccore:date",
        "//third_party/parser:reflex",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/time",
        "@com_google_absl//absl/status:statusor",
    ]
)

cc_library(
    name = "builder",
    srcs = ["builder.cc"],
    hdrs = ["builder.h"],
    deps = [
        ":parser_headers",
        ":inter_cc_proto",
        ":options_cc_proto",
        ":ledger",
        "//beancount/ccore:std_utils",
        "//beancount/ccore:account_types",
        "//beancount/ccore:account",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/types:optional",
        "@mpdecimal//:mpdecimal",
        "@com_google_protobuf//:protobuf",
        "@com_google_re2//:re2",
    ]
)

cc_library(
    name = "parser",
    srcs = [
        "scanner.cc",
        "parser.cc",
    ],
    copts = [
        "-Wno-implicit-fallthrough",
        "-O2",
        "-Wno-sign-compare",
    ],
    # Generated and real files are interdependent.
    tags = ["nofixdeps"],
    deps = [
        ":builder",
        ":inter_cc_proto",
        ":options_cc_proto",
        "//beancount/ccore:date",
        "//beancount/ccore:number",
        "//beancount/ccore:data_cc_proto",
        "//third_party/parser:reflex",
        ":parser_headers",
        ":ledger",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/types:optional",
        "@com_google_absl//absl/time",
        "@mpdecimal//:mpdecimal",
    ],
)

cc_test(
    name = "scanner_test",
    srcs = ["scanner_test.cc"],
    deps = [
        ":test_utils",
        ":parser",
        "@com_google_googletest//:gtest_main",
    ],
)

cc_test(
    name = "parser_test",
    srcs = ["parser_test.cc"],
    deps = [
        "//beancount/ccore:data_cc_proto",
        ":test_utils",
        ":parser",
        ":builder",
        ":ledger",
        "@com_google_googletest//:gtest_main",
        "@com_google_absl//absl/strings",
        "@com_google_protobuf//:protobuf",
    ],
)

#-------------------------------------------------------------------------------
# Intermediate representation.

proto_library(
    name = "parser_proto",
    srcs = ["inter.proto"],
    deps = [
        "//beancount/ccore:number_proto",
        "//beancount/ccore:date_proto",
    ],
)

cc_proto_library(
    name = "inter_cc_proto",
    deps = [":parser_proto"],
)

py_proto_library(
    name = "inter_py_proto",
    srcs = ["inter.proto"],
    deps = [
        "//beancount/ccore:number_py_proto",
        "//beancount/ccore:date_py_proto",
    ],
)

#-------------------------------------------------------------------------------
# Options and other info.

proto_library(
    name = "options_proto",
    srcs = ["options.proto"],
    deps = [
        "//beancount/ccore:precision_proto",
    ],
)

cc_proto_library(
    name = "options_cc_proto",
    deps = [":options_proto"],
)

py_proto_library(
    name = "options_py_proto",
    srcs = ["options.proto"],
    deps = [
        "//beancount/ccore:precision_py_proto",
    ],
)

#-------------------------------------------------------------------------------
# Proto container for everything.

proto_library(
    name = "ledger_proto",
    srcs = ["ledger.proto"],
    deps = [
        "//beancount/ccore:data_proto",
        "//beancount/cparser:options_proto",
    ],
)

cc_proto_library(
    name = "ledger_cc_proto",
    deps = [":ledger_proto"],
)

cc_library(
    name = "ledger",
    srcs = ["ledger.cc"],
    hdrs = ["ledger.h"],
    deps = [
        "//beancount/ccore:data_cc_proto",
        "//beancount/ccore:number",
        "//beancount/ccore:precision",
        "//beancount/ccore:std_utils",
        "//beancount/cparser:inter_cc_proto",
        "//beancount/cparser:ledger_cc_proto",
        "//beancount/cparser:options_cc_proto",
        "//beancount/utils:errors",
        "@com_google_protobuf//:protobuf",
        "@com_google_absl//absl/strings:str_format",
        "@mpdecimal//:mpdecimal",
    ]
)

#-------------------------------------------------------------------------------
# Extension module.

# TODO(blais): Let the pybind11 authors know py_extension works better.
# pybind_extension(
#     name = "extmodule",
#     srcs = ["extmodule.cc"],
#     # copts = ["-DPYTHON_EXECUTABLE=/usr/bin/python3"],
#     deps = [
#         ":parser",
#     ],
# )

py_extension(
    name = "extmodule",
    srcs = ["extmodule.cc"],
    deps = [
        ":parser",
        ":ledger",
        "//beancount/ccore:data_cc_proto",
        "//beancount/ccore:datapy",
        "@local_config_python//:python_headers",
        "@pybind11",
    ],
)

py_test(
    name = "extmodule_test",
    srcs = ["extmodule_test.py"],
    data = ["//examples:example"],
    deps = [
        ":extmodule",
        "//beancount/parser:printer",
        "//beancount/core:number",
    ],
)

#-------------------------------------------------------------------------------

cc_library(
    name = "test_utils",
    srcs = [
        "test_utils.cc"
    ],
    hdrs = [
        "test_utils.h"
    ],
    deps = [
        ":ledger",
        "//beancount/ccore:data_cc_proto",
        "@com_google_absl//absl/strings",
        "@com_google_protobuf//:protobuf",
    ],
)

cc_test(
    name = "test_utils_test",
    srcs = ["test_utils_test.cc"],
    deps = [
        "//beancount/ccore:data_cc_proto",
        ":test_utils",
        "@com_google_absl//absl/strings",
        "@com_google_googletest//:gtest_main",
        "@com_google_protobuf//:protobuf",
    ],
)
