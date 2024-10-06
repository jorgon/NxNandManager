cmake_minimum_required(VERSION 3.16)
project(NxNandManager LANGUAGES C CXX)


include(qt.cmake)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC ON)


find_package(QT NAMES Qt6 REQUIRED COMPONENTS Core)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Gui Network Widgets)

qt_standard_project_setup()

file(GLOB ziplibcpp ${PROJECT_SOURCE_DIR}/lib/ZipLib/*.cpp)
file(GLOB ziplibh ${PROJECT_SOURCE_DIR}/lib/ZipLib/*.h)
file(GLOB ziplibdetailcpp ${PROJECT_SOURCE_DIR}/lib/ZipLib/detail/*.cpp)
file(GLOB ziplibdetailh ${PROJECT_SOURCE_DIR}/lib/ZipLib/detail/*.h)
file(GLOB ziplibbz2c ${PROJECT_SOURCE_DIR}/lib/ZipLib/extlibs/bzip2/*.c)
file(GLOB ziplibbz2lh ${PROJECT_SOURCE_DIR}/lib/ZipLib/extlibs/bzip2/*.h)
file(GLOB zipliblzmac ${PROJECT_SOURCE_DIR}/lib/ZipLib/extlibs/lzma/*.c)
file(GLOB zipliblzmah ${PROJECT_SOURCE_DIR}/lib/ZipLib/extlibs/lzma/*.h)
file(GLOB ziplibzlibc ${PROJECT_SOURCE_DIR}/lib/ZipLib/extlibs/zlib/*.c)
file(GLOB ziplibzlibh ${PROJECT_SOURCE_DIR}/lib/ZipLib/extlibs/zlib/*.h)
file(GLOB ziplibutilsh ${PROJECT_SOURCE_DIR}/lib/ZipLib/utils/*.h)

file(GLOB nx ${PROJECT_SOURCE_DIR}/*)
file(GLOB gui ${PROJECT_SOURCE_DIR}/gui/*)

file(GLOB fatfs ${PROJECT_SOURCE_DIR}/lib/fatfs/*)

set(CMAKE_DEPFILE_FLAGS_CXX /showIncludes)
qt_add_executable(NxNandManager

    ${ziplibcpp}
    ${ziplibdetailcpp}
    ${ziplibbz2c}
    ${zipliblzmah}
    ${fatfs}
    ${nx}

    res/fat32.cpp res/fat32.h
    res/hactool/ivfc.h
    res/hactool/settings.h
    res/hactool/types.h
    res/hactool/utils.c res/hactool/utils.h
    res/hex_string.cpp res/hex_string.h
    res/mbr.cpp res/mbr.h
    res/progress_info.cpp res/progress_info.h
    res/types.h
    res/utils.cpp res/utils.h
    res/win_ioctl.cpp res/win_ioctl.h
    virtual_fs/filenode.cpp virtual_fs/filenode.h
    virtual_fs/filenodes.cpp virtual_fs/filenodes.h
    virtual_fs/virtual_fs.cpp virtual_fs/virtual_fs.h
    virtual_fs/virtual_fs_helper.cpp virtual_fs/virtual_fs_helper.h
    virtual_fs/virtual_fs_operations.cpp virtual_fs/virtual_fs_operations.h
    ${gui}
)

set_target_properties(NxNandManager
    PROPERTIES
        WIN32_EXECUTABLE TRUE
)


target_include_directories(NxNandManager PRIVATE
    virtual_fs/dokan/include
    gui/include
    ../../OpenSSL_mingw64/include
)

target_compile_definitions(NxNandManager PRIVATE
    QT_DEPRECATED_WARNINGS
)

target_link_libraries(NxNandManager PRIVATE
    # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/../../../../../mingw64/lib
    # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/../../../../../mingw64/qt5-static/lib
    # Remove: pthread
    Qt::Core
    Qt::Gui
    Qt::Network
)

#target_compile_options(NxNandManager
#    --Wno-unused-function
#    --Wno-unused-variable
#    -fpermissive
#    -pthread
#    -std=c++0x
#)


# Resources:
set(application_resource_files
    "gui/images/autorcm.png"
    "gui/images/close-window-32.ico"
    "gui/images/copy.png"
    "gui/images/cut.png"
    "gui/images/decrypt.png"
    "gui/images/drive.png"
    "gui/images/emunand.png"
    "gui/images/encrypt.png"
    "gui/images/explorer.png"
    "gui/images/export.png"
    "gui/images/extract_decrypt.png"
    "gui/images/format.png"
    "gui/images/import.png"
    "gui/images/incognito.png"
    "gui/images/keyset.png"
    "gui/images/loader_bgFFFFFF.gif"
    "gui/images/loading_wheel.gif"
    "gui/images/logo.png"
    "gui/images/magnifying_glass.png"
    "gui/images/new.png"
    "gui/images/new_file.png"
    "gui/images/nsw.png"
    "gui/images/open.png"
    "gui/images/paste.png"
    "gui/images/resize.png"
    "gui/images/restore.png"
    "gui/images/save.png"
    "gui/images/save2.png"
    "gui/images/save_advanced.png"
    "gui/images/search.png"
    "gui/images/stop-32.ico"
    "gui/images/unbox.png"
    "gui/images/unlock_file.png"
    "gui/images/unmount.png"
    "gui/images/vfs_nca.png"
    "gui/res/all_titles.json"
    "gui/res/nca.json"
)

qt_add_resources(NxNandManager "application"
    PREFIX
        "/"
    FILES
        ${application_resource_files}
)

if((QT_VERSION_MAJOR GREATER 4))
    target_link_libraries(NxNandManager PRIVATE
        Qt::Widgets
    )
endif()

    target_sources(NxNandManager PUBLIC
        ${ziplibzlibc}
    )

if(ARCH32)
    target_compile_definitions(NxNandManager PRIVATE
        ARCH32
    )

    target_link_libraries(NxNandManager PRIVATE
        # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/../virtual_fs/dokan/x86/lib/
        dokan1
    )
endif()

if(ARCH64)
    target_compile_definitions(NxNandManager PRIVATE
        ARCH64
    )

    target_link_libraries(NxNandManager PRIVATE
        # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/../virtual_fs/dokan/x64/lib/
        dokan1
    )
endif()

if(WIN32)
    target_link_libraries(NxNandManager PRIVATE
        # Remove: L/lib/
        crypto
    )
endif()

if(WIN32 AND NOT win32-g++)
    target_link_libraries(NxNandManager PRIVATE
        crypto
    )
endif()

if((NOT (WIN32 AND NOT win32-g++)) AND (win32-g++))
    target_link_libraries(NxNandManager PRIVATE
        crypto
    )
endif()

install(TARGETS NxNandManager
    BUNDLE DESTINATION .
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

qt_generate_deploy_app_script(
    TARGET NxNandManager
    FILENAME_VARIABLE deploy_script
    NO_UNSUPPORTED_PLATFORM_ERROR
)
install(SCRIPT ${deploy_script})
