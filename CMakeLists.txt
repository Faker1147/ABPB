#
# ABPB, based on PODBot by Count Floyd
# Maintained by YaPB Team <yapb@jeefo.net>
#
# SPDX-License-Identifier: Unlicense
#
# local builds (no zig needed for native):
#   cmake --preset release && cmake --build --preset release
# cross builds need zig on path, one preset per target, e.g.:
#   cmake --preset ci-linux-x86 && cmake --build --preset ci-linux-x86
# see cmakepresets.json for the full matrix (mirrors ci)
#
cmake_minimum_required(VERSION 3.22)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

project(abpb VERSION 4.3.3 DESCRIPTION "Artificial Intelligence Bot" HOMEPAGE_URL "https://yapb.github.io/" LANGUAGES C CXX)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

option(ABPB_64BIT "Build as 64-bit binary" OFF)
option(ABPB_NATIVE "Configure compiler for a native machine build" OFF)
option(ABPB_WINXP "Configure MSVC build for Windows XP compatible binary" OFF)
option(ABPB_NOSIMD "Disable all SIMD or NEON optimizations" OFF)
option(ABPB_STATIC_LINKENT "Use predefined entity link list" OFF)
option(ABPB_LTO "Enable link-time optimization" ON)
option(ABPB_SANITIZE "Enable sanitizers, e.g. address or address,undefined (dev builds)" "")
option(ABPB_WITH_TLS "Enable HTTPS support via vendored mbedtls" ON)
option(ABPB_BUILD_TESTS "Build test host (fake engine harness, native gcc/clang only, requires -DABPB_64BIT=ON)" OFF)
option(ABPB_WITH_AMXX "Build AMX Mod X module (i386 only)" ON)

# android (ndk toolchain) fixes the abi for us: follow the pointer size instead of
# the manual flag, so the parent never has to force ABPB_64BIT before adding us
if(ANDROID)
  if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(ABPB_64BIT ON CACHE BOOL "Build as 64-bit binary" FORCE)
  else()
    set(ABPB_64BIT OFF CACHE BOOL "Build as 64-bit binary" FORCE)
  endif()
endif()

# classic headers only, no c++20 modules
set(CMAKE_CXX_SCAN_FOR_MODULES OFF)

if(ZIG_TARGET)
  set(IS_ZIG TRUE)
else()
  set(IS_ZIG FALSE)
endif()

set(_PROC "${CMAKE_SYSTEM_PROCESSOR}")
if(_PROC MATCHES "^(arm64|ARM64)$")
  set(_PROC "aarch64")
endif()

if(_PROC MATCHES "^(arm|ppc|ppc64|riscv)")
  set(CPU_NON_X86 TRUE)
else()
  set(CPU_NON_X86 FALSE)
endif()

if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU") AND NOT MSVC)
  set(CLANG_OR_GCC TRUE)
else()
  set(CLANG_OR_GCC FALSE)
endif()

if(_PROC STREQUAL "aarch64")
  set(LIB_SUFFIX "_arm64")
elseif(_PROC MATCHES "^riscv")
  set(LIB_SUFFIX "_riscv64d")
elseif(ABPB_64BIT)
  set(LIB_SUFFIX "_amd64")
else()
  set(LIB_SUFFIX "")
endif()

set(TARGET_NAME "ABPB${LIB_SUFFIX}")

include(ABPBVersion)
include(ABPBFlags)

set(SRC
  "src/analyze.cpp"
  "src/behavior.cpp"
  "src/buying.cpp"
  "src/chatlib.cpp"
  "src/clients.cpp"
  "src/combat.cpp"
  "src/config.cpp"
  "src/control.cpp"
  "src/debug.cpp"
  "src/engine.cpp"
  "src/fakeping.cpp"
  "src/funmode.cpp"
  "src/graph.cpp"
  "src/hooks.cpp"
  "src/linkage.cpp"
  "src/manager.cpp"
  "src/message.cpp"
  "src/module.cpp"
  "src/navigate.cpp"
  "src/planner.cpp"
  "src/practice.cpp"
  "src/radio.cpp"
  "src/rehlds.cpp"
  "src/sounds.cpp"
  "src/storage.cpp"
  "src/support.cpp"
  "src/tasks.cpp"
  "src/vision.cpp"
  "src/vistable.cpp"
  "src/weapons.cpp"
  "src/human_copycat.cpp"
  "src/human_chat.cpp"
  "src/human_explore.cpp"
  "src/human_final.cpp"
  "src/human_aim.cpp"
  "src/copycat/copycat_aim.cpp"
  "src/copycat/copycat_final.cpp"
  "src/copycat/copycat_explore.cpp"
  "src/copycat/copycat_dialogue.cpp"
  "src/copycat/copycat_core.cpp"
  "src/copycat/copycat_lite.cpp"
  "src/copycat/copycat_routes.cpp"
  "src/copycat/copycat_learning.cpp"
)

# Keep imported, independently tested RealBot code outside unity/PCH: the
# original portable library uses standard C++ containers rather than ystl.
set_source_files_properties(src/human_copycat.cpp src/copycat/copycat_core.cpp
  src/copycat/copycat_lite.cpp src/copycat/copycat_routes.cpp src/copycat/copycat_learning.cpp src/copycat/copycat_explore.cpp src/copycat/copycat_dialogue.cpp src/copycat/copycat_final.cpp src/copycat/copycat_aim.cpp src/human_aim.cpp src/human_explore.cpp src/human_final.cpp src/human_chat.cpp PROPERTIES SKIP_UNITY_BUILD_INCLUSION ON
  SKIP_PRECOMPILE_HEADERS ON)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  set_source_files_properties(src/copycat/copycat_core.cpp src/copycat/copycat_lite.cpp src/copycat/copycat_routes.cpp src/copycat/copycat_learning.cpp src/copycat/copycat_explore.cpp src/copycat/copycat_dialogue.cpp src/copycat/copycat_final.cpp src/copycat/copycat_aim.cpp
    PROPERTIES COMPILE_OPTIONS "-w")
  set_source_files_properties(src/human_copycat.cpp src/human_explore.cpp src/human_final.cpp src/human_chat.cpp src/human_aim.cpp
    PROPERTIES COMPILE_OPTIONS "-Wno-effc++;-Wno-zero-as-null-pointer-constant")
endif()

if(CMAKE_SYSTEM_NAME MATCHES "Emscripten")
  add_executable(${PROJECT_NAME} ${SRC})
else()
  add_library(${PROJECT_NAME} SHARED ${SRC})
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
  OUTPUT_NAME "${TARGET_NAME}"
  CXX_STANDARD 20
  CXX_STANDARD_REQUIRED ON
  CXX_EXTENSIONS OFF
)

# android expects lib<name>.so, keep the default prefix (libabpb_android_<arch>.so)
if(NOT ANDROID)
  set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
endif()

abpb_configure_target(${PROJECT_NAME} STRICT DIST)

# unity + pch: every src/*.cpp starts with <abpb.h>, parse it once (off for Debug and Android)
if(NOT ANDROID AND (CMAKE_CONFIGURATION_TYPES OR NOT CMAKE_BUILD_TYPE MATCHES "Debug"))
  set_target_properties(${PROJECT_NAME} PROPERTIES UNITY_BUILD ON UNITY_BUILD_BATCH_SIZE 8)
  target_precompile_headers(${PROJECT_NAME} PRIVATE "${PROJECT_SOURCE_DIR}/inc/abpb.h")
endif()

# undecorated stdcall export alias, main library only: amxx has no such symbol
if(WIN32 AND IS_ZIG AND NOT ABPB_64BIT AND NOT CPU_NON_X86)
  target_link_options(${PROJECT_NAME} PRIVATE ${PROJECT_SOURCE_DIR}/ext/ldscripts/abpb-x86.def)
endif()

# windows resource with version info, main library only
if(WIN32 AND GIT_FOUND)
  if(IS_ZIG)
    # drive `zig rc` directly: the triple never leaves our control here
    add_custom_command(
      OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/abpb.res
      # pass -- so zig rc stops option parsing at the input file
      COMMAND zig rc -DVERSION_GENERATED -I${PROJECT_BINARY_DIR} /fo${CMAKE_CURRENT_BINARY_DIR}/abpb.res -- ${PROJECT_SOURCE_DIR}/vc/abpb.rc
      DEPENDS ${PROJECT_SOURCE_DIR}/vc/abpb.rc ${VERSION_H}
      COMMENT "Compiling Windows resource vc/abpb.rc"
      VERBATIM
    )
    target_sources(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/abpb.res)
    # ninja with zig drops .res from the link, pass it explicitly
    target_link_options(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/abpb.res)
  else()
    target_sources(${PROJECT_NAME} PRIVATE "vc/abpb.rc")
  endif()
endif()

# subprojects are C and don't inherit per-target -m32, pin their bitness
if(NOT ABPB_64BIT AND NOT IS_ZIG AND NOT ANDROID AND NOT _PROC STREQUAL "aarch64" AND NOT CPU_NON_X86)
  set(YSTL_32BIT ON)
else()
  set(YSTL_32BIT OFF)
endif()

# without mbedtls https requests cleanly report HttpOnly
if(ABPB_WITH_TLS AND NOT VITA AND NOT ABPB_WINXP AND NOT CMAKE_SYSTEM_NAME STREQUAL "Emscripten")
  if(POLICY CMP0218)
    cmake_policy(SET CMP0218 NEW)
  endif()

  add_subdirectory(ext/ystl/ext/mbedtls)
  target_link_libraries(${PROJECT_NAME} PRIVATE ystl_mbedtls)

  # vendored tls: -w shuts its warnings up (no unity: upstream relies on
  # per-tu MBEDTLS_ALLOW_PRIVATE_ACCESS, blobs don't compile)
  foreach(_tls_tgt mbedtls mbedx509 tfpsacrypto)
    if(TARGET ${_tls_tgt})
      target_compile_options(${_tls_tgt} PRIVATE -w)
    endif()
  endforeach()
  unset(_tls_tgt)
endif()

# no %n by design; winxp toolchain predates <stdbool.h>
if(NOT ABPB_WINXP)
  add_subdirectory(ext/ystl/ext/printf)
  target_link_libraries(${PROJECT_NAME} PRIVATE ystl_printf)
endif()

# amx mod x module (i386 linux/windows only), see ext/amxx
if(ABPB_WITH_AMXX AND NOT ABPB_64BIT AND NOT CPU_NON_X86 AND NOT _PROC STREQUAL "aarch64" AND NOT ANDROID)
  add_subdirectory(ext/amxx)
endif()

# legacy natives without a platform flags file (upstream android etc.)
if(ANDROID)
  target_link_libraries(${PROJECT_NAME} PRIVATE m dl log)
elseif(NOT VITA AND NOT WIN32 AND NOT APPLE AND NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
  target_link_libraries(${PROJECT_NAME} PRIVATE m dl pthread)
endif()

find_package(Python3 COMPONENTS Interpreter)

if(GIT_FOUND AND Python3_FOUND AND PROJECT_IS_TOP_LEVEL)
  add_custom_target(package
    COMMAND ${CMAKE_COMMAND} -E env "SOURCE_ROOT=${PROJECT_SOURCE_DIR}"
      ${Python3_EXECUTABLE} "${PROJECT_SOURCE_DIR}/tools/gen_package.py" "${PROJECT_VERSION}.${BUILD_COUNT}"
    WORKING_DIRECTORY "${PROJECT_BINARY_DIR}"
    COMMENT "Packaging release ${PROJECT_VERSION}.${BUILD_COUNT}"
    VERBATIM
  )
endif()

# test host (fake engine harness); native x64 only, ci presets never opt in
if(ABPB_BUILD_TESTS AND ABPB_64BIT AND ABPB_NATIVE
  AND EXISTS "${PROJECT_SOURCE_DIR}/tests/CMakeLists.txt")
  enable_testing()
  add_subdirectory(tests)

  abpb_configure_target(abpb_static STRICT DIST)
  if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    set_source_files_properties(src/copycat/copycat_core.cpp src/copycat/copycat_lite.cpp src/copycat/copycat_routes.cpp src/copycat/copycat_learning.cpp src/copycat/copycat_explore.cpp src/copycat/copycat_dialogue.cpp src/copycat/copycat_final.cpp src/copycat/copycat_aim.cpp
      TARGET_DIRECTORY abpb_static PROPERTIES COMPILE_OPTIONS "-w")
    set_source_files_properties(src/human_copycat.cpp src/human_explore.cpp src/human_final.cpp src/human_chat.cpp src/human_aim.cpp
      TARGET_DIRECTORY abpb_static PROPERTIES COMPILE_OPTIONS "-Wno-effc++;-Wno-zero-as-null-pointer-constant")
  endif()

  if(NOT ABPB_WINXP)
    target_link_libraries(abpb_static PRIVATE ystl_printf)
  endif()
endif()

if(GAME_DIR AND SERVER_INSTALL_DIR)
  install(TARGETS ${PROJECT_NAME}
    DESTINATION "${GAME_DIR}/${SERVER_INSTALL_DIR}/"
    PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
    GROUP_READ GROUP_EXECUTE
    WORLD_READ WORLD_EXECUTE)

  if(MSVC)
    install(FILES $<TARGET_PDB_FILE:${PROJECT_NAME}>
      DESTINATION "${GAME_DIR}/${SERVER_INSTALL_DIR}/" OPTIONAL)
  endif()
endif()
