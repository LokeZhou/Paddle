cc_library(
  op_graph_view
  SRCS op_graph_view.cc
  DEPS op_handle_base)
cc_library(
  conditional_block_op_eager_deletion_pass
  SRCS conditional_block_op_eager_deletion_pass.cc
  DEPS conditional_block_op_helper graph_helper pass computation_op_handle)
cc_library(
  while_op_eager_deletion_pass
  SRCS while_op_eager_deletion_pass.cc
  DEPS while_op_helper graph_helper pass computation_op_handle)
cc_library(
  recurrent_op_eager_deletion_pass
  SRCS recurrent_op_eager_deletion_pass.cc
  DEPS recurrent_op_helper graph_helper pass computation_op_handle)
cc_library(
  reference_count_pass_helper
  SRCS reference_count_pass_helper.cc
  DEPS garbage_collector computation_op_handle var_handle)
cc_library(
  reference_count_pass
  SRCS reference_count_pass.cc
  DEPS computation_op_handle graph graph_helper pass op_graph_view
       reference_count_pass_helper)

set(EAGER_DELETETION_PASS_DEPS
    computation_op_handle
    eager_deletion_op_handle
    graph
    graph_helper
    pass
    conditional_block_op_eager_deletion_pass
    while_op_eager_deletion_pass
    recurrent_op_eager_deletion_pass
    reference_count_pass_helper)
if(WITH_CINN)
  cc_library(
    share_varinfo_into_cinn_pass
    SRCS share_varinfo_into_cinn_pass.cc
    DEPS pass enforce graph_helper computation_op_handle
         eager_deletion_op_handle cinn_compiler)
  cc_test(
    share_varinfo_into_cinn_pass_test
    SRCS share_varinfo_into_cinn_pass_test.cc
    DEPS share_varinfo_into_cinn_pass parallel_executor cinn_compiler
         elementwise_add_op mul_op cinn_launch_op)
  list(APPEND EAGER_DELETETION_PASS_DEPS share_varinfo_into_cinn_pass)
endif()

cc_library(
  eager_deletion_pass
  SRCS eager_deletion_pass.cc
  DEPS ${EAGER_DELETETION_PASS_DEPS})

cc_library(
  memory_reuse_pass
  SRCS memory_reuse_pass.cc
  DEPS computation_op_handle reference_count_pass_helper
       share_tensor_buffer_op_handle graph pass multi_devices_helper)

cc_library(
  buffer_shared_inplace_op_pass
  SRCS buffer_shared_inplace_op_pass.cc
  DEPS memory_reuse_pass executor_gc_helper)
cc_library(
  buffer_shared_cross_op_memory_reuse_pass
  SRCS buffer_shared_cross_op_memory_reuse_pass.cc
  DEPS memory_reuse_pass)

cc_library(
  inplace_addto_op_pass
  SRCS inplace_addto_op_pass.cc
  DEPS memory_reuse_pass)

cc_test(
  test_reference_count_pass_last_lived_ops
  SRCS test_reference_count_pass_last_lived_ops.cc
  DEPS parallel_executor elementwise_mul_op elementwise_add_op generated_op phi)
