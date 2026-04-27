cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION_FULL 1.0.${PATCH_VERSION})

project(physics VERSION ${PROJECT_VERSION_FULL})

set(CMAKE_CXX_STANDARD 17)

# Убираем поиск компонентов, просто ищем SFML целиком
find_package(SFML QUIET)

if (SFML_FOUND)
    message(STATUS "Using system SFML ${SFML_VERSION}")
    set(USE_SYSTEM_SFML TRUE)
else()
    message(STATUS "System SFML not found, will download SFML 2.6.1 from source")
    set(USE_SYSTEM_SFML FALSE)
    
    include(FetchContent)
    FetchContent_Declare(SFML
        GIT_REPOSITORY https://github.com/SFML/SFML.git
        GIT_TAG 2.6.1
        GIT_SHALLOW TRUE)
    
    set(SFML_BUILD_AUDIO OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_NETWORK OFF CACHE BOOL "" FORCE)
    set(SFML_BUILD_GRAPHICS ON CACHE BOOL "" FORCE)
    set(SFML_BUILD_WINDOW ON CACHE BOOL "" FORCE)
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
    
    FetchContent_MakeAvailable(SFML)
endif()

if (MSVC)
    add_definitions(-D_USE_MATH_DEFINES)
endif()

add_executable(physics
    main.cpp World.cpp Ball.cpp Physics.cpp Color.cpp Painter.cpp
    black_box/Application.cpp black_box/View.cpp black_box/PainterImpl.cpp Dust.cpp)

# Линкуем правильные таргеты в зависимости от ситуации
if(USE_SYSTEM_SFML)
    # Для системной SFML линкуем напрямую
    target_link_libraries(physics PRIVATE sfml-graphics sfml-window sfml-system)
else()
    # Для собранной из исходников
    target_link_libraries(physics PRIVATE sfml-graphics)
endif()

target_compile_options(physics PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Wpedantic>)

if(WIN32)
    set_target_properties(physics PROPERTIES OUTPUT_NAME "physics-${PROJECT_VERSION}-Windows")
elseif(UNIX)
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
elseif(UNIX)
    set(CPACK_GENERATOR DEB)
    set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION}-Linux")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_CONTACT}")
endif()

include(CPack)