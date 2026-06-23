cmake_minimum_required(VERSION 3.21)
project(SPIRVVM)

# ── Sources ────────────────────────────────────────────────────────────────────
set(SOURCES
    src/program.c
    src/types.c
    src/image.c
    src/result.c
    src/state.c
    src/value.c
    src/context.c
    src/opcode_execute.c
    src/opcode_setup.c
    src/ext/GLSL450.c
)

# ── Library target ─────────────────────────────────────────────────────────────
add_library(SPIRVVM STATIC ${SOURCES})

# Link math library on UNIX (not needed on Windows / MSVC).
if(UNIX)
    target_link_libraries(SPIRVVM PUBLIC m)
endif()

# PUBLIC: the inc/ directory is part of SPIRVVM's interface.
# Consumers who link wyvrn::SPIRVVM automatically get <spvm/...> on their
# include path — no manual target_include_directories needed downstream.
target_include_directories(SPIRVVM
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
        $<INSTALL_INTERFACE:include>
)

# ── Install / export (needed by wyvrnpm publish) ───────────────────────────────
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Install the compiled library (.lib / .a) per configuration.
install(TARGETS SPIRVVM
    EXPORT SPIRVVMTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/$<CONFIG>
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/$<CONFIG>
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/$<CONFIG>
)

# Install the public headers so consumers get <spvm/...> after find_package.
install(DIRECTORY inc/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Generate and install CMake package config files so downstream consumers
# can call find_package(SPIRV-VM CONFIG REQUIRED) and link wyvrn::SPIRVVM.
install(EXPORT SPIRVVMTargets
    FILE        SPIRVVMTargets.cmake
    NAMESPACE   wyvrn::
    DESTINATION ${CMAKE_INSTALL_DATADIR}/SPIRV-VM
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/SPIRV-VMConfigVersion.cmake"
    VERSION       1.0.0
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/SPIRV-VMConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/SPIRV-VMConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_DATADIR}/SPIRV-VM
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/SPIRV-VMConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/SPIRV-VMConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_DATADIR}/SPIRV-VM
)
