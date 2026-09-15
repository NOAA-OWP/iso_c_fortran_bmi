enable_language(C)

add_subdirectory(test_bmi_fortran)

add_executable(test_iso_c test_iso_c.c)
target_link_libraries(test_iso_c testbmifortranmodel iso_c_bmi)
target_compile_options(test_iso_c PRIVATE -Wno-implicit-int -Wno-implicit-function-declaration)

add_test(
    NAME test_iso_c
    COMMAND test_iso_c
    WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})

find_package(Threads REQUIRED)

# Concurrency regression test for the shim's string conversion.
# Built, but deliberately not registered with CTest: it is a stress test meant
# to be run by hand, not on every `ctest` invocation.
add_executable( test_threads test_iso_c_threads.c)
target_link_libraries( test_threads testbmifortranmodel iso_c_bmi Threads::Threads )
