SET(CMAKE_AUTOMOC ON)
SET(CMAKE_AUTOUIC ON)
SET(CMAKE_AUTORCC ON)
SET(CMAKE_INCLUDE_CURRENT_DIR ON)

add_executable(net64_qt_gui
                resources.qrc
                main.cpp
                mainframe.cpp
                mainframe.hpp
                mainframe.ui
                m64p_settings_window.cpp
                m64p_settings_window.hpp
                app_settings.cpp
                app_settings.hpp
                mario20client.cpp
                mario20client.hpp
                mario20server.cpp
                mario20server.hpp
                mario20protocol.cpp
                mario20protocol.hpp)

target_link_libraries(net64_qt_gui net64_core)

# Qt libs
find_package(Qt5 COMPONENTS Core Widgets WebSockets REQUIRED)
target_link_libraries(net64_qt_gui Qt5::Core Qt5::Widgets Qt5::WebSockets)
target_include_directories(net64_qt_gui PRIVATE ${Qt5_Widgtes_INCLUDES})

# json library
target_link_libraries(net64_qt_gui nlohmann_json::nlohmann_json)

# zlib
target_link_libraries(net64_qt_gui zlibstatic)

if(UNIX)
    target_link_libraries(net64_qt_gui -lpthread -lstdc++fs)
endif()

if(WIN32)
    # Include MSVC libs
    set(CMAKE_INSTALL_SYSTEM_RUNTIME_DESTINATION usr/bin)
    set(CMAKE_INSTALL_UCRT_LIBRARIES TRUE)
    include(InstallRequiredSystemLibraries)
    # Run windeployqt
    get_target_property(MOC_EXECUTABLE_LOCATION Qt5::moc IMPORTED_LOCATION)
    get_filename_component(QT_BINARY_DIRECTORY "${MOC_EXECUTABLE_LOCATION}" DIRECTORY)
    find_program(WINDEPLOYQT_EXE windeployqt HINTS "${QT_BINARY_DIRECTORY}")

    add_custom_command(TARGET net64_qt_gui POST_BUILD
        COMMAND "${WINDEPLOYQT_EXE}" --no-translations --no-compiler-runtime "$<TARGET_FILE:net64_qt_gui>"
        WORKING_DIRECTORY "${QT_BINARY_DIRECTORY}"
    )
endif()


# Installation
if(CMAKE_CONFIGURATION_TYPES)
    install(DIRECTORY "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIG>/"
            DESTINATION ${CMAKE_INSTALL_PREFIX}/usr/bin)
else()
    install(DIRECTORY "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/"
            DESTINATION ${CMAKE_INSTALL_PREFIX}/usr/bin)
endif()
