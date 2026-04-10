cmake_minimum_required(VERSION 3.12...3.26)

# iso_c_fortran_bmi
#
# Output Targets:
# - `iso_c_bmi` (installed / exported as `iso_c_fortran_bmi::iso_c_bmi`)

project(iso_c_fortran_bmi VERSION 1.0.0 LANGUAGES Fortran)

include(GNUInstallDirs)

set(_source_dir "${CMAKE_CURRENT_LIST_DIR}")
set(_iso_c_bmi_module_dir "${CMAKE_CURRENT_BINARY_DIR}/mod")

add_library(iso_c_bmi SHARED)
file(GLOB _ISO_C_BMI_OBJECTS CONFIGURE_DEPENDS "${_source_dir}/src/*.f90")
set_target_properties(iso_c_bmi
    PROPERTIES
        Fortran_MODULE_DIRECTORY "${_iso_c_bmi_module_dir}"
        VERSION ${PROJECT_VERSION})

target_sources(iso_c_bmi PRIVATE ${_ISO_C_BMI_OBJECTS})
target_compile_options(iso_c_bmi PRIVATE -cpp -ffree-line-length-none)
target_include_directories(iso_c_bmi INTERFACE
    $<BUILD_INTERFACE:${_iso_c_bmi_module_dir}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/iso_c_fortran_bmi>)

unset(_ISO_C_BMI_OBJECTS)

install(TARGETS iso_c_bmi
        EXPORT iso_c_fortran_bmiTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

# Install the .mod files from iso_c_bmi's Fortran module output directory.
# Trailing slash copies the directory *contents*, not the directory itself.
install(DIRECTORY "${_iso_c_bmi_module_dir}/"
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/iso_c_fortran_bmi
        FILES_MATCHING PATTERN "*.mod")

install(EXPORT iso_c_fortran_bmiTargets
        FILE iso_c_fortran_bmiTargets.cmake
        NAMESPACE iso_c_fortran_bmi::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iso_c_fortran_bmi)

include(CMakePackageConfigHelpers)
configure_package_config_file(
    "${_source_dir}/cmake/iso_c_fortran_bmiConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/iso_c_fortran_bmiConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iso_c_fortran_bmi)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/iso_c_fortran_bmiConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion)
install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/iso_c_fortran_bmiConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/iso_c_fortran_bmiConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/iso_c_fortran_bmi)

configure_file(iso_c_bmi.pc.in iso_c_bmi.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/iso_c_bmi.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

if(PROJECT_IS_TOP_LEVEL)
    include(CTest)
    # include(CTest) above defines BUILD_TESTING as a cache option (default ON).
    if(BUILD_TESTING)
        add_subdirectory(test)
    endif()
endif()
