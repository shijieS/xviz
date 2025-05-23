find_package(protobuf REQUIRED)
find_package(fmt REQUIRED)

vehiclas_print_variable_name(protobuf)

# generate protobuf source files
add_library(
  xviz_pb OBJECT
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/annotation.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/core.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/declarativeui.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/envelope.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/options.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/primitives.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/session.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/style.proto
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto/xviz/v2/uiprimitives.proto)

target_link_libraries(xviz_pb PUBLIC protobuf::libprotobuf)
target_include_directories(
  xviz_pb PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/generated>
                 $<INSTALL_INTERFACE:include>)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/generated)

# include protobuf-build/cmake/protobuf protobuf-generate.cmake
include(${protobuf_BINARY_DIR}/cmake/protobuf/protobuf-generate.cmake)

protobuf_generate(
  TARGET
  xviz_pb
  LANGUAGE
  cpp
  IMPORT_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}/../proto
  PROTOC_OUT_DIR
  ${CMAKE_CURRENT_BINARY_DIR}/generated)

# xviz source files
add_library(xviz ${CMAKE_CURRENT_SOURCE_DIR}/xviz.cc
                 ${CMAKE_CURRENT_SOURCE_DIR}/utils/utils.cc
                 ${CMAKE_CURRENT_SOURCE_DIR}/utils/base64.cc
                 )

target_link_libraries(xviz xviz_pb protobuf::libprotobuf fmt::fmt)

target_compile_definitions(xviz PUBLIC XVIZ_VERSION="${XVIZ_VERSION}")

target_include_directories(
  xviz PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../include>
              $<INSTALL_INTERFACE:include>)

include(GNUInstallDirs)
install(
  TARGETS xviz xviz_pb
  EXPORT xvizTargets
  INCLUDES
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

# headers
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../include/xviz
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/generated/xviz
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(
  EXPORT xvizTargets
  FILE xvizTargets.cmake
  NAMESPACE xviz::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/xvizConfigVersion.cmake"
  VERSION ${xviz_VERSION}
  COMPATIBILITY AnyNewerVersion)

configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/../cmake/xvizConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/xvizConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/xvizConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/xvizConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
