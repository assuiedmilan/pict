find_package(Perl)

if(NOT PERL_FOUND)
  message(WARNING "Perl not found - the 'run_pict' test will not be registered")
  return()
endif()

add_test(
  NAME run_pict
  COMMAND ${PERL_EXECUTABLE} test.pl $<TARGET_FILE:pict_cli> $<IF:$<CONFIG:Debug>,dbg,rel>.log
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)
