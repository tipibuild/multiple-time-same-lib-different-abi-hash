cmake_minimum_required(VERSION 3.17.0)

project(commonLib VERSION "0.0.1")

if (NOT DEFINED INCLUDE_FEATURES)
  set(INCLUDE_FEATURES libByDefault otherThing)
endif()

set (generated_lib_on_demand "")
set (generated_stringlib_on_demand "")
foreach(SPECIAL_LIBNAME ${INCLUDE_FEATURES})
  list(APPEND generated_lib_on_demand commonLib_${SPECIAL_LIBNAME})
  set(generated_stringlib_on_demand "${generated_stringlib_on_demand} ${SPECIAL_LIBNAME}")
endforeach()


add_definitions(-DINCLUDE_FEATURES="${generated_stringlib_on_demand}")
add_executable(testexec testexec.cpp)

add_library(commonLib commonlib.cpp)

set(include_install_dir "include")
target_include_directories(commonLib BEFORE PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/> 
  $<INSTALL_INTERFACE:${include_install_dir}/>)

add_library(commonLib::commonLib ALIAS commonLib)
foreach(SPECIAL_LIBNAME ${INCLUDE_FEATURES})
  message("Special name '${SPECIAL_LIBNAME}'")
  add_library(${SPECIAL_LIBNAME} commonlib_${SPECIAL_LIBNAME}.cpp)
  target_include_directories(${SPECIAL_LIBNAME} BEFORE PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/> 
    $<INSTALL_INTERFACE:${include_install_dir}/>)
  add_library(commonLib::${SPECIAL_LIBNAME} ALIAS ${SPECIAL_LIBNAME})
endforeach()




# Installing

# Layout. This works for all platforms:
#   * <prefix>/lib/cmake/<PROJECT-NAME>
#   * <prefix>/lib/
#   * <prefix>/include/
set(config_install_dir "lib/cmake/${PROJECT_NAME}")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/modules/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)


# Targets:
install(
    TARGETS commonLib ${INCLUDE_FEATURES}
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)


# Headers:
install(
    DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/
    DESTINATION "${include_install_dir}"
    FILES_MATCHING PATTERN "*.[ih]*"
    PATTERN ${CMAKE_CURRENT_LIST_DIR}/build EXCLUDE
)



# Config
#   * <prefix>/lib/cmake/{{project}}/{{project}}Config.cmake
#   * <prefix>/lib/cmake/{{project}}/{{project}}ConfigVersion.cmake
#   * <prefix>/lib/cmake/{{project}}/{{project}}Targets.cmake
install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)
install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
