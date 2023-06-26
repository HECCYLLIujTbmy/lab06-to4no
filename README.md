# lab06-to4no
# Homework

*Вы продолжаете проходить стажировку в "Formatter Inc." (см подробности).*

*В прошлый раз ваше задание заключалось в настройке автоматизированной системы CMake.*

*Сейчас вам требуется настроить систему непрерывной интеграции для библиотек и приложений, с которыми вы работали в прошлый раз. Настройте сборочные процедуры на различных платформах:*

## используйте TravisCI для сборки на операционной системе Linux с использованием компиляторов gcc и clang;
## используйте AppVeyor для сборки на операционной системе Windows.

# 1. Клонируем репозиторий из 4 лабораторной работы,
```sh
$ git clone https://github.com/HECCYLLIujTbmy/lab04-pls lab06-to4no
```

# 2. Добавляем файлы CPackConfig.cmake и CMakeLists.txt
```sh
$ nano CMakeLists.txt
cmake_minimum_required(VERSION 3.4)
project(lab06)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include_directories("formatter_lib")
include_directories("formatter_ex_lib")
include_directories("solver_lib")

add_library(formatter_lib STATIC "formatter_lib/formatter.cpp")
add_library(formatter_ex_lib STATIC "formatter_ex_lib/formatter_ex.cpp")
add_library(solver_lib STATIC "solver_lib/solver.cpp")

add_executable(solver "solver_application/equation.cpp")

target_link_libraries(solver solver_lib formatter_ex_lib formatter_lib)

include(CPackConfig.cmake)
```
```sh
$ nano CPackConfig.cmake
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT donotdisturb@yandex.ru)
set(CPACK_PACKAGE_VERSION_MAJOR ${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")

set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print-solver")
set(CPACK_RPM_PACKAGE_VERSION CPACK_PACKAGE_VERSION)

set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_VERSION CPACK_PACKAGE_VERSION)

include(CPack)
```
# 3. Добавляем в .github/workflows файл CI.yml
```sh
$ cd .github/workflows 
$ nano CI.yml
name: CMake

on:
  push:
    branches: [main]
    tags: -"v*0.*"
  pull_request:
    branches: [main]

env:
  BUILD_TYPE: Release

jobs:
  build_Linux:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Configure Solver
        run: cmake -H. -B_build -DCPACK_GENERATOR="TGZ"

      - name: Build Solver
        run: cmake --build _build --target package

      - name: Make Solver Package
        run: cd _build && cpack -G "DEB" &&
             cpack -G "RPM" &&
             mkdir ../artifacts &&
             mv *.tar.gz ../artifacts/ &&
             mv *.deb ../artifacts/ &&
             mv *.rpm ../artifacts/
      - name: Publish
        uses: actions/upload-artifact@v2
        with:
          name: DebRpm
          path: artifacts/
```
