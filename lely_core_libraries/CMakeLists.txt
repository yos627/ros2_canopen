cmake_minimum_required(VERSION 3.8)
project(lely_core_libraries)
find_package(ament_cmake REQUIRED)
find_package(ament_cmake_python REQUIRED)

include(ExternalProject)
ExternalProject_Add(upstr_lely_core_libraries    # Name for custom target
  #--Download step--------------
  GIT_REPOSITORY https://gitlab.com/lely_industries/lely-core.git
  GIT_TAG 7824cbb2ac08d091c4fa2fb397669b938de9e3f5
  TIMEOUT 60
  #--Update/Patch step----------
  UPDATE_COMMAND git apply ${CMAKE_CURRENT_SOURCE_DIR}/patches/0001-Don-t-build-bdist.patch
  COMMAND git apply ${CMAKE_CURRENT_SOURCE_DIR}/patches/0002-No-bdist.patch
  #--Configure step-------------
  CONFIGURE_COMMAND autoreconf -i <SOURCE_DIR> COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-cython --disable-doc --disable-tests --disable-static --disable-diag
  #--Build step-----------------
  BUILD_IN_SOURCE 1         # Use source dir for build dir
  #--Install step---------------
  INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}"  # Installation prefix
  BUILD_COMMAND cd <SOURCE_DIR>  COMMAND $(MAKE) --silent
)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib DESTINATION ${CMAKE_INSTALL_PREFIX})
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include DESTINATION ${CMAKE_INSTALL_PREFIX})
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin/ DESTINATION bin
        PATTERN "bin/*"
        PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
        GROUP_EXECUTE GROUP_READ)

set(lely_core_cmake_DIR "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include("cmake/lely_core_libraries-extras.cmake" NO_POLICY_SCOPE)


install(
  DIRECTORY cmake
  DESTINATION share/${PROJECT_NAME}
)

ament_python_install_package(cogen SCRIPTS_DESTINATION lib/cogen)

# install entry-point script(s) in bin as well
install(
    DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/ament_cmake_python/cogen/scripts/
    DESTINATION bin/
    USE_SOURCE_PERMISSIONS)

ament_export_include_directories(include)
ament_export_libraries(lely-can lely-co lely-coapp lely-ev lely-io2 lely-io lely-libc lely-tap lely-util)
ament_package(
  CONFIG_EXTRAS
  "cmake/lely_core_libraries-extras.cmake"
)
