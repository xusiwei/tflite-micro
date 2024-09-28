cmake_minimum_required(VERSION 3.18)

# project(tflite-micro)

# TFLM 顶层目录
if(NOT DEFINED TENSORFLOW_ROOT)
set(TENSORFLOW_ROOT ${CMAKE_CURRENT_LIST_DIR}/)
endif()
message(STATUS "TENSORFLOW_ROOT: ${TENSORFLOW_ROOT}")

set(RELATIVE_MAKEFILE_DIR tensorflow/lite/micro/tools/make)
set(MAKEFILE_DIR ${TENSORFLOW_ROOT}${RELATIVE_MAKEFILE_DIR})

# 下载目录，存放下载的第三方库
set(DOWNLOADS_DIR ${MAKEFILE_DIR}/downloads)

# 生成目录，存放生成的代码文件
set(GENERATED_SUBDIR gen)
set(GENERATED_DIR ${TENSORFLOW_ROOT}${GENERATED_SUBDIR})

# 公共头文件目录列表，用于将其传递到依赖TFLM 库的目标的 include_directories 中
set(TFLM_PUBLIC_INCLUDES
    ${TENSORFLOW_ROOT}
    ${GENERATED_DIR}
    ${DOWNLOADS_DIR}/flatbuffers/include
    ${DOWNLOADS_DIR}/gemmlowp
)
# message(STATUS "TFLM_PUBLIC_INCLUDES: ${TFLM_PUBLIC_INCLUDES}")

# 头文件目录列表
set(TFLM_INCLUDES
    .
    ${DOWNLOADS_DIR}
    ${DOWNLOADS_DIR}/gemmlowp
    ${DOWNLOADS_DIR}/flatbuffers/include
    ${DOWNLOADS_DIR}/kissfft
    ${DOWNLOADS_DIR}/ruy
    ${TFLM_PUBLIC_INCLUDES}
)

# 编译警告相关选项
set(CC_WARNINGS 
    -Wsign-compare
    -Wdouble-promotion
    -Wunused-variable
    -Wunused-function
    -Wswitch
    -Wvla
    -Wall
    -Wextra
    -Wmissing-field-initializers
    -Wstrict-aliasing
    -Wno-unused-parameter
)

# 不是 gcc 系列编译器，则添加 -Wshadow
if (NOT CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    list(APPEND CC_WARNINGS -Wshadow)
endif()

# C/C++通用 编译选项
set(COMMON_FLAGS
    -Werror
    -fno-unwind-tables
    -ffunction-sections
    -fdata-sections
    -fmessage-length=0
    # -DTF_LITE_STATIC_MEMORY
    -DTF_LITE_DISABLE_X86_NEON
)

# C++编译选项
set(CXXFLAGS
    -std=c++17
    -fno-rtti
    -fno-exceptions
    -fno-threadsafe-statics
    -Wnon-virtual-dtor
    ${COMMON_FLAGS}
)

# C 编译选项
list(APPEND CXXFLAGS
    -Wimplicit-function-declaration
    # -std=c17
)

# 如果当前构建类型是 release，则添加 -DNDEBUG
if (CMAKE_BUILD_TYPE STREQUAL "Release")
    message(STATUS "TFLite: Release build")
    list(APPEND CXXFLAGS -DNDEBUG) #  -DTF_LITE_STRIP_ERROR_STRINGS
else()
    message(STATUS "TFLite: ${CMAKE_BUILD_TYPE} build")
    list(APPEND CXXFLAGS -DDEBUG -O0) #  -DTF_LITE_STRIP_ERROR_STRINGS
endif()


# MICROLITE_BENCHMARK_SRCS := ...
set(MICROLITE_BENCHMARK_SRCS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/benchmarks/*benchmark.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/tools/benchmarking/*benchmark.cc
)

set(MICROLITE_TEST_SRCS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/fake_micro_context_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/flatbuffer_utils_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_arena_threshold_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_helpers_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_allocator_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_allocation_info_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_interpreter_context_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_log_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_interpreter_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_mutable_op_resolver_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_resource_variable_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_time_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_utils_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/recording_micro_allocator_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/arena_allocator/non_persistent_arena_buffer_allocator_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/arena_allocator/persistent_arena_buffer_allocator_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/arena_allocator/recording_single_arena_buffer_allocator_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/arena_allocator/single_arena_buffer_allocator_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/testing_helpers_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_planner/greedy_memory_planner_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_planner/linear_memory_planner_test.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_planner/non_persistent_buffer_planner_shim_test.cc
)

# MICROLITE_CC_KERNEL_SRCS := ...
set(MICROLITE_CC_KERNEL_SRCS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/activations.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/activations_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/add.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/add_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/add_n.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/arg_min_max.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/assign_variable.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/batch_matmul.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/batch_to_space_nd.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/broadcast_args.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/broadcast_to.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/call_once.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/cast.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/ceil.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/circular_buffer.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/circular_buffer_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/comparisons.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/concatenation.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/conv.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/conv_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/cumsum.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/depth_to_space.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/depthwise_conv.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/depthwise_conv_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/dequantize.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/dequantize_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/detection_postprocess.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/div.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/elementwise.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/elu.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/embedding_lookup.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/ethosu.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/exp.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/expand_dims.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/fill.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/floor.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/floor_div.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/floor_mod.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/fully_connected.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/fully_connected_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/gather.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/gather_nd.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/hard_swish.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/hard_swish_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/if.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/kernel_runner.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/kernel_util.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/l2norm.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/l2_pool_2d.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/leaky_relu.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/leaky_relu_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/logical.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/logical_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/logistic.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/logistic_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/log_softmax.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/lstm_eval.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/lstm_eval_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/maximum_minimum.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/micro_tensor_utils.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/mirror_pad.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/mul.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/mul_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/neg.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/pack.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/pad.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/pooling.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/pooling_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/prelu.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/prelu_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/quantize.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/quantize_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/read_variable.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/reduce.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/reduce_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/reshape.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/reshape_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/resize_bilinear.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/resize_nearest_neighbor.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/round.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/select.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/shape.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/slice.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/softmax.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/softmax_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/space_to_batch_nd.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/space_to_depth.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/split.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/split_v.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/squared_difference.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/squeeze.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/strided_slice.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/strided_slice_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/sub.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/sub_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/svdf.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/svdf_common.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/tanh.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/transpose.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/transpose_conv.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/unidirectional_sequence_lstm.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/unpack.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/var_handle.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/while.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/zeros_like.cc
)

# MICROLITE_CC_SIGNAL_KERNEL_SRCS := ...
set(MICROLITE_CC_SIGNAL_KERNEL_SRCS
    ${TENSORFLOW_ROOT}signal/micro/kernels/delay.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/energy.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/fft_auto_scale_kernel.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/fft_auto_scale_common.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/filter_bank.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/filter_bank_log.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/filter_bank_square_root.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/filter_bank_square_root_common.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/filter_bank_spectral_subtraction.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/framer.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/irfft.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/rfft.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/stacker.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/overlap_add.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/pcan.cc
    ${TENSORFLOW_ROOT}signal/micro/kernels/window.cc
    ${TENSORFLOW_ROOT}signal/src/circular_buffer.cc
    ${TENSORFLOW_ROOT}signal/src/energy.cc
    ${TENSORFLOW_ROOT}signal/src/fft_auto_scale.cc
    ${TENSORFLOW_ROOT}signal/src/filter_bank.cc
    ${TENSORFLOW_ROOT}signal/src/filter_bank_log.cc
    ${TENSORFLOW_ROOT}signal/src/filter_bank_square_root.cc
    ${TENSORFLOW_ROOT}signal/src/filter_bank_spectral_subtraction.cc
    ${TENSORFLOW_ROOT}signal/src/irfft_float.cc
    ${TENSORFLOW_ROOT}signal/src/irfft_int16.cc
    ${TENSORFLOW_ROOT}signal/src/irfft_int32.cc
    ${TENSORFLOW_ROOT}signal/src/log.cc
    ${TENSORFLOW_ROOT}signal/src/max_abs.cc
    ${TENSORFLOW_ROOT}signal/src/msb_32.cc
    ${TENSORFLOW_ROOT}signal/src/msb_64.cc
    ${TENSORFLOW_ROOT}signal/src/overlap_add.cc
    ${TENSORFLOW_ROOT}signal/src/pcan_argc_fixed.cc
    ${TENSORFLOW_ROOT}signal/src/rfft_float.cc
    ${TENSORFLOW_ROOT}signal/src/rfft_int16.cc
    ${TENSORFLOW_ROOT}signal/src/rfft_int32.cc
    ${TENSORFLOW_ROOT}signal/src/square_root_32.cc
    ${TENSORFLOW_ROOT}signal/src/square_root_64.cc
    ${TENSORFLOW_ROOT}signal/src/window.cc
)

# TFL_CC_SRCS := ...
file(GLOB_RECURSE TFL_CC_SRCS
    ${TENSORFLOW_ROOT}tensorflow/*.cc
    ${TENSORFLOW_ROOT}tensorflow/*.c
)

# 排除 ${TENSORFLOW_ROOT}tensorflow/lite/experimental/*.*
list(FILTER TFL_CC_SRCS EXCLUDE REGEX "${TENSORFLOW_ROOT}tensorflow/lite/experimental/.*")

# 排除 ${TENSORFLOW_ROOT}tensorflow/lite/micro/*.*
list(FILTER TFL_CC_SRCS EXCLUDE REGEX "${TENSORFLOW_ROOT}tensorflow/lite/micro/.*")
# message(STATUS "TFL_CC_SRCS: ${TFL_CC_SRCS}")

# TFL_CC_HDRS := ...
file(GLOB_RECURSE TFL_CC_HDRS
    ${TENSORFLOW_ROOT}tensorflow/*.h
)

# 排除 ${TENSORFLOW_ROOT}tensorflow/lite/experimental/.*
list(FILTER TFL_CC_HDRS EXCLUDE REGEX "${TENSORFLOW_ROOT}tensorflow/lite/experimental/.*")

# 排除 ${TENSORFLOW_ROOT}tensorflow/lite/micro/.*
list(FILTER TFL_CC_HDRS EXCLUDE REGEX "${TENSORFLOW_ROOT}tensorflow/lite/micro/.*")
# message(STATUS "TFL_CC_HDRS: ${TFL_CC_HDRS}")

file(GLOB MICROLITE_CC_BASE_SRCS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/*.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/arena_allocator/*.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_planner/*.cc
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/tflite_bridge/*.cc
)

################################################################################
# 过滤掉 tensorflow/lite/micro/micro_time.cc
list(FILTER MICROLITE_CC_BASE_SRCS EXCLUDE REGEX "${TENSORFLOW_ROOT}tensorflow/lite/micro/micro_time.cc")
################################################################################

# TFL_CC_SRCS 追加到 MICROLITE_CC_BASE_SRCS
list(APPEND MICROLITE_CC_BASE_SRCS ${TFL_CC_SRCS})

file(GLOB MICROLITE_CC_HDRS
    ${TENSORFLOW_ROOT}signal/micro/kernels/*.h
    ${TENSORFLOW_ROOT}signal/src/*.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/*.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/benchmarks/*model_data.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/kernels/*.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/arena_allocator/*.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/memory_planner/*.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/tflite_bridge/*.h
    ${TENSORFLOW_ROOT}LICENSE
)

# TFL_CC_HDRS 追加到 MICROLITE_CC_HDRS
list(APPEND MICROLITE_CC_HDRS ${TFL_CC_HDRS})

# MICROLITE_CC_SRCS := $(filter-out $(MICROLITE_TEST_SRCS), $(MICROLITE_CC_BASE_SRCS))
set(MICROLITE_CC_SRCS ${MICROLITE_CC_BASE_SRCS})
list(REMOVE_ITEM MICROLITE_CC_SRCS ${MICROLITE_TEST_SRCS})

# MICROLITE_CC_SRCS := $(filter-out $(MICROLITE_BENCHMARK_SRCS), $(MICROLITE_CC_SRCS))
set(MICROLITE_CC_SRCS ${MICROLITE_CC_SRCS})
list(REMOVE_ITEM MICROLITE_CC_SRCS ${MICROLITE_BENCHMARK_SRCS})

# MICROLITE_CC_KERNEL_SRCS += $(MICROLITE_CC_SIGNAL_KERNEL_SRCS)
list(APPEND MICROLITE_CC_KERNEL_SRCS ${MICROLITE_CC_SIGNAL_KERNEL_SRCS})

# TFLM 库文件名称
set(MICROLITE_LIB_NAME "tflite-micro")

set(TFLM_SRCS
    ${MICROLITE_CC_SRCS}
    ${MICROLITE_CC_HDRS}
    ${MICROLITE_CC_KERNEL_SRCS}
)

# declare tflite-micro static library
add_library(${MICROLITE_LIB_NAME} STATIC
    ${TFLM_SRCS}
)

target_include_directories(${MICROLITE_LIB_NAME}
    PRIVATE ${TFLM_INCLUDES}
    PUBLIC ${TFLM_PUBLIC_INCLUDES}
)

target_compile_options(${MICROLITE_LIB_NAME}
    # PRIVATE ${CXXFLAGS}
    # PRIVATE ${CCFLAGS}
    PRIVATE "-Ofast"
    PRIVATE ${CC_WARNINGS}
)

set(TFLM_BENCHMARK_SRCS)

# Arguments are:
# 0 - Name of target.
# 1 - C/C++ source files
# 2 - C/C++ header files
# 3 - Model sources and model test inputs in.tflite, .wav, .bmp or .csv format.
function(microlite_test NAME SRCS HDRS INPUTS)
    # message(STATUS "microlite_test: NAME=${NAME}")
    # message(STATUS "microlite_test: SRCS=${SRCS}")
    # message(STATUS "microlite_test: HDRS=${HDRS}")
    # message(STATUS "microlite_test: INPUTS=${INPUTS}")
    set(${NAME}_LOCAL_SRCS ${SRCS} ${HDRS})
    # message(STATUS "${NAME}_LOCAL_SRCS: ${${NAME}_LOCAL_SRCS}")
    set(${NAME}_GENERATED_SRCS)
    set(${NAME}_GENERATE_TARGETS)

    # 处理 INPUTS 列表， 生成数组代码文件
    foreach(INPUT ${INPUTS})
        message(STATUS "Generating C++ for: ${INPUT}...")

        # 获取 INPUT 的文件名称， 以及不含扩展名的名称
        get_filename_component(INPUT_NAME ${INPUT} NAME)
        get_filename_component(INPUT_BASE ${INPUT} NAME_WE)

        # 从 ${TENSORFLOW_ROOT} 到 ${INPUT} 的相对路径
        file(RELATIVE_PATH INPUT_RELPATH ${TENSORFLOW_ROOT} ${INPUT})

        # 获取 INPUT_RELPATH 的目录部分
        get_filename_component(INPUT_SUBDIR ${INPUT_RELPATH} DIRECTORY)

        # 根据输入文件后缀名，生成不同的输出文件后缀，这部分和 generate_cc_arrays.py 中的代码对应
        if (INPUT_NAME MATCHES "\\.tflite$")
            set(OUTPUT_NAME_EXT "_model_data")
        elseif (INPUT_NAME MATCHES "\\.bmp$")
            set(OUTPUT_NAME_EXT "_image_data")
        elseif (INPUT_NAME MATCHES "\\.wav$")
            set(OUTPUT_NAME_EXT "_audio_data")
        elseif (INPUT_NAME MATCHES "\\.csv$")
            set(OUTPUT_NAME_EXT "_test_data")
        elseif (INPUT_NAME MATCHES "\\.npy$")
            set(OUTPUT_NAME_EXT "_test_data")
        endif()

        # 生成的源文件路径
        set(GENERATED_SRC_PATH ${GENERATED_DIR}/${INPUT_SUBDIR}/${INPUT_BASE}${OUTPUT_NAME_EXT}.cc)
        message(STATUS "GENERATED_SRC_PATH: ${GENERATED_SRC_PATH}")

        # 添加自定义命令，用于生成数组代码文件
        set(GENERATE_CC_SCRIPT tensorflow/lite/micro/tools/generate_cc_arrays.py)
        message(STATUS "Command: python3 ${GENERATE_CC_SCRIPT} ${GENERATED_SUBDIR} ${INPUT_RELPATH}")
        add_custom_command(
            OUTPUT ${GENERATED_SRC_PATH}
            COMMAND python3 ${GENERATE_CC_SCRIPT} ${GENERATED_SUBDIR} ${INPUT_RELPATH}
            WORKING_DIRECTORY ${TENSORFLOW_ROOT}
        )

        # 生成的源文件 添加到 ${NAME}_GENERATED_SRCS 列表
        list(APPEND ${NAME}_GENERATED_SRCS ${GENERATED_SRC_PATH})

        # 自定义目标名称
        set(GENERATE_TARGET_NAME ${NAME}_CODEGEN_${INPUT_BASE})
        # message(STATUS "GENERATE_TARGET_NAME: ${GENERATE_TARGET_NAME}")

        # 添加自定义目标，用于提供依赖关系
        add_custom_target(
            ${GENERATE_TARGET_NAME}
            DEPENDS ${GENERATED_SRC_PATH}
        )

        list(APPEND ${NAME}_GENERATE_TARGETS ${GENERATE_TARGET_NAME})
    endforeach()
    message(STATUS "${NAME}_GENERATED_SRCS: ${${NAME}_GENERATED_SRCS}")
    message(STATUS "${NAME}_GENERATE_TARGETS: ${${NAME}_GENERATE_TARGETS}")

    # 添加静态库目标
    add_library(${NAME} STATIC
        ${${NAME}_LOCAL_SRCS}
        ${${NAME}_GENERATED_SRCS}
    )
    add_dependencies(${NAME} ${${NAME}_GENERATE_TARGETS})
    list(APPEND TFLM_BENCHMARK_SRCS ${${NAME}_LOCAL_SRCS})

    # 重命名main函数为 ${NAME}_main
    target_compile_definitions(${NAME} PRIVATE -Dmain=${NAME}_main)

    # 链接TFLM库
    target_link_libraries(${NAME} PUBLIC ${MICROLITE_LIB_NAME})
endfunction()

# KEYWORD_BENCHMARK_SRCS := 
set(KEYWORD_BENCHMARK_SRCS ${TENSORFLOW_ROOT}tensorflow/lite/micro/benchmarks/keyword_benchmark.cc)

# KEYWORD_BENCHMARK_GENERATOR_INPUTS := 
set(KEYWORD_BENCHMARK_GENERATOR_INPUTS ${TENSORFLOW_ROOT}tensorflow/lite/micro/models/keyword_scrambled.tflite)

# KEYWORD_BENCHMARK_HDRS := 
set(KEYWORD_BENCHMARK_HDRS ${TENSORFLOW_ROOT}tensorflow/lite/micro/benchmarks/micro_benchmark.h)

microlite_test(keyword_benchmark
    "${KEYWORD_BENCHMARK_SRCS}"
    "${KEYWORD_BENCHMARK_HDRS}"
    "${KEYWORD_BENCHMARK_GENERATOR_INPUTS}"
)

# PERSON_DETECTION_BENCHMARK_SRCS := 
set(PERSON_DETECTION_BENCHMARK_SRCS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/benchmarks/person_detection_benchmark.cc
)

# PERSON_DETECTION_BENCHMARK_HDRS := 
set(PERSON_DETECTION_BENCHMARK_HDRS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/examples/person_detection/model_settings.h
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/benchmarks/micro_benchmark.h
)

# PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS := 
set(PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/examples/person_detection/testdata/person.bmp
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/examples/person_detection/testdata/no_person.bmp
)

# PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS += 
list(APPEND PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS
    ${TENSORFLOW_ROOT}tensorflow/lite/micro/models/person_detect.tflite
)

microlite_test(person_detection_benchmark
    "${PERSON_DETECTION_BENCHMARK_SRCS}"
    "${PERSON_DETECTION_BENCHMARK_HDRS}"
    "${PERSON_DETECTION_BENCHMARK_GENERATOR_INPUTS}"
)
