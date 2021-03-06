set(MAIN_SRC
  "Kovri.cpp")

include_directories(
  "."
  "client"
  "core")

add_subdirectory(api)
add_subdirectory(client)
add_subdirectory(core)
add_subdirectory(tests)

if(WITH_BINARY)
  add_executable(${MAIN_NAME} ${MAIN_SRC})
  target_link_libraries(
    ${MAIN_NAME}
    ${CLIENT_NAME} ${API_NAME} ${CORE_NAME}
    ${Boost_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
  if(NOT MSVC) # FIXME: incremental linker file name (.ilk) collision for dll & exe
    set_target_properties(
      ${MAIN_NAME} PROPERTIES OUTPUT_NAME "${PROJECT_NAME}")
    if(WITH_STATIC)
      set_target_properties(${MAIN_NAME} PROPERTIES LINK_FLAGS "-static" )
    endif()
  endif()
  if(WITH_HARDENING AND CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set_target_properties(${MAIN_NAME} PROPERTIES LINK_FLAGS "-z relro -z now")
  endif()
  # FindBoost pulls pthread for thread which is broken for static linking
  # (at least on Ubuntu 15.04)
  list(GET Boost_LIBRARIES -1 LAST_Boost_LIBRARIES)
  if(${LAST_Boost_LIBRARIES} MATCHES ".*pthread.*")
    list(REMOVE_AT Boost_LIBRARIES -1)
  endif()
  if(MSVC)
    install(FILES
      $<TARGET_PDB_FILE:${MAIN_NAME}> DESTINATION "bin" CONFIGURATIONS DEBUG)
  endif()
endif()

# vim: noai:ts=2:sw=2
