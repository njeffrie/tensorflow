load("//tensorflow:tensorflow.bzl", "tf_cc_binary", "tf_cc_test", "tf_py_test")
load("//tensorflow:tensorflow.bzl", "tf_python_pybind_extension")
load("//tensorflow/compiler/mlir/tfr:build_defs.bzl", "gen_op_libraries")
load(
    "//third_party/mlir:tblgen.bzl",
    "gentbl",
)
load("//tensorflow/compiler/mlir:glob_lit_test.bzl", "glob_lit_tests")

package(
    default_visibility = [
        ":friends",
    ],
    licenses = ["notice"],  # Apache 2.0
)

package_group(
    name = "friends",
    includes = ["//third_party/mlir:subpackages"],
    packages = [
        "//learning/brain/experimental/mlir/tfr/...",
        "//tensorflow/c/...",
        "//tensorflow/compiler/...",
    ],
)

filegroup(
    name = "tfr_ops_td_files",
    srcs = [
        "ir/tfr_ops.td",
        "//tensorflow/compiler/mlir/tensorflow:ir/tf_op_base.td",
        "//tensorflow/compiler/mlir/tensorflow:ir/tf_op_interfaces.td",
        "@llvm-project//mlir:OpBaseTdFiles",
        "@llvm-project//mlir:include/mlir/Dialect/Shape/IR/ShapeBase.td",
        "@llvm-project//mlir:include/mlir/Dialect/Shape/IR/ShapeOps.td",
        "@llvm-project//mlir:include/mlir/IR/SymbolInterfaces.td",
        "@llvm-project//mlir:include/mlir/Interfaces/CallInterfaces.td",
        "@llvm-project//mlir:include/mlir/Interfaces/ControlFlowInterfaces.td",
        "@llvm-project//mlir:include/mlir/Interfaces/SideEffectInterfaces.td",
    ],
)

gentbl(
    name = "tfr_ops_inc_gen",
    tbl_outs = [
        (
            "-gen-op-decls",
            "ir/tfr_ops.h.inc",
        ),
        (
            "-gen-op-defs",
            "ir/tfr_ops.cc.inc",
        ),
    ],
    tblgen = "@llvm-project//mlir:mlir-tblgen",
    td_file = "ir/tfr_ops.td",
    td_srcs = [
        ":tfr_ops_td_files",
    ],
)

cc_library(
    name = "tfr",
    srcs = [
        "ir/tfr_ops.cc",
        "ir/tfr_ops.cc.inc",
    ],
    hdrs = [
        "ir/tfr_ops.h",
        "ir/tfr_ops.h.inc",
        "ir/tfr_types.h",
    ],
    deps = [
        ":tfr_ops_inc_gen",
        "//tensorflow/compiler/mlir/tensorflow",
        "//tensorflow/compiler/mlir/tensorflow:tensorflow_attributes",
        "//tensorflow/compiler/mlir/tensorflow:tensorflow_types",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:ControlFlowInterfaces",
        "@llvm-project//mlir:Dialect",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Shape",
        "@llvm-project//mlir:SideEffects",
        "@llvm-project//mlir:StandardOps",
        "@llvm-project//mlir:Support",
        "@llvm-project//mlir:TransformUtils",
    ],
)

cc_library(
    name = "utils",
    srcs = [
        "utils/utils.cc",
    ],
    hdrs = [
        "utils/utils.h",
    ],
    deps = [
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:Support",
    ],
)

cc_library(
    name = "passes",
    srcs = [
        "passes/canonicalize.cc",
        "passes/decompose.cc",
        "passes/raise_to_tf.cc",
    ],
    hdrs = [
        "passes/passes.h",
    ],
    deps = [
        ":tfr",
        ":utils",
        "//tensorflow/compiler/mlir/tensorflow",
        "@com_google_absl//absl/memory",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Pass",
        "@llvm-project//mlir:QuantOps",
        "@llvm-project//mlir:SCFDialect",
        "@llvm-project//mlir:SCFToStandard",
        "@llvm-project//mlir:StandardOps",
        "@llvm-project//mlir:Support",
        "@llvm-project//mlir:TransformUtils",
    ],
    alwayslink = 1,
)

tf_cc_binary(
    name = "tfr-opt",
    srcs = ["passes/tfr_opt.cc"],
    deps = [
        ":passes",
        ":tfr",
        "//tensorflow/compiler/mlir:init_mlir",
        "//tensorflow/compiler/mlir:passes",
        "//tensorflow/compiler/mlir/tensorflow",
        "//tensorflow/compiler/mlir/tensorflow:tensorflow_passes",
        "@llvm-project//mlir:AllPassesAndDialectsNoRegistration",
        "@llvm-project//mlir:MlirOptLib",
        "@llvm-project//mlir:SCFDialect",
        "@llvm-project//mlir:Shape",
        "@llvm-project//mlir:StandardOps",
    ],
)

glob_lit_tests(
    data = [
        ":test_utilities",
        "@llvm-project//mlir:run_lit.sh",
    ],
    driver = "//tensorflow/compiler/mlir:run_lit.sh",
    test_file_exts = ["mlir"],
)

# Bundle together all of the test utilities that are used by tests.
filegroup(
    name = "test_utilities",
    testonly = True,
    data = [
        "//tensorflow/compiler/mlir/tfr:tfr-opt",
        "@llvm-project//llvm:FileCheck",
        "@llvm-project//llvm:not",
    ],
)

cc_library(
    name = "tfr_decompose_ctx",
    srcs = ["integration/tfr_decompose_ctx.cc"],
    hdrs = ["integration/tfr_decompose_ctx.h"],
    deps = [
        ":passes",
        ":tfr",
        "//tensorflow/compiler/mlir/tensorflow",
        "//tensorflow/compiler/mlir/tensorflow:convert_attr",
        "//tensorflow/compiler/mlir/tensorflow:convert_graphdef",
        "//tensorflow/compiler/mlir/tensorflow:convert_type",
        "//tensorflow/compiler/mlir/tensorflow:mlir_roundtrip_flags",
        "//tensorflow/compiler/mlir/tensorflow:tensorflow_passes",
        "//tensorflow/core:lib",
        "//tensorflow/core:lib_internal",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/stream_executor/lib",
        "@com_google_absl//absl/strings",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Parser",
        "@llvm-project//mlir:Pass",
        "@llvm-project//mlir:SCFDialect",
        "@llvm-project//mlir:Shape",
        "@llvm-project//mlir:StandardOps",
        "@llvm-project//mlir:TransformUtils",
    ],
)

tf_cc_test(
    name = "tfr_decompose_ctx_test",
    srcs = ["integration/tfr_decompose_ctx_test.cc"],
    deps = [
        ":tfr_decompose_ctx",
        "//tensorflow/compiler/xla:test",
        "//tensorflow/core:framework",
        "//tensorflow/core:protos_all_cc",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/stream_executor/lib",
        "@com_google_absl//absl/types:span",
        "@llvm-project//mlir:AllPassesAndDialectsNoRegistration",
        "@llvm-project//mlir:IR",
    ],
)

cc_library(
    name = "graph_decompose_pass",
    srcs = ["integration/graph_decompose_pass.cc"],
    hdrs = ["integration/graph_decompose_pass.h"],
    deps = [
        ":tfr_decompose_ctx",
        "//tensorflow/compiler/mlir:mlir_graph_optimization_pass",
        "//tensorflow/stream_executor/lib",
        "@llvm-project//mlir:IR",
    ],
    alwayslink = 1,
)

tf_py_test(
    name = "graph_decompose_test",
    size = "small",
    srcs = ["integration/graph_decompose_test.py"],
    data = ["//tensorflow/compiler/mlir/tfr/resources:decomposition_lib"],
    python_version = "PY3",
    tags = [
        "no_pip",
        "no_windows",  # TODO(b/170752141)
        "nomac",  # TODO(b/170752141)
    ],
    deps = [
        "//tensorflow/compiler/mlir/tfr/resources:composite_ops",
        "//tensorflow/python/eager:def_function",
    ],
)

cc_library(
    name = "node_expansion_pass",
    srcs = ["integration/node_expansion_pass.cc"],
    hdrs = ["integration/node_expansion_pass.h"],
    deps = [
        ":tfr_decompose_ctx",
        "//tensorflow/core/common_runtime/eager:core",
        "//tensorflow/core/common_runtime/eager:eager_op_rewrite_registry",
        "//tensorflow/stream_executor/lib",
        "@com_google_absl//absl/strings",
    ],
    alwayslink = 1,
)

tf_py_test(
    name = "node_expansion_test",
    size = "small",
    srcs = ["integration/node_expansion_test.py"],
    data = ["//tensorflow/compiler/mlir/tfr/resources:decomposition_lib"],
    python_version = "PY3",
    tags = [
        "no_pip",
        "no_windows",  # TODO(b/170752141)
        "nomac",  # TODO(b/170752141)
    ],
    deps = [
        "//tensorflow/compiler/mlir/tfr/resources:composite_ops",
    ],
)

tf_python_pybind_extension(
    name = "tfr_wrapper",
    srcs = ["python/tfr_wrapper.cc"],
    module_name = "tfr_wrapper",
    visibility = ["//visibility:public"],
    deps = [
        "//tensorflow/compiler/mlir/tensorflow",
        "//tensorflow/compiler/mlir/tfr",
        "//tensorflow/python:pybind11_lib",
        "//tensorflow/python:pybind11_status",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Parser",
        "@llvm-project//mlir:SCFDialect",
        "@llvm-project//mlir:Shape",
        "@llvm-project//mlir:StandardOps",
        "@pybind11",
    ],
)

py_library(
    name = "composite",
    srcs = ["python/composite.py"],
    srcs_version = "PY2AND3",
)

py_library(
    name = "tfr_gen",
    srcs = ["python/tfr_gen.py"],
    srcs_version = "PY2AND3",
    deps = [
        "//tensorflow:tensorflow_py",
        "//tensorflow/compiler/mlir/tfr:tfr_wrapper",
    ],
)

tf_py_test(
    name = "tfr_gen_test",
    size = "small",
    srcs = ["python/tfr_gen_test.py"],
    python_version = "PY3",
    tags = ["no_pip"],
    deps = [
        ":composite",
        ":tfr_gen",
        "//tensorflow/compiler/mlir/python/mlir_wrapper:filecheck_wrapper",
        "//tensorflow/compiler/mlir/tfr/resources:test_ops",
        "//tensorflow/python:array_ops",
        "//tensorflow/python:client_testlib",
        "//tensorflow/python:math_ops",
    ],
)

py_library(
    name = "op_reg_gen",
    srcs = ["python/op_reg_gen.py"],
    srcs_version = "PY2AND3",
    deps = [
        "//tensorflow:tensorflow_py",
    ],
)

py_test(
    name = "op_reg_gen_test",
    size = "small",
    srcs = ["python/op_reg_gen_test.py"],
    python_version = "PY3",
    srcs_version = "PY2AND3",
    deps = [
        ":composite",
        ":op_reg_gen",
        "//tensorflow/compiler/mlir/python/mlir_wrapper:filecheck_wrapper",
        "//tensorflow/python:client_testlib",
    ],
)

py_library(
    name = "test_utils",
    srcs = ["python/test_utils.py"],
    srcs_version = "PY2AND3",
    deps = [
        "//tensorflow/python:client_testlib",
    ],
)

gen_op_libraries(
    name = "one_op",
    src = "define_op_template.py",
)
