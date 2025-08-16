# CMakeList.txt: rbslib 的 CMake 项目，在此处包括源代码并定义
# 项目特定的逻辑。
#

cmake_minimum_required (VERSION 3.8)

# 如果支持，请为 MSVC 编译器启用热重载。
if (POLICY CMP0141)
    cmake_policy(SET CMP0141 NEW)
    set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<IF:$<AND:$<C_COMPILER_ID:MSVC>,$<CXX_COMPILER_ID:MSVC>>,$<$<CONFIG:Debug,RelWithDebInfo>:EditAndContinue>,$<$<CONFIG:Debug,RelWithDebInfo>:ProgramDatabase>>")
endif()

set(proj_name "minecraftspeedproxy")
project (${proj_name})

if(MSVC)
    # Windows上的编译器设置 (MSVC)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
endif()

# 设置 C++ 标准为 C++20
set(CMAKE_CXX_STANDARD 20)

# 获取 Git 版本号
EXECUTE_PROCESS(COMMAND git describe --tags --always --dirty=-dev
                TIMEOUT 5
                OUTPUT_VARIABLE GIT_VERSION
                OUTPUT_STRIP_TRAILING_WHITESPACE
                )
MESSAGE(STATUS "building version ${GIT_VERSION}")
add_definitions(-DBUILD_VERSION=\"${GIT_VERSION}\")

if (${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    add_definitions(-DDEBUG)
else ()
    # -UDEV_CODE 表示取消 DEV_CODE 的定义
    add_definitions(-DRELEASE)
endif ()

# 查找头文件和源文件
file(GLOB RBS_LIB_HEADERS "rbslib/*.h" "rbslib/Windows/*.h")
file(GLOB RBS_LIB_SRC "rbslib/*.cpp" "rbslib/Windows/*.cpp")
file(GLOB JSON_LIB_HEADERS "json/*.h")
file(GLOB JSON_LIB_SRC "json/*.cpp")

# 添加可执行文件
add_executable (minecraftspeedproxy "main.cpp" 
                            "WhiteBlackList.cpp" "WhiteBlackList.h"
                            "config.cpp" "config.h"
                            "datapackage.cpp" "datapackage.h"
                            "logger.cpp" "logger.h"
                            "proxy.cpp" "proxy.h"
                            "helper.cpp" "helper.h"
                            "WebControl.cpp" "WebControl.h"
                            ${RBS_LIB_SRC} ${JSON_LIB_SRC} )

# 如果是Windows平台，链接ws2_32.lib
if (WIN32)
    target_link_libraries(minecraftspeedproxy ws2_32.lib)
endif()

# 设置 MinGW 编码相关选项
if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    add_compile_options(-finput-charset=UTF-8)
    add_compile_options(-fexec-charset=UTF-8)
    add_definitions(-DNO_WIN32_ANSI)
endif()

# 链接时强制控制台子系统（仅适用于Windows且MinGW）
if (WIN32 AND CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_link_options(minecraftspeedproxy PRIVATE -Wl,--subsystem,console)
endif()
