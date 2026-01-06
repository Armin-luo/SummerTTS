cmake_minimum_required(VERSION 3.5)

project(SummerTTS VERSION 1.0.0 LANGUAGES CXX C)

# 设置C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 编译选项
set(CMAKE_CXX_FLAGS_RELEASE " -O3 -fopenmp -DNDEBUG ")
set(CMAKE_C_FLAGS_RELEASE " -O3 -fopenmp -DNDEBUG ")
set(CMAKE_CXX_FLAGS_DEBUG " -g -fopenmp ")
set(CMAKE_C_FLAGS_DEBUG " -g -fopenmp ")

# 如果没有指定构建类型，默认为Release
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# 查找OpenMP
find_package(OpenMP REQUIRED)

# 创建源文件列表（排除test/main.cpp）
set(SUMMERTTS_SOURCES
               ./src/tn/glog/src/demangle.cc
               ./src/tn/glog/src/logging.cc
               ./src/tn/glog/src/raw_logging.cc
               ./src/tn/glog/src/symbolize.cc
               ./src/tn/glog/src/utilities.cc
               ./src/tn/glog/src/vlog_is_on.cc
               ./src/tn/glog/src/signalhandler.cc
               ./src/tn/gflags/src/gflags.cc
               ./src/tn/gflags/src/gflags_reporting.cc
               ./src/tn/gflags/src/gflags_completions.cc
               ./src/tn/openfst/src/lib/compat.cc
               ./src/tn/openfst/src/lib/flags.cc
               ./src/tn/openfst/src/lib/fst.cc
               ./src/tn/openfst/src/lib/fst-types.cc
               ./src/tn/openfst/src/lib/mapped-file.cc
               ./src/tn/openfst/src/lib/properties.cc
               ./src/tn/openfst/src/lib/symbol-table.cc
               ./src/tn/openfst/src/lib/symbol-table-ops.cc
               ./src/tn/openfst/src/lib/util.cc
               ./src/tn/openfst/src/lib/weight.cc
               ./src/tn/processor.cc
               ./src/tn/token_parser.cc
               ./src/tn/utf8_string.cc
               ./src/engipa/EnglishText2Id.cpp
               ./src/engipa/InitIPASymbols.cpp
               ./src/engipa/alphabet.cpp
               ./src/engipa/ipa.cpp
               ./src/hz2py/hanzi2phoneid.cpp
               ./src/hz2py/Hanz2Piny.cpp
               ./src/hz2py/pinyinmap.cpp
               ./src/nn_op/nn_conv1d.cpp
               ./src/nn_op/nn_softmax.cpp
               ./src/nn_op/nn_layer_norm.cpp
               ./src/nn_op/nn_relu.cpp
               ./src/nn_op/nn_gelu.cpp
               ./src/nn_op/nn_tanh.cpp
               ./src/nn_op/nn_flip.cpp
               ./src/nn_op/nn_cumsum.cpp
               ./src/nn_op/nn_softplus.cpp
               ./src/nn_op/nn_clamp_min.cpp
               ./src/nn_op/nn_sigmoid.cpp
               ./src/nn_op/nn_conv1d_transposed.cpp
               ./src/nn_op/nn_leaky_relu.cpp
               ./src/platform/tts_file_io.cpp
               ./src/platform/tts_logger.cpp
               ./src/utils/utils.cpp
               ./src/modules/iStft.cpp
               ./src/modules/hann.cpp
               ./src/modules/attention_encoder.cpp
               ./src/modules/multi_head_attention.cpp
               ./src/modules/ffn.cpp
               ./src/modules/ConvFlow.cpp
               ./src/modules/DDSConv.cpp
               ./src/modules/ElementwiseAffine.cpp
               ./src/modules/random_gen.cpp
               ./src/modules/ResidualCouplingLayer.cpp
               ./src/modules/ResBlock1.cpp
               ./src/modules/WN.cpp
               ./src/modules/pqmf.cpp
               ./src/models/TextEncoder.cpp
               ./src/models/StochasticDurationPredictor.cpp
               ./src/models/FixDurationPredictor.cpp
               ./src/models/DurationPredictor_base.cpp
               ./src/models/ResidualCouplingBlock.cpp
               ./src/models/Generator_base.cpp
               ./src/models/Generator_hifigan.cpp
               ./src/models/Generator_MS.cpp
               ./src/models/Generator_Istft.cpp
               ./src/models/Generator_MBB.cpp
               ./src/models/SynthesizerTrn.cpp)

# 创建动态库
add_library(SummerTTS SHARED ${SUMMERTTS_SOURCES})

# 设置库的版本信息
set_target_properties(SummerTTS PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION 1
    PUBLIC_HEADER "include/SynthesizerTrn.h;include/tts_file_io.h;include/tts_logger.h;include/utils.h"
)

# 包含目录
target_include_directories(SummerTTS 
    PUBLIC 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/eigen-3.4.0
        ${CMAKE_CURRENT_SOURCE_DIR}/src/tn/header
        ${CMAKE_CURRENT_SOURCE_DIR}/src/header
)

# 链接OpenMP
target_link_libraries(SummerTTS PRIVATE OpenMP::OpenMP_CXX)

# 为库设置导出宏（可选）
target_compile_definitions(SummerTTS PRIVATE SUMMERTTS_EXPORTS)
target_compile_definitions(SummerTTS PUBLIC 
    $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:SUMMERTTS_STATIC_DEFINE>
)

# 安装规则
install(TARGETS SummerTTS
    EXPORT SummerTTSTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    PUBLIC_HEADER DESTINATION include
)

# 安装cmake配置文件
install(EXPORT SummerTTSTargets
    FILE SummerTTSTargets.cmake
    NAMESPACE SummerTTS::
    DESTINATION lib/cmake/SummerTTS
)

# 生成配置文件
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/SummerTTSConfigVersion.cmake"
    VERSION "${PROJECT_VERSION}"
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/SummerTTSConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/SummerTTSConfig.cmake"
    INSTALL_DESTINATION lib/cmake/SummerTTS
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/SummerTTSConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/SummerTTSConfigVersion.cmake"
    DESTINATION lib/cmake/SummerTTS
)

# 可选：构建测试程序
option(BUILD_TESTS "Build test programs" ON)
if(BUILD_TESTS)
    add_executable(tts_test ./test/main.cpp)
    target_link_libraries(tts_test PRIVATE SummerTTS)
    target_include_directories(tts_test PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/header)
    
    # 如果需要，可以安装测试程序
    # install(TARGETS tts_test DESTINATION bin)
endif()

# 可选：生成pkg-config文件
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/SummerTTS.pc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/SummerTTS.pc"
    @ONLY
)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/SummerTTS.pc"
    DESTINATION lib/pkgconfig
)
