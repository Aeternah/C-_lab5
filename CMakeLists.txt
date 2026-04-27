cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 1.0.${PATCH_VERSION})

project(physics VERSION ${PROJECT_VESRION})

set(CMAKE_CXX_STANDARD 17)

# Сначала ищем без компонентов, чтобы не было ошибки
find_package(SFML QUIET)

if (SFML_FOUND AND TARGET sfml-graphics)
    message(STATUS "The SFML installed in the system will be used")
    set(USE_SYSTEM_SFML TRUE)
else()
    message(STATUS "SFML not found or missing graphics, will download from source")
    set(USE_SYSTEM_SFML FALSE)
endif ()

if (NOT USE_SYSTEM_SFML)
    message(STATUS "Getting SFML from source code")
    include(FetchContent)
    FetchContent_Declare(SFML
            GIT_REPOSITORY https://github.com/SFML/SFML.git
            GIT_TAG 2.6.x)
    
    set(SFML_BUILD_AUDIO OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_NETWORK OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_GRAPHICS ON CACHE BOOL "" FORCE)
    set(SFML_BUILD_WINDOW ON CACHE BOOL "" FORCE)
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
    set(WARNINGS_AS_ERRORS FALSE CACHE BOOL "" FORCE)
    
    FetchContent_MakeAvailable(SFML)
endif ()

if (MSVC)
    add_definitions(-D_USE_MATH_DEFINES)
endif()

add_executable(physics
        main.cpp World.cpp Ball.cpp Physics.cpp Color.cpp Painter.cpp
        black_box/Application.cpp black_box/View.cpp black_box/PainterImpl.cpp Dust.cpp)

target_link_libraries(physics PRIVATE sfml-graphics)

target_compile_options(physics PRIVATE
        $<$<CXX_COMPILER_ID:MSVC>:/W4>
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>
        )

if(WIN32)
    set_target_properties(physics PROPERTIES OUTPUT_NAME "physics-${PROJECT_VERSION}-Windows")
endif()

install(TARGETS physics DESTINATION bin)

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_CONTACT "example@example.com")

if(WIN32)
    set(CPACK_GENERATOR ZIP) 
    set(CPACK_SOURCE_GENERATOR ZIP)
elseif(UNIX)
    set(CPACK_GENERATOR DEB) 
    set(CPACK_SOURCE_GENERATOR TGZ)
endif()

include(CPack)