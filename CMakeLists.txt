cmake_minimum_required(VERSION 2.8)

project (luv)

include(deps/uv.cmake)

if (LUV_LUA_INCDIR)
  include_directories(${LUV_LUA_INCDIR})
  link_directories(${LUV_LUA_LIBDIR})
else(LUV_LUA_INCDIR)
  include(deps/luajit.cmake)
  include_directories(deps/luajit/src)
endif(LUV_LUA_INCDIR)

if( NOT EXISTS "deps/libuv/src" )
  execute_process(COMMAND
    git submodule update --init
    WORKING_DIRECTORY PROJECT_SOURCE_DIR
  )
  if( NOT EXISTS "deps/libuv/src" )
    message( SEND_ERROR "The git submodules are not available. Please run git submodule update --init")
  endif()
endif()


add_library (luv MODULE src/luv.c)

set_target_properties(luv PROPERTIES OUTPUT_NAME "luv")
set_target_properties(luv PROPERTIES PREFIX "")

if(APPLE)
  set(CMAKE_SHARED_MODULE_CREATE_C_FLAGS
    "${CMAKE_SHARED_MODULE_CREATE_C_FLAGS} -flat_namespace -undefined suppress"
  )
  # execute_process(COMMAND which luajit OUTPUT_VARIABLE LUAJIT)
  # set(CMAKE_SHARED_MODULE_CREATE_C_FLAGS
  #   "${CMAKE_SHARED_MODULE_CREATE_C_FLAGS} -bundle_loader ${LUAJIT}"
  # )
endif()

if(WIN32)
  add_definitions(-DLUA_BUILD_AS_DLL -DLUA_LIB)
  target_link_libraries(luv uv lua51)
elseif("${CMAKE_SYSTEM_NAME}" MATCHES "Linux")
  target_link_libraries(luv uv rt)
else()
  target_link_libraries(luv uv)
endif()

if (LUV_INSTALL_PREFIX)
  install(TARGETS luv DESTINATION "${LUV_INSTALL_PREFIX}/lib")
endif(LUV_INSTALL_PREFIX)
