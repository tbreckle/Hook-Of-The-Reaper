cmake_minimum_required(VERSION 3.19)
project(HookOfTheReaper LANGUAGES CXX)

# Include FetchContent for automatic dependency download
include(FetchContent)

# Option to select Qt version
# Use Qt5 for Windows 7/8, Qt6 for Windows 10/11
option(USE_QT6 "Use Qt6 instead of Qt5 (for Windows 10/11)" ON)

if(USE_QT6)
    message(STATUS "Building with Qt6 (Windows 10/11 support)")
    find_package(Qt6 6.5 REQUIRED COMPONENTS Core Widgets SerialPort Network Multimedia Bluetooth)
    set(QT_VERSION_MAJOR 6)
else()
    message(STATUS "Building with Qt5 (Windows 7/8 support)")
    find_package(Qt5 5.15 REQUIRED COMPONENTS Core Widgets SerialPort Network Multimedia Bluetooth)
    set(QT_VERSION_MAJOR 5)
endif()

# Qt standard setup (Qt6 only)
if(USE_QT6)
    qt_standard_project_setup()
endif()

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ============================================================================
# Automatic dependency download and setup
# ============================================================================

# Download and setup HIDAPI from GitHub
message(STATUS "Fetching HIDAPI from GitHub...")
FetchContent_Declare(
    hidapi
    GIT_REPOSITORY https://github.com/libusb/hidapi.git
    GIT_TAG hidapi-0.15.0  # Use a stable release tag
    GIT_SHALLOW TRUE
)
set(HIDAPI_BUILD_HIDTEST OFF CACHE BOOL "Build hidapi test applications" FORCE)
FetchContent_MakeAvailable(hidapi)

# Download and setup Ultimarc PacDrive SDK
message(STATUS "Fetching Ultimarc PacDrive SDK...")
FetchContent_Declare(
    ultimarcsdk
    URL https://www.ultimarc.com/PacDriveSDK.zip
    DOWNLOAD_NO_EXTRACT FALSE
)
FetchContent_GetProperties(ultimarcsdk)
if(NOT ultimarcsdk_POPULATED)
    FetchContent_Populate(ultimarcsdk)
    # Set paths for Ultimarc SDK
    set(ULTIMARC_SDK_DIR "${ultimarcsdk_SOURCE_DIR}")
    message(STATUS "Ultimarc SDK extracted to: ${ULTIMARC_SDK_DIR}")
endif()

# Set dependency paths
set(HIDAPI_INCLUDE_DIR "${hidapi_SOURCE_DIR}/hidapi")
if(WIN32)
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        # 64-bit build
        set(ULTIMARC_LIB_DIR "${ULTIMARC_SDK_DIR}/x64")
    else()
        # 32-bit build
        set(ULTIMARC_LIB_DIR "${ULTIMARC_SDK_DIR}/x86")
    endif()
else()
    set(ULTIMARC_LIB_DIR "${ULTIMARC_SDK_DIR}")
endif()

message(STATUS "HIDAPI include directory: ${HIDAPI_INCLUDE_DIR}")
message(STATUS "Ultimarc SDK library directory: ${ULTIMARC_LIB_DIR}")

# ============================================================================
# Source files definition
# ============================================================================

# Define source files list (shared between Qt5 and Qt6 builds)
# Organized by directory structure

# Root files
set(ROOT_FILES
    main.cpp
    ${app_icon_resource_windows}
    hookOfTheReaper.cpp
    hookOfTheReaper.h
    hookOfTheReaper.ui
    Global.h
    README.md
)

# Data files
set(DATA_FILES
    data/icons/icons.rc
    data/icons/hOTRIcon256.ico
    data/icons/icons.qrc
    data/sounds/sounds.qrc
    data/iniDefault.hor
    data/lgDefault.hor
    data/settings.hor
    data/mx24.hor
    data/rs3Reaper.hor
    data/jbgun4ir.hor
    data/nonDefaultLG.hor
    data/fusion.hor
    data/blamcon.hor
    data/alienUSB.hor
    data/openFire.hor
    data/xGunner.hor
)

# INI configuration files
set(INI_FILES
    ini/default.ini
    ini/MAME/digdug.ini
)

# Default light gun configuration files
set(DEFAULTLG_FILES
    defaultLG/hotd4.txt
    defaultLG/jp.txt
    defaultLG/rambo.txt
    defaultLG/lgi.txt
    defaultLG/lgi3d.txt
    defaultLG/lgj.txt
    defaultLG/alien3.txt
    defaultLG/confmiss.txt
    defaultLG/ptblank2.txt
)

# COM Device List sources
set(COMDEVICELIST_FILES
    COMDeviceList/ComDeviceList.cpp
    COMDeviceList/ComDeviceList.h
    COMDeviceList/LightGun.cpp
    COMDeviceList/LightGun.h
    COMDeviceList/LightController.h
    COMDeviceList/LightController.cpp
    COMDeviceList/PacDriveControl.h
    COMDeviceList/PacDriveControl.cpp
    COMDeviceList/LightExecution.h
    COMDeviceList/LightExecution.cpp
)

# Hooker Engine sources
set(HOOKERENGINE_FILES
    HookerEngine/HookCOMPort.cpp
    HookerEngine/HookCOMPort.h
    HookerEngine/HookerEngine.cpp
    HookerEngine/HookerEngine.h
    HookerEngine/HookTCPSocket.cpp
    HookerEngine/HookTCPSocket.h
    HookerEngine/HookCOMPortWin.h
    HookerEngine/HookCOMPortWin.cpp
    HookerEngine/GamePlayer.h
    HookerEngine/GamePlayer.cpp
    HookerEngine/HookLight.h
    HookerEngine/HookLight.cpp
    HookerEngine/LightCommand.h
    HookerEngine/LightCommand.cpp
)

# UI sources
set(WINDOWS_FILES
    Windows/playerAssignWindow.cpp
    Windows/playerAssignWindow.h
    Windows/playerAssignWindow.ui
    Windows/settingsWindow.cpp
    Windows/settingsWindow.h
    Windows/settingsWindow.ui
    Windows/aboutWindow.cpp
    Windows/aboutWindow.h
    Windows/aboutWindow.ui
    Windows/testComPortWindow.h
    Windows/testComPortWindow.cpp
    Windows/testComPortWindow.ui
    Windows/addLightGunWindowV2.h
    Windows/addLightGunWindowV2.cpp
    Windows/addLightGunWindowV2.ui
    Windows/editLightGunWindowV2.h
    Windows/editLightGunWindowV2.cpp
    Windows/editLightGunWindowV2.ui
    Windows/addLightControllerWindow.h
    Windows/addLightControllerWindow.cpp
    Windows/addLightControllerWindow.ui
    Windows/editLightControllerWindow.h
    Windows/editLightControllerWindow.cpp
    Windows/editLightControllerWindow.ui
)

# Combine all source files
set(SOURCE_FILES
    ${ROOT_FILES}
    ${DATA_FILES}
    ${INI_FILES}
    ${DEFAULTLG_FILES}
    ${COMDEVICELIST_FILES}
    ${HOOKERENGINE_FILES}
    ${WINDOWS_FILES}
)

# Create executable with appropriate command for Qt version
if(USE_QT6)
    qt_add_executable(HookOfTheReaper WIN32 MACOSX_BUNDLE ${SOURCE_FILES})
else()
    add_executable(HookOfTheReaper WIN32 MACOSX_BUNDLE ${SOURCE_FILES})
endif()


# Set include directories based on Qt version
if(USE_QT6)
    target_include_directories(HookOfTheReaper
        PRIVATE
        ${Qt6Core_INCLUDE_DIRS}
        ${Qt6Widgets_INCLUDE_DIRS}
        ${Qt6SerialPort_INCLUDE_DIRS}
        ${Qt6Network_INCLUDE_DIRS}
        ${Qt6Multimedia_INCLUDE_DIRS}
        ${Qt6Bluetooth_INCLUDE_DIRS}
        ${HIDAPI_INCLUDE_DIR}
        ${ULTIMARC_SDK_DIR}
    )
else()
    target_include_directories(HookOfTheReaper
        PRIVATE
        ${Qt5Core_INCLUDE_DIRS}
        ${Qt5Widgets_INCLUDE_DIRS}
        ${Qt5SerialPort_INCLUDE_DIRS}
        ${Qt5Network_INCLUDE_DIRS}
        ${Qt5Multimedia_INCLUDE_DIRS}
        ${Qt5Bluetooth_INCLUDE_DIRS}
        ${HIDAPI_INCLUDE_DIR}
        ${ULTIMARC_SDK_DIR}
    )
endif()

target_link_directories(HookOfTheReaper
    PRIVATE
    ${ULTIMARC_LIB_DIR}
)

# Link libraries based on Qt version
if(USE_QT6)
    target_link_libraries(HookOfTheReaper
        PRIVATE
            Qt6::Core
            Qt6::Widgets
            Qt6::SerialPort
            Qt6::Network
            Qt6::Multimedia
            Qt6::Bluetooth
            hidapi::hidapi
            PacDrive64
    )
else()
    target_link_libraries(HookOfTheReaper
        PRIVATE
            Qt5::Core
            Qt5::Widgets
            Qt5::SerialPort
            Qt5::Network
            Qt5::Multimedia
            Qt5::Bluetooth
            hidapi::hidapi
            PacDrive64
    )
endif()


include(GNUInstallDirs)

install(TARGETS HookOfTheReaper
    BUNDLE  DESTINATION .
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# Qt deployment script (Qt6 only)
if(USE_QT6)
    qt_generate_deploy_app_script(
        TARGET HookOfTheReaper
        OUTPUT_SCRIPT deploy_script
        NO_UNSUPPORTED_PLATFORM_ERROR
        NO_TRANSLATIONS
    )
    install(SCRIPT ${deploy_script})
endif()

set(app_icon_resource_windows "${CMAKE_CURRENT_SOURCE_DIR}/data/icons/icons.rc")

target_link_options(HookOfTheReaper PRIVATE "-Wl,--start-group")

# Windows version targeting
if(WIN32)
    if(USE_QT6)
        # Qt6: Windows 10/11 (minimum version 10.0)
        target_compile_definitions(HookOfTheReaper PRIVATE 
            _WIN32_WINNT=0x0A00  # Windows 10
            WINVER=0x0A00
        )
        message(STATUS "Targeting Windows 10/11")
    else()
        # Qt5: Windows 7/8 (minimum version 6.1)
        target_compile_definitions(HookOfTheReaper PRIVATE 
            _WIN32_WINNT=0x0601  # Windows 7
            WINVER=0x0601
        )
        message(STATUS "Targeting Windows 7/8")
    endif()
else()
    message(FATAL_ERROR "This project is only supported on Windows platforms. Current platform: ${CMAKE_SYSTEM_NAME}")
endif()
