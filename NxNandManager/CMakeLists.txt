cmake_minimum_required(VERSION 3.29)
project(NxNandManager)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

set(CMAKE_PREFIX_PATH "C:/Qt/6.7.3/mingw_64")

find_package(Qt6 COMPONENTS
        Core
        Gui
        Widgets
        Network
        REQUIRED)

file(GLOB main *.cpp *.h)
file(GLOB gui gui/*.cpp gui/*.h gui/*.ui)
file(GLOB res res/*.cpp res/*.h)
file(GLOB ziplib
        lib/ZipLib/*.cpp
        lib/ZipLib/*.h
        lib/ZipLib/detail/*.cpp
        lib/ZipLib/detail/*.h
        lib/ZipLib/extlibs/bzip2/*.c
        lib/ZipLib/extlibs/bzip2/*.h
        lib/ZipLib/extlibs/lzma/*.c
        lib/ZipLib/extlibs/lzma/*.h
        lib/ZipLib/extlibs/zlib/*.c
        lib/ZipLib/extlibs/zlib/*.h
        lib/ZipLib/utils/*.h)

file(GLOB fatfs lib/fatfs/*)

add_executable(NxNandManager
        ${main}
        ${gui}
        ${res}
        ${ziplib}
        ${fatfs}
)
target_link_libraries(NxNandManager
        Qt::Core
        Qt::Gui
        Qt::Widgets
        Qt::Network
)

target_link_libraries(NxNandManager
        dokan1
        crypto
)

target_link_directories(NxNandManager PRIVATE virtual_fs/dokan/x64/lib)
target_link_directories(NxNandManager PRIVATE ../OpenSSL_mingw64/lib)

target_include_directories(NxNandManager PRIVATE ../OpenSSL_mingw64/include)
target_include_directories(NxNandManager PRIVATE virtual_fs/dokan/include)
target_include_directories(NxNandManager PRIVATE gui/include)

if (WIN32 AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    set(DEBUG_SUFFIX)
    if (MSVC AND CMAKE_BUILD_TYPE MATCHES "Debug")
        set(DEBUG_SUFFIX "d")
    endif ()
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    foreach (QT_LIB Core Gui Widgets)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt6${QT_LIB}${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)
endif ()
