if(WIN32)
    cmake_minimum_required(VERSION 3.1.0)
    set(CMAKE_CXX_STANDARD 11)
else()
    cmake_minimum_required(VERSION 2.8.12)
    add_compile_options(-std=c++11)
endif()

# --- BUILD CONFIGURATION ---

project(HorizonLauncher C CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(SOURCE
    Source/Main.cpp
    Source/TabWidget.cpp
    Source/Library.cpp
    Source/MainPanel.cpp
    Source/Database.cpp
    Source/DRMSetupWizard.cpp
    Source/Navbar.cpp
    Source/Homepage.cpp
    Source/BuddyList.cpp
    Source/Community.cpp
    Source/AddGameWizard.cpp
    Source/Settings.cpp
    Source/GameLauncher.cpp
    Source/News.cpp
    Source/NewsItemWidget.cpp
    Source/SetupWizard.cpp
    Source/GridGameWidget.cpp
    Source/NewsFeedChooserWindow.cpp
)

set(HEADERS
    Source/TabWidget.h
    Source/Library.h
    Source/MainPanel.h
    Source/Database.h
    Source/Init.h
    Source/Navbar.h
    Source/Libs/SteamVdfParse.hpp
    Source/DRMSetupWizard.h
    Source/Homepage.h
    Source/BuddyList.h
    Source/Community.h
    Source/AddGameWizard.h
    Source/Settings.h
    Source/GameLauncher.h
    Source/News.h
    Source/NewsItemWidget.h
    Source.SetupWizard.h
    Source/Defines.h
    Source/GridGameWidget.h
    Source/NewsFeedChooserWindow.h
)

set(FORMS
    Resource/UI/Community.ui
    Resource/UI/Settings.ui
)

set(RESOURCES
    Resource/Icons.qrc
    Resource/Fonts.qrc
    Resource/Styles.qrc
    Resource/Images.qrc
)

set(GUI_TYPE "")

set(TRANSLATIONS
    Translations/horizon_da.ts
    Translations/horizon_nb.ts
    Translations/horizon_nb.ts
    Translations/horizon_pt_BR.ts
    Translations/horizon_fr.ts
)

if(WIN32)
    #Additional configuration for Windows systems.
    # TODO: Get working "Release" config for multi-configuration MSVC builds.

    # set(GUI_TYPE WIN32)

    set(SOURCE ${SOURCE}
        Source/BorderlessWindow.cpp
        Source/WinWindow.cpp
        Source/WinPanel.cpp
        BorderlessWindow/QWinHost.cpp
        BorderlessWindow/QWinWidget.cpp
    )

    set(HEADERS ${HEADERS}
        Source/BorderlessWindow.h
        Source/WinWindow.h
        Source/WinPanel.h
        BorderlessWindow/QWinHost.h
        BorderlessWindow/QWinWidget.h
    )
elseif(UNIX)
    # Additional configuration for Unix systems.
    # TODO: Get working icon / bundle for OSX systems without bricking the build.

    # if(APPLE)
    #     set(GUI_TYPE MACOSX_BUNDLE)
    # endif(APPLE)

    set(SOURCE ${SOURCE}
        Source/UnixWindow.cpp
        Source/UnixPanel.cpp
    )

    set(HEADERS ${HEADERS}
        Source/UnixWindow.h
        Source/UnixPanel.h
    )
else()
    # Suspend the configuration process if the target OS isn't recognized.
    MESSAGE(FATAL_ERROR "Target OS not recognized or supported, aborting CMake process.")
endif()

find_package(Qt5Widgets REQUIRED)
find_package(Qt5Core REQUIRED)
find_package(Qt5Gui REQUIRED)
find_package(Qt5Network REQUIRED)
# find_package(Qt5WebKit REQUIRED)
# find_package(Qt5WebKitWidgets REQUIRED)
find_package(Qt5Sql REQUIRED)
find_package(Boost REQUIRED)
find_package(Threads REQUIRED)
find_package(Qt5LinguistTools REQUIRED)

qt5_add_resources(UI_RESOURCES ${RESOURCES})
qt5_add_translation(TRANSLATIONS ${TRANSLATIONS})
include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${Qt5Gui_INCLUDE_DIRS}
    ${Qt5Gui_PRIVATE_INCLUDE_DIRS}
    ${Qt5Widgets_INCLUDE_DIRS}
    ${Qt5Core_INCLUDE_DIRS}
    ${Qt5Network_INCLUDE_DIRS}
#   ${Qt5WebKit_INCLUDE_DIRS}
#   ${Qt5WebKitWidgets_INCLUDE_DIRS}
    ${Qt5Sql_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIRS}
    BorderlessWindow
    Source
)
add_executable(HorizonLauncher
    ${GUI_TYPE}
    ${SOURCE}
    ${OS_SOURCE}
    ${UI_RESOURCES}
    ${TRANSLATIONS}
)
target_link_libraries(HorizonLauncher
    Qt5::Widgets
    Qt5::Core
    Qt5::Gui
  	Qt5::Network
#	Qt5::WebKit
#	Qt5::WebkitWidgets
    Qt5::Sql
    ${Boost_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
)

if(WIN32)
    target_link_libraries(HorizonLauncher
        user32.lib
        gdi32.lib
        dwmapi.lib
    )

    if(MSVC)
        set(WIN_LINKFLAGS \"${CMAKE_CURRENT_SOURCE_DIR}/Resource/Launcher.res\")
        set_target_properties(HorizonLauncher PROPERTIES LINK_FLAGS ${WIN_LINKFLAGS})
        set_target_properties(HorizonLauncher PROPERTIES COMPILE_FLAGS "/EHsc")
    endif(MSVC)
endif()

if(WITH_TESTS)
    enable_testing()
    add_executable(test_runner
        Source/Database.cpp
        Tests/Test.cpp)
    target_link_libraries(test_runner
        Qt5::Sql)
    add_test(test_runner test_runner)
    foreach(FILE
            appinfo.vdf
            game)
    add_custom_command(TARGET HorizonLauncher POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/Tests/${FILE} $<TARGET_FILE_DIR:HorizonLauncher>)
        endforeach()
endif(WITH_TESTS)

# --- RUNTIME DEPENDENCIES ---
# Only necessary on Windows.

if(WIN32)
    cmake_policy(SET CMP0026 OLD)

    get_target_property(BuildLocation HorizonLauncher LOCATION)
    get_filename_component(TargetDirectory ${BuildLocation} PATH)

    if (EXISTS ${Qt5Core_DIR}/../../../bin/icuin53.dll)
        set(ICU_VER 53)
    else()
        set(ICU_VER 54)
    endif()

    foreach(DEPENDENCY
        Qt5Widgetsd.dll
        Qt5Guid.dll
        Qt5Cored.dll
        Qt5Networkd.dll
#       Qt5WebKitd.dll
#       Qt5WebKitWidgetsd.dll
        Qt5Sqld.dll
        Qt5Sensorsd.dll
        Qt5Positioningd.dll
        Qt5Quickd.dll
        Qt5Qmld.dll
        Qt5Multimediad.dll
        Qt5MultimediaWidgetsd.dll
#       Qt5WebChanneld.dll
        Qt5Svgd.dll
        Qt5OpenGLd.dll
        Qt5PrintSupportd.dll
        icuin${ICU_VER}.dll
        icuuc${ICU_VER}.dll
        icudt${ICU_VER}.dll
    )
        add_custom_command(TARGET HorizonLauncher POST_BUILD
            # Copy the given dependency into the executable directory.
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${Qt5Core_DIR}/../../../bin/${DEPENDENCY} ${TargetDirectory}
            COMMENT "Copying ${DEPENDENCY} into '${TargetDirectory}/'" VERBATIM
        )
    endforeach()
endif()
