cmake_minimum_required(VERSION 3.0)
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)

#todo - deduce this from NODEJS_ROOT

set(ZLIB_INCLUDE_DIR ${NODEJS_ROOT}/include)
if(ZLIB_INCLUDE_DIR AND EXISTS "${ZLIB_INCLUDE_DIR}/zlib.h")
    file(STRINGS "${ZLIB_INCLUDE_DIR}/zlib.h" ZLIB_H REGEX "^#define ZLIB_VERSION \"[^\"]*\"$")

    string(REGEX REPLACE "^.*ZLIB_VERSION \"([0-9]+).*$" "\\1" ZLIB_VERSION_MAJOR "${ZLIB_H}")
    string(REGEX REPLACE "^.*ZLIB_VERSION \"[0-9]+\\.([0-9]+).*$" "\\1" ZLIB_VERSION_MINOR  "${ZLIB_H}")
    string(REGEX REPLACE "^.*ZLIB_VERSION \"[0-9]+\\.[0-9]+\\.([0-9]+).*$" "\\1" ZLIB_VERSION_PATCH "${ZLIB_H}")
    set(ZLIB_VERSION_STRING "${ZLIB_VERSION_MAJOR}.${ZLIB_VERSION_MINOR}.${ZLIB_VERSION_PATCH}")

    # only append a TWEAK version if it exists:
    set(ZLIB_VERSION_TWEAK "")
    if( "${ZLIB_H}" MATCHES "ZLIB_VERSION \"[0-9]+\\.[0-9]+\\.[0-9]+\\.([0-9]+)")
        set(ZLIB_VERSION_TWEAK "${CMAKE_MATCH_1}")
        string(APPEND ZLIB_VERSION_STRING ".${ZLIB_VERSION_TWEAK}")
    endif()

    set(ZLIB_MAJOR_VERSION "${ZLIB_VERSION_MAJOR}")
    set(ZLIB_MINOR_VERSION "${ZLIB_VERSION_MINOR}")
    set(ZLIB_PATCH_VERSION "${ZLIB_VERSION_PATCH}")
else()
    message(FATAL_ERROR "invalid NODEJS_ROOT : ${NODEJS_ROOT}")
endif()

project(ZLIB VERSION "${ZLIB_VERSION_STRING}" LANGUAGES C)

set(VERSION "${PROJECT_VERSION}")

option(ASM686 "Enable building i686 assembly implementation")
option(AMD64 "Enable building amd64 assembly implementation")

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
set(INSTALL_MAN_DIR "${CMAKE_INSTALL_PREFIX}/share/man" CACHE PATH "Installation directory for manual pages")
set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")


if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
  ####
  # Installation (https://github.com/forexample/package-example)

  set(config_install_dir "lib/cmake/${PROJECT_NAME}")
  set(include_install_dir "include")

  set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

  set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
  set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
  set(targets_export_name "${PROJECT_NAME}Targets")
  set(namespace "${PROJECT_NAME}::")

  include(CMakePackageConfigHelpers)

  # Note: PROJECT_VERSION is used as a VERSION
  write_basic_package_version_file(
      "${version_config}" COMPATIBILITY SameMajorVersion
  )

  # Note: variable 'targets_export_name' used
  configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

  install(
      FILES "${project_config}" "${version_config}"
      DESTINATION "${config_install_dir}"
  )

endif()
