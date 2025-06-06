cmake_minimum_required(VERSION 3.5...3.25)
project(Snake LANGUAGES CXX)

# 1. Настройка версии и метаданных для пакета
set(PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_NAME "snake-game")
set(CPACK_PACKAGE_VENDOR "Your Name")
set(CPACK_PACKAGE_DESCRIPTION "Simple Snake Game using SFML")
set(CPACK_PACKAGE_CONTACT "your.email@example.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libsfml-dev (>= 2.5.1), libc6 (>= 2.27)")
set(CPACK_GENERATOR "DEB")

# 2. Подключение необходимых модулей
include(FetchContent)
include(CPack)

# 3. Настройка SFML
set(SFML_VERSION 2.5.1)

# Сначала пробуем найти SFML в системе
find_package(SFML ${SFML_VERSION} QUIET COMPONENTS graphics window system audio network)

if(NOT SFML_FOUND)
    message(STATUS "SFML не найден, пробуем скачать...")
    
    # Альтернативные источники для загрузки SFML
    set(SFML_URL_OPTIONS
        URL https://www.sfml-dev.org/files/SFML-${SFML_VERSION}-sources.zip
        URL https://github.com/SFML/SFML/archive/refs/tags/${SFML_VERSION}.zip
    )
    
    FetchContent_Declare(
        sfml
        ${SFML_URL_OPTIONS}
        SOURCE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src
        BINARY_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-build
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    )
    
    # Настройка параметров сборки SFML
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build static libraries")
    set(SFML_BUILD_DOC FALSE CACHE BOOL "Don't build documentation")
    set(SFML_BUILD_EXAMPLES FALSE CACHE BOOL "Don't build examples")
    
    FetchContent_MakeAvailable(sfml)
    
    # Установка путей для собранного SFML
    set(SFML_ROOT ${CMAKE_BINARY_DIR}/_deps/sfml-build)
    set(SFML_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src/include)
    
    # Повторная проверка после сборки
    find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS graphics window system audio network)
endif()

# 4. Проверка наличия SFML
if(NOT SFML_FOUND)
    message(FATAL_ERROR "SFML не найден и не может быть загружен автоматически")
endif()

# 5. Копирование ресурсов с проверкой
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/images)
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/images DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
    message(STATUS "Ресурсы images скопированы")
else()
    message(WARNING "Папка images не найдена!")
endif()

# 6. Создание исполняемого файла
add_executable(snake main.cpp)
target_compile_features(snake PRIVATE cxx_std_17)
target_include_directories(snake PRIVATE ${SFML_INCLUDE_DIR})

# 7. Линковка SFML
foreach(COMPONENT graphics window system audio network)
    if(TARGET sfml-${COMPONENT})
        target_link_libraries(snake PRIVATE sfml-${COMPONENT})
    else()
        message(WARNING "Компонент SFML ${COMPONENT} не найден!")
    endif()
endforeach()

# 8. Установка в систему
install(TARGETS snake 
    RUNTIME DESTINATION bin
    BUNDLE DESTINATION bin
)

# 9. Установка ресурсов
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/images)
    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/images
        DESTINATION share/${PROJECT_NAME}
    )
endif()

# 10. Дополнительные настройки для Windows
if(WIN32)
    # Копирование DLL
    add_custom_command(TARGET snake POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "$<TARGET_FILE:sfml-graphics>"
        "$<TARGET_FILE:sfml-window>"
        "$<TARGET_FILE:sfml-system>"
        "$<TARGET_FILE:sfml-audio>"
        "$<TARGET_FILE:sfml-network>"
        "$<TARGET_FILE_DIR:snake>"
        COMMENT "Копирование DLL SFML"
    )
    
    # Установка DLL при инсталляции
    install(FILES
        "$<TARGET_FILE:sfml-graphics>"
        "$<TARGET_FILE:sfml-window>"
        "$<TARGET_FILE:sfml-system>"
        "$<TARGET_FILE:sfml-audio>"
        "$<TARGET_FILE:sfml-network>"
        DESTINATION bin
    )
endif()
