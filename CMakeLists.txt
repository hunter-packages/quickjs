cmake_minimum_required(VERSION 3.9)
project(quickjs)

add_definitions(-DCONFIG_VERSION="2019-07-21")
add_definitions(-D_GNU_SOURCE)

get_filename_component(config_cc "${CMAKE_C_COMPILER}" NAME_WE)
add_definitions(-DCONFIG_CC="${config_cc}")

# That's not right...
add_definitions(-DCONFIG_PREFIX="${CMAKE_INSTALL_PREFIX}")

add_executable(
    qjsc
    qjsc.c
    quickjs.c
    libregexp.c
    libunicode.c
    cutils.c
    quickjs-libc.c
)

include(CheckLibraryExists)
check_library_exists(m "pow" "" __math_system_library)

if(__math_system_library)
  target_link_libraries(qjsc PRIVATE m)
endif()

target_link_libraries(qjsc PRIVATE ${CMAKE_DL_LIBS})

add_custom_command(
    OUTPUT repl.c
    COMMAND qjsc -c -o repl.c -m ${CMAKE_CURRENT_LIST_DIR}/repl.js
)

add_executable(
    qjs
    qjs.c
    repl.c
    quickjs.c
    libregexp.c
    libunicode.c
    cutils.c
    quickjs-libc.c
)

if(__math_system_library)
  target_link_libraries(qjs PRIVATE m)
endif()

target_link_libraries(qjs PRIVATE ${CMAKE_DL_LIBS})

add_executable(
    qjsbnc
    qjsc.c
    quickjs.c
    libregexp.c
    libunicode.c
    cutils.c
    quickjs-libc.c
    libbf.c
)

target_compile_definitions(qjsbnc PRIVATE CONFIG_BIGNUM)

if(__math_system_library)
  target_link_libraries(qjsbnc PRIVATE m)
endif()

target_link_libraries(qjsbnc PRIVATE ${CMAKE_DL_LIBS})

add_custom_command(
    OUTPUT repl-bn.c
    COMMAND qjsbnc -c -o repl-bn.c -m ${CMAKE_CURRENT_LIST_DIR}/repl.js
)

add_custom_command(
    OUTPUT qjscalc.c
    COMMAND qjsbnc -c -o qjscalc.c ${CMAKE_CURRENT_LIST_DIR}/qjscalc.js
)

add_executable(
    qjsbn
    qjs.c
    repl-bn.c
    qjscalc.c
    quickjs.c
    libregexp.c
    libunicode.c
    cutils.c
    quickjs-libc.c
    libbf.c
)

if(__math_system_library)
  target_link_libraries(qjsbn PRIVATE m)
endif()

target_link_libraries(qjsbn PRIVATE ${CMAKE_DL_LIBS})

target_compile_definitions(qjsbn PRIVATE CONFIG_BIGNUM)

include(GNUInstallDirs)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# * TARGETS_EXPORT_NAME
# * PROJECT_NAME
configure_package_config_file(
    "cmake/template/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

add_library(
    quickjs
    quickjs.c
    libregexp.c
    libunicode.c
    cutils.c
    quickjs-libc.c
)

add_library(
    quickjs.bn
    quickjs.c
    libregexp.c
    libunicode.c
    cutils.c
    quickjs-libc.c
    libbf.c
)

target_compile_definitions(quickjs.bn PRIVATE CONFIG_BIGNUM)

install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${config_install_dir}"
)

install(
    TARGETS
    qjsc
    qjs
    qjsbnc
    qjsbn
    quickjs
    quickjs.bn
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(FILES quickjs.h quickjs-libc.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/quickjs)

add_executable(run-test262 run-test262.c)
target_link_libraries(run-test262 PRIVATE quickjs)

if(__math_system_library)
  target_link_libraries(run-test262 PRIVATE m)
endif()

target_link_libraries(run-test262 PRIVATE ${CMAKE_DL_LIBS})

find_package(Threads REQUIRED)
target_link_libraries(run-test262 PRIVATE Threads::Threads)
