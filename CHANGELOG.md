All notable changes to this project will be documented in this file.
We follow the [Semantic Versioning 2.0.0](http://semver.org/) format.

## 1.2.0 - 2026-10-06

### Fixed

- Functions taking string arguments now avoid hidden thread-safety pitfalls
- Functions taking string arguments avoid repeatedly calculating string length
- Test checks for failure to open init_config file

## 1.1.0 - 2026-04-10

### Added

- CMake package config export (`iso_c_fortran_bmiConfig.cmake` + version file), so downstream projects can consume the installed library via `find_package(iso_c_fortran_bmi)` and link against `iso_c_fortran_bmi::iso_c_bmi`.
- `BUILD_INTERFACE` / `INSTALL_INTERFACE` generator expressions on `iso_c_bmi`'s interface include directories, so the exported target carries the correct include path in both build and install trees.
- Top-level build wiring for the `test/` subdirectory (gated on `PROJECT_IS_TOP_LEVEL`), CTest registration for the test binary, and `enable_language(C)` scoped to `test/` so consumers don't need a C toolchain for the library itself.

### Deprecated

- Nothing.

### Removed

- Stale `NGEN_IS_MAIN_PROJECT` fallback in `test/test_bmi_fortran/CMakeLists.txt` that reached out to a sibling `iso_c_fortran_bmi` directory no longer present at that path.
- `install()` calls for the `testbmifortranmodel` test fixture — it's a test fixture, not a shipped artifact, and its broken `.pc` install path was causing `cmake --install` to fail.
- `NGEN_ACTIVE` conditional compilation from the `testbmifortranmodel` test fixture, along with its now-unused standalone `bmif_2_0` module (`test/test_bmi_fortran/src/bmi.f90`); the fixture always builds against the ISO C BMI.
- Stale `extern/iso_c_fortran_bmi/` paths and the separate `cd test && cmake ...` flow from the README.

### Fixed

- Fortran `.mod` files are now actually installed. The previous `install(DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY} ...)` referenced an unset variable and silently shipped nothing, leaving downstream Fortran consumers unable to `use iso_c_bmif_2_0`.
- `iso_c_bmi.pc.in` rewritten: relocatable `${pcfiledir}`-based prefix, real `Name` / `Description`, and correct `-liso_c_bmi` instead of the stale `-lmylib` boilerplate.
- `iso_c_bmi.pc` install moved from `share/pkgconfig` to `lib/pkgconfig`, the default `pkg-config` search path.
- `test/CMakeLists.txt` no longer references the stale `../../test_bmi_fortran` path; uses the local `test/test_bmi_fortran/` subdirectory.
- Test executable renamed from `test` to `test_iso_c` to avoid colliding with CTest's reserved `test` target.
- README build and test instructions now match the single-build-tree flow that actually works from the repo root.


## 1.0.0 - 2026-03-06

### Added

- Initial release of `iso_c_fortran_bmi` as a standalone repository, extracted from the [NOAA-OWP/ngen](https://github.com/NOAA-OWP/ngen) repository so the ISO C Fortran BMI bindings can be developed, versioned, and consumed independently of ngen.

### Deprecated

- Nothing.

### Removed

- Nothing.

### Fixed

- Nothing.
