# ISO C Fortran BMI bindings

## About

This is a small library that exposes a C/Fortran interoperability layer for BMI functions.  The library presents the BMI functions as a set of free functions that operate on a properly initialized and boxed Fortran BMI object.  This requires Fortran BMI models to implement a `register_bmi` function which, when called by a C/C++ executable, provides an opaque handle to the underlying BMI type `BMI_TYPE`:

```fortran
  function register_bmi(this) result(bmi_status) bind(C, name="register_bmi")
   use, intrinsic:: iso_c_binding, only: c_ptr, c_loc, c_int
   use iso_c_bmif_2_0
   implicit none
   type(c_ptr) :: this
   integer(kind=c_int) :: bmi_status
   !Create the model instance to use
   type(BMI_TYPE), pointer :: bmi_model
   !Create a simple pointer wrapper
   type(box), pointer :: bmi_box

   !allocate model
   allocate(BMI_TYPE::bmi_model)
   !allocate the pointer box
   allocate(bmi_box)

   !associate the wrapper pointer the created model instance
   bmi_box%ptr => bmi_model

   if( .not. associated( bmi_box ) .or. .not. associated( bmi_box%ptr ) ) then
    bmi_status = BMI_FAILURE
   else
    !Return the pointer to box
    this = c_loc(bmi_box)
    bmi_status = BMI_SUCCESS
   endif
 end function register_bmi
```

# Usage

Once the handle has been obtained by calling `register_bmi` it can be be passed to any of the BMI functions in liu of the usual `BMI *`, with the rest of the arguments following the same semantics as the BMI interface arguements.  A small example of calling `update` on a Fortran BMI model from a C binary linked to this ISO C BMI library and an approriate BMI model implementing `register_bmi`

```C

extern register_bmi(void *);
extern initialize(void *, char *);
extern update(void *);
extern finalize(void*);

int main(int argc, char** argv)
{
    void** bmi_handle;
    int status = -1;

    char name[2048];
    //Get the opaque handle to pass to the BMI functions
    status = register_bmi(&bmi_handle);

    char init_file[2048] = "namelist.input";
    //Initialize the BMI model conained in the bmi_handle
    status = initialize(&bmi_handle, init_file);
    //Update the model contained in the bmi_handle
    update(&bmi_handle);

    //Finalize and shut down the model
    status = finalize(&bmi_handle);
}

```

## Building The Library

### Generate the build system
Before library files can be built, CMake must generate a build system.  E.g., from the top-level directory of the repository:

```sh
cmake -B cmake_build -S .
```

This will create an out-of-source build directory `cmake_build/` within the top-level repo directory.


### Build the Shared Library Target
After there is build system directory, the shared library can be built using:
```sh
cmake --build cmake_build --target iso_c_bmi
```
This will build a `cmake_build/libiso_c_bmi.so.<version>.<ext>` file, where the version is configured within the CMake config, and the extension depends on the local machine's operating system.    

## Building and Running the Tests

When this repository is built as a top-level CMake project, a companion test binary (`test_iso_c`) and a small Fortran BMI test model (`testbmifortranmodel`) are configured alongside the library.  No separate build-system generation is required — the same `cmake_build` directory from above is reused.

Build the test target:
```sh
cmake --build cmake_build --target test_iso_c
```

Run the tests via CTest:
```sh
ctest --test-dir cmake_build --output-on-failure
```

The test executable needs to run relative to its input config (`test_bmi_fortran_config_0.txt` in `test/`); CTest handles this automatically by setting the working directory.

When this repository is consumed as a subdirectory from another CMake project (e.g. ngen), the test targets are not added — only the `iso_c_bmi` library target is exposed.