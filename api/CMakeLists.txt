include(GNUInstallDirs)

set(target_name
  pict_api
)

set(pict_api_output
  pict
)

set(pict_api_inc
  ${CMAKE_CURRENT_SOURCE_DIR}/deriver.h
  ${CMAKE_CURRENT_SOURCE_DIR}/generator.h
  ${CMAKE_CURRENT_SOURCE_DIR}/pictapi.h
  ${CMAKE_CURRENT_SOURCE_DIR}/resource.h
  ${CMAKE_CURRENT_SOURCE_DIR}/trie.h
)

set(pict_api_src
  ${CMAKE_CURRENT_SOURCE_DIR}/combination.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/deriver.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/exclusion.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/model.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/parameter.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/pictapi.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/task.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/worklist.cpp
)

add_library(${target_name}
  STATIC
    ${pict_api_inc}
    ${pict_api_src}
)

source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}
  FILES
    ${pict_api_inc}
    ${pict_api_src}
)

target_include_directories(${target_name}
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

target_link_libraries(${target_name}
  PRIVATE
    PictBuildOptions
)

set_target_properties(${target_name}
  PROPERTIES
    FOLDER src
    OUTPUT_NAME ${pict_api_output}
    PUBLIC_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/pictapi.h
)

install(
  TARGETS ${target_name}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Shared library exporting the PICT API (pict.dll / libpict.dylib / libpict.so)
# for consumption via C interop, e.g. C# P/Invoke [DllImport("pict")].
set(shared_target_name
  pict_shared
)

add_library(${shared_target_name}
  SHARED
    ${pict_api_inc}
    ${pict_api_src}
    # Exports on Windows come from the module-definition file
    $<$<BOOL:${WIN32}>:${CMAKE_CURRENT_SOURCE_DIR}/pict.def>
    $<$<BOOL:${WIN32}>:${CMAKE_CURRENT_SOURCE_DIR}/pict.rc>
)

target_include_directories(${shared_target_name}
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    # pict.rc includes ver.h for version info. Windows-only: on Unix, cli/ on
    # the include path would shadow the system <strings.h> with cli/strings.h.
    $<$<BOOL:${WIN32}>:${CMAKE_SOURCE_DIR}/cli>
)

target_link_libraries(${shared_target_name}
  PRIVATE
    PictBuildOptions
)

set_target_properties(${shared_target_name}
  PROPERTIES
    FOLDER src
    OUTPUT_NAME ${pict_api_output}
    # Keep the MSVC import library from clashing with the static pict.lib
    ARCHIVE_OUTPUT_NAME pict_shared
)

install(
  TARGETS ${shared_target_name}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
