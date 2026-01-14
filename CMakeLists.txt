cmake_minimum_required(VERSION 3.26)
set(LUA_VERSION 5.5.0)
project(Lua
  VERSION ${LUA_VERSION}
  DESCRIPTION "Lua built using CMake"
  HOMEPAGE_URL https://github.com/DevelopersCommunity/cmake-lua
  LANGUAGES C
  )

if(NOT WIN32)
  error("Only Windows is supported.")
endif()

# https://www.lua.org/ftp/
set(LUA_TARBALL lua-${CMAKE_PROJECT_VERSION}.tar.gz)
set(LUA_TARBALL_SHA256 57ccc32bbbd005cab75bcc52444052535af691789dba2b9016d5c50640d68b3d)

if(NOT EXISTS ${CMAKE_CURRENT_LIST_DIR}/${LUA_TARBALL})
  file(DOWNLOAD https://www.lua.org/ftp/${LUA_TARBALL}
    ${CMAKE_CURRENT_LIST_DIR}/${LUA_TARBALL}
    EXPECTED_HASH SHA256=${LUA_TARBALL_SHA256}
    )
  file(ARCHIVE_EXTRACT INPUT ${CMAKE_CURRENT_LIST_DIR}/${LUA_TARBALL}
    DESTINATION ${CMAKE_CURRENT_LIST_DIR}
    )
endif()

set(LUA_SRC lua-${CMAKE_PROJECT_VERSION}/src)
set(LUA_DOC lua-${CMAKE_PROJECT_VERSION}/doc)

set(LIB
  ${LUA_SRC}/lapi.c
  ${LUA_SRC}/lcode.c
  ${LUA_SRC}/lctype.c
  ${LUA_SRC}/ldebug.c
  ${LUA_SRC}/ldo.c
  ${LUA_SRC}/ldump.c
  ${LUA_SRC}/lfunc.c
  ${LUA_SRC}/lgc.c
  ${LUA_SRC}/llex.c
  ${LUA_SRC}/lmem.c
  ${LUA_SRC}/lobject.c
  ${LUA_SRC}/lopcodes.c
  ${LUA_SRC}/lparser.c
  ${LUA_SRC}/lstate.c
  ${LUA_SRC}/lstring.c
  ${LUA_SRC}/ltable.c
  ${LUA_SRC}/ltm.c
  ${LUA_SRC}/lundump.c
  ${LUA_SRC}/lvm.c
  ${LUA_SRC}/lzio.c
  ${LUA_SRC}/lauxlib.c
  ${LUA_SRC}/lbaselib.c
  ${LUA_SRC}/lcorolib.c
  ${LUA_SRC}/ldblib.c
  ${LUA_SRC}/liolib.c
  ${LUA_SRC}/lmathlib.c
  ${LUA_SRC}/loadlib.c
  ${LUA_SRC}/loslib.c
  ${LUA_SRC}/lstrlib.c
  ${LUA_SRC}/ltablib.c
  ${LUA_SRC}/lutf8lib.c
  ${LUA_SRC}/linit.c
)

set(LUA
  ${LUA_SRC}/lua.c
  )

set (LUAC
  ${LUA_SRC}/luac.c
  )

add_executable(lua ${LUA} ${LIB})
add_executable(luac ${LUAC} ${LIB})

add_library(lualib SHARED ${LIB})
target_sources(lualib
  PUBLIC
  FILE_SET include TYPE HEADERS BASE_DIRS ${LUA_SRC}
  FILES ${LUA_SRC}/lauxlib.h  ${LUA_SRC}/lua.h  ${LUA_SRC}/lua.hpp  ${LUA_SRC}/luaconf.h  ${LUA_SRC}/lualib.h
  )

target_compile_definitions(lualib PUBLIC LUA_BUILD_AS_DLL)
set_property(TARGET lualib PROPERTY OUTPUT_NAME lua${CMAKE_PROJECT_VERSION_MAJOR}${CMAKE_PROJECT_VERSION_MINOR})

install(TARGETS lua luac)
install(TARGETS lualib FILE_SET include)
install(FILES ${CMAKE_CURRENT_LIST_DIR}/LICENSE.txt TYPE DOC)

set(CPACK_GENERATOR ZIP)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/lua.pdb
  ${CMAKE_CURRENT_BINARY_DIR}/luac.pdb
  ${CMAKE_CURRENT_BINARY_DIR}/lua55.pdb
  DESTINATION bin)
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_LIST_DIR}/LICENSE.txt)
set(CPACK_PACKAGE_INSTALL_DIRECTORY ${CMAKE_PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR DEVCOM)
include(CPack)