enable_language(C)

add_subdirectory(test_bmi_fortran)

add_executable(test_iso_c test_iso_c.c)
target_link_libraries(test_iso_c testbmifortranmodel iso_c_bmi)
target_compile_options(test_iso_c PRIVATE -Wno-implicit-int -Wno-implicit-function-declaration)

add_test(
    NAME test_iso_c
    COMMAND test_iso_c
    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
