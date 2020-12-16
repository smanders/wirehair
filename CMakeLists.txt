cmake_minimum_required(VERSION 3.5)
project(wirehair)

set(CMAKE_CXX_STANDARD 11)
include(flags OPTIONAL)

set(LIB_SOURCE_FILES
        wirehair.cpp
        include/wirehair/wirehair.h
        gf256.cpp
        gf256.h
        WirehairCodec.cpp
        WirehairCodec.h
        WirehairTools.cpp
        WirehairTools.h
        )

set(UNIT_TEST_SOURCE_FILES
        test/SiameseTools.cpp
        test/SiameseTools.h
        test/UnitTest.cpp
        )

set(GEN_SMALL_DSEEDS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GenerateSmallDenseSeeds.cpp
        )

set(GEN_PEEL_SEEDS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GeneratePeelSeeds.cpp
        )

set(GEN_MOST_DSEEDS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GenerateMostDenseSeeds.cpp
        )

set(GEN_DCOUNTS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GenerateDenseCount.cpp
        )

set(GEN_TABLES
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/TableGenerator.cpp
        tables/HeavyRowGenerator.cpp
        tables/HeavyRowGenerator.h
        gf256.cpp
        gf256.h
        )

if(MSVC)
else()
    include(CheckCXXCompilerFlag)
    foreach(flag -Wall -Wextra -march=native -mssse3)
        string(REPLACE "=" "_" _flag ${flag})
        check_cxx_compiler_flag("${flag}" has_wh${_flag})
        if(has_wh${_flag})
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${flag}")
        endif()
    endforeach()
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
    if(XP_BUILD_VERBOSE)
        message(STATUS "CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
        message(STATUS "CMAKE_CXX_FLAGS_DEBUG: ${CMAKE_CXX_FLAGS_DEBUG}")
        message(STATUS "CMAKE_CXX_FLAGS_RELEASE: ${CMAKE_CXX_FLAGS_RELEASE}")
    endif()
    if (CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
        add_definitions(-DLINUX_ARM=1)
    endif()
endif()

if(DEFINED WIREHAIR_VER)
  set(_ver _${WIREHAIR_VER})
endif()
if(DONT_INSTALL_PYTHON)
  set(optionalInstall EXCLUDE_FROM_ALL)
endif()

add_library(wirehair ${LIB_SOURCE_FILES})
set_target_properties(wirehair PROPERTIES VERSION 2)
set_target_properties(wirehair PROPERTIES SOVERSION 2)
set_target_properties(wirehair PROPERTIES OUTPUT_NAME wirehair${_ver})
target_include_directories(wirehair PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )

add_executable(unit_test ${UNIT_TEST_SOURCE_FILES})
target_link_libraries(unit_test wirehair)

add_executable(gen_small_dseeds ${GEN_SMALL_DSEEDS})
target_link_libraries(gen_small_dseeds wirehair)

add_executable(gen_peel_seeds ${GEN_PEEL_SEEDS})
target_link_libraries(gen_peel_seeds wirehair)

add_executable(gen_most_dseeds ${GEN_MOST_DSEEDS})
target_link_libraries(gen_most_dseeds wirehair)

add_executable(gen_dcounts ${GEN_DCOUNTS})
target_link_libraries(gen_dcounts wirehair)

add_executable(gen_tables ${GEN_TABLES})

include(GNUInstallDirs)

set(targetsFile ${PROJECT_NAME}${_ver}-targets)
install(TARGETS wirehair EXPORT ${targetsFile}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
if(DEFINED XP_NAMESPACE)
  set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
endif()
install(EXPORT ${targetsFile} DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake ${nameSpace})
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY python DESTINATION ${CMAKE_INSTALL_PREFIX} ${optionalInstall})
