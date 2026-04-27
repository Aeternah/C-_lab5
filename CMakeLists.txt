cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION_FULL 1.0.${PATCH_VERSION})

project(physics VERSION ${PROJECT_VERSION_FULL})

set(CMAKE_CXX_STANDARD 17)

# Очищаем кэш если поменялся генератор
if(CMAKE_GENERATOR)
    message(STATUS "Using generator: ${CMAKE_GENERATOR}")
endif()

# Windows - собираем SFML из исходников (2.6.1)
# Linux - используем системную SFML
if(WIN32)
    message(STATUS "Windows: Building SFML from source")
    
    include(FetchContent)
    
    # Отключаем предупреждения и фиксим кэш
    set(FETCHCONTENT_QUIET OFF)
    
    FetchContent_Declare(SFML
        GIT_REPOSITORY https://github.com/SFML/SFML.git
        GIT_TAG 2.6.1
        GIT_SHALLOW TRUE)
    
    set(SFML_BUILD_AUDIO OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_NETWORK OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_GRAPHICS ON CACHE BOOL "" FORCE)
    set(SFML_BUILD_WINDOW ON CACHE BOOL "" FORCE)
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
    
    # Принудительно используем тот же генератор
    set(FETCHCONTENT_FULLY_DISCONNECTED OFF)
    
    FetchContent_MakeAvailable(SFML)
else()
    find_package(SFML COMPONENTS graphics window system REQUIRED)
endif()

add_executable(physics
    main.cpp World.cpp Ball.cpp Physics.cpp Color.cpp Painter.cpp
    black_box/Application.cpp black_box/View.cpp black_box/PainterImpl.cpp Dust.cpp)

if(WIN32)
    target_link_libraries(physics PRIVATE sfml-graphics)
else()
    target_link_libraries(physics PRIVATE sfml-graphics sfml-window sfml-system)
endif()

target_compile_options(physics PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>)

if(WIN32)
    set_target_properties(physics PROPERTIES OUTPUT_NAME "physics-${PROJECT_VERSION}-Windows")
else()
    set_target_properties(physics PROPERTIES OUTPUT_NAME "physics-${PROJECT_VERSION}-Linux")
endif()

install(TARGETS physics DESTINATION bin)

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_CONTACT "example@example.com")
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})

if(WIN32)
    set(CPACK_GENERATOR ZIP)
    set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION}-Windows")
else()
    set(CPACK_GENERATOR "TGZ;DEB")
    set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION}-Linux")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_CONTACT}")
endif()

include(CPack)