


# Linguist settings 

set(FORWARD_LANGS en_US uk_UA)

set(FORWARD_TS_FILES "")
set(FORWARD_QM_FILES "")

set(FORWARD_LANGS_DIR Resources/langs)
set(FORWARD_CURRENT_LANGS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Resources/langs)

foreach(lang IN LISTS FORWARD_LANGS)
    list(APPEND FORWARD_TS_FILES "${FORWARD_CURRENT_LANGS_DIR}/lang_${lang}.ts")
    list(APPEND FORWARD_QM_FILES "lang_${lang}.qm")
endforeach()

message(STATUS "TS Files")

foreach(ts IN LISTS FORWARD_TS_FILES)
    message("TS File generated: ${ts}")
endforeach()

set_source_files_properties(${FORWARD_TS_FILES} 
    PROPERTIES OUTPUT_LOCATION 
        ${CMAKE_CURRENT_BINARY_DIR}/${FORWARD_LANGS_DIR}
)

#

set(FORWARD_RESOURCES 
    Resources/qrc/Forward.qrc
    Resources/winrc/Forward.rc
    Resources/winrc/Forward.manifest
)
set(FORWARD_SOURCE
    Source/LogicLayer/main.cpp
)

set(FORWARD_QML_FILES "")
set(FORWARD_QML_SOURCE
    Source/FrontLayer/pages/App.qml
    Source/FrontLayer/pages/Login.qml
    Source/FrontLayer/pages/Start.qml
    Source/FrontLayer/pages/Main.qml
    Source/FrontLayer/pages/Conversation.qml
)
remove_path_from_list(FORWARD_QML_SOURCE "Source/FrontLayer/pages" FORWARD_QML_FILES)


# Registering RESOURCE_ALIAS for qml files

foreach(dir_file file_name IN ZIP_LISTS FORWARD_QML_SOURCE FORWARD_QML_FILES)
    set_source_files_properties(${dir_file} 
        PROPERTIES 
            QT_RESOURCE_ALIAS ${file_name}
    )
endforeach()

#

qt_add_executable(Forward WIN32 ${FORWARD_SOURCE} ${FORWARD_RESOURCES})

set_target_properties(Forward 
    PROPERTIES 
        AUTOMOC ON
        AUTORCC ON
)

target_link_libraries(Forward 
    PRIVATE 
        Qt::QuickControls2
        ForwardClientLibplugin
        ForwardUILibplugin
)

# Generating QM file from TS files

qt6_add_translations(Forward
    TS_FILES 
        ${FORWARD_TS_FILES}
    SOURCES
        ${FORWARD_QML_SOURCE}
        ${FORWARD_SOURCE}
    QM_FILES_OUTPUT_VARIABLE 
        FORWARD_QM_OUTPUT_FILES
)

message(STATUS "QM Genarating files")

foreach(dir_file file_name IN ZIP_LISTS FORWARD_QM_OUTPUT_FILES FORWARD_QM_FILES)
    set_source_files_properties(${dir_file} PROPERTIES QT_RESOURCE_ALIAS ${file_name})
endforeach()

foreach(qm IN LISTS FORWARD_QM_OUTPUT_FILES)
    message("QM File generated: ${qm}")
endforeach()

#

qt6_add_resources(Forward langs.qrc
    PREFIX /langs
    FILES
        ${FORWARD_QM_OUTPUT_FILES}
)
qt6_add_qml_module(Forward
    URI forward
    VERSION 1.0
    RESOURCE_PREFIX /imports
    QML_FILES 
        ${FORWARD_QML_SOURCE}
    NO_PLUGIN
)

set_target_properties(Forward PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER lib.forward.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

install(FILES ${qm_files} DESTINATION ${FORWARD_LANGS_DIR})

install(TARGETS Forward
    RUNTIME DESTINATION .
    BUNDLE DESTINATION .
    LIBRARY DESTINATION .
)
