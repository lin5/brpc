if(BUILD_UNIT_TESTS)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUNIT_TEST -DBVAR_NOT_LINK_DEFAULT_VARIABLES")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DUNIT_TEST")
elseif(NOT DEBUG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNDEBUG")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG")
endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${CMAKE_SOURCE_DIR}/src)

add_library(BUTIL_LIB OBJECT ${BUTIL_SOURCES})
add_library(OBJ_LIB OBJECT ${SOURCES})

# shared library needs POSITION_INDEPENDENT_CODE
set_property(TARGET ${OBJ_LIB} PROPERTY POSITION_INDEPENDENT_CODE 1)
set_property(TARGET ${BUTIL_LIB} PROPERTY POSITION_INDEPENDENT_CODE 1)

add_library(brpc-shared SHARED $<TARGET_OBJECTS:BUTIL_LIB> $<TARGET_OBJECTS:OBJ_LIB>)
add_library(brpc-static STATIC $<TARGET_OBJECTS:BUTIL_LIB> $<TARGET_OBJECTS:OBJ_LIB>)

target_link_libraries(brpc-shared ${DYNAMIC_LIB})

if(WITH_GLOG)
    target_link_libraries(brpc-shared ${GLOG_LIB})
endif()

SET_TARGET_PROPERTIES(brpc-static PROPERTIES OUTPUT_NAME brpc CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(brpc-shared PROPERTIES OUTPUT_NAME brpc CLEAN_DIRECT_OUTPUT 1)

# for protoc-gen-mcpack
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/output/bin)
    
set(protoc_gen_mcpack_SOURCES
    ${CMAKE_SOURCE_DIR}/src/mcpack2pb/generator.cpp
 )
add_executable(protoc-gen-mcpack ${protoc_gen_mcpack_SOURCES})
target_link_libraries(protoc-gen-mcpack brpc-shared)
    
get_property(LIB64 GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS)
if ("${LIB64}" STREQUAL "TRUE")
    set(LIBSUFFIX 64)
else()
    set(LIBSUFFIX "")
endif()

#install directory
# cmake -DCMAKE_INSTALL_PREFIX=/usr
install(TARGETS brpc-shared
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib${LIBSUFFIX}
        ARCHIVE DESTINATION lib${LIBSUFFIX}
        )

install(TARGETS brpc-static
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib${LIBSUFFIX}
        ARCHIVE DESTINATION lib${LIBSUFFIX}
        )
