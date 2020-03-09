cmake_minimum_required(VERSION 3.10.1)

file(STRINGS "${CMAKE_SOURCE_DIR}/makespec/VERSION" QV2RAY_VERSION)
file(STRINGS "${CMAKE_SOURCE_DIR}/makespec/BUILDVERSION" QV2RAY_BUILD_VERSION)

if(NOT ${CMAKE_BUILD_TYPE} STREQUAL "Release")
    math(EXPR QV2RAY_BUILD_VERSION "1 + ${QV2RAY_BUILD_VERSION}" OUTPUT_FORMAT DECIMAL)
    message("Increasing BUILDVERSION")
    file(WRITE "${CMAKE_SOURCE_DIR}/makespec/BUILDVERSION" ${QV2RAY_BUILD_VERSION})
endif()

set(PACKAGE_VERSION "${QV2RAY_VERSION}.${QV2RAY_BUILD_VERSION}")
project(qv2ray)

set(VERSION_LIST ${PACKAGE_VERSION})
string(REPLACE "." ";" VERSION_LIST ${VERSION_LIST})
separate_arguments(VERSION_LIST)

list(GET VERSION_LIST 0 QV2RAY_VERSION_MAJOR)
list(GET VERSION_LIST 1 QV2RAY_VERSION_MINOR)
list(GET VERSION_LIST 2 QV2RAY_VERSION_BUGFIX)
list(GET VERSION_LIST 3 QV2RAY_VERSION_BUILD)

add_definitions(-DQV2RAY_VERSION_MAJOR=${QV2RAY_VERSION_MAJOR})
add_definitions(-DQV2RAY_VERSION_MINOR=${QV2RAY_VERSION_MINOR})
add_definitions(-DQV2RAY_VERSION_BUGFIX=${QV2RAY_VERSION_BUGFIX})
add_definitions(-DQV2RAY_VERSION_BUILD=${QV2RAY_VERSION_BUILD})

add_definitions(-DQV2RAY_VERSION_STRING="v${PACKAGE_VERSION}")
add_definitions(-DXTOSTRUCT_QT)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt5 COMPONENTS Core Gui Widgets Network REQUIRED)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

cmake_policy(SET CMP0071 NEW)

if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

message(" ")
message("Qv2ray Version: v${PACKAGE_VERSION}")
message("|-------------------------------------------------|")
message("| Qv2ray, A Cross Platform v2ray Qt GUI Client.   |")
message("| Licenced under GPLv3                            |")
message("|                                                 |")
message("| You may only use this program to the extent     |")
message("| permitted by local law.                         |")
message("|                                                 |")
message("| See: https://www.gnu.org/licenses/gpl-3.0.html  |")
message("|-------------------------------------------------|")
message("| Project Homepage: https://github.com/Qv2ray     |")
message("| Welcome to contribute!                          |")
message("|-------------------------------------------------|")
message(" ")

if(WIN32)
  add_compile_options("/std:c++17")
  add_definitions(-DUNICODE -D_UNICODE)
  add_definitions(-D_HAS_STD_BYTE=0 -D_WIN32_WINNT=0x600 -D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS)
  set(GUI_TYPE WIN32)
  if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
      if(CMAKE_CL_64)
        include(${CMAKE_SOURCE_DIR}/libs/x64-windows/scripts/buildsystems/vcpkg.cmake)
      else()
        include(${CMAKE_SOURCE_DIR}/libs/x86-windows/scripts/buildsystems/vcpkg.cmake)
      endif()
  endif()
endif()

if(UNIX AND NOT APPLE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
endif()

include(cmake/translations.cmake)
include(cmake/qnodeeditor.cmake)
include(cmake/singleapplication.cmake)
include(cmake/qzxing.cmake)
include(cmake/protobuf.cmake)
include(cmake/cpp-httplib.cmake)
include(cmake/backend.cmake)

if(APPLE)
  find_package(Iconv REQUIRED)
  set(MACOSX_FRAMEWORK
    "-framework Carbon"
    "-framework Cocoa"
    "-framework Security"
  )
  set(MACOSX_ICONV_LIBRARY
    Iconv::Iconv
  )

  set(MACOSX_ICONV_INCLUDE_PATH
    ${Iconv_INCLUDE_DIR}
  )
endif()

if(QV2RAY_BUILD_INFO)
    set(_QV2RAY_BUILD_INFO_STR_ "${QV2RAY_BUILD_INFO}")
elseif(DEFINED ENV{_QV2RAY_BUILD_INFO_})
    set(_QV2RAY_BUILD_INFO_STR_ "$ENV{_QV2RAY_BUILD_INFO_}")
else()
    set(_QV2RAY_BUILD_INFO_STR_ "Qv2ray from manual build")
endif()

if(QV2RAY_BUILD_EXTRA_INFO)
    set(_QV2RAY_BUILD_EXTRA_INFO_STR_ "${QV2RAY_BUILD_EXTRA_INFO}")
elseif(DEFINED ENV{_QV2RAY_BUILD_EXTRA_INFO_})
    set(_QV2RAY_BUILD_EXTRA_INFO_STR_ "$ENV{_QV2RAY_BUILD_EXTRA_INFO_}")
else()
    set(_QV2RAY_BUILD_EXTRA_INFO_STR_ "Qv2ray v${PACKAGE_VERSION}")
endif()

set(QV2RAY_BUILD_INFO ${_QV2RAY_BUILD_INFO_STR_} CACHE STRING "Qv2ray build info")
set(QV2RAY_BUILD_EXTRA_INFO ${_QV2RAY_BUILD_EXTRA_INFO_STR_} CACHE STRING "Qv2ray build extra info")

add_definitions(-D_QV2RAY_BUILD_INFO_STR_="${_QV2RAY_BUILD_INFO_STR_}")
add_definitions(-D_QV2RAY_BUILD_EXTRA_INFO_STR_="${_QV2RAY_BUILD_EXTRA_INFO_STR_}")
message("Qv2ray build info: ${_QV2RAY_BUILD_INFO_STR_}")
message("Qv2ray build info ex: ${_QV2RAY_BUILD_EXTRA_INFO_STR_}")

set(QV2RAY_DEFAULT_VASSETS_PATH "unset" CACHE STRING "v2ray assets path")
set(QV2RAY_DEFAULT_VCORE_PATH "unset" CACHE STRING "v2ray core path")
set(QV2RAY_TRANSLATION_PATH "unset" CACHE STRING "Qv2ray translations path")
set(EMBED_TRANSLATIONS OFF CACHE BOOL "Embed translations")

if(QV2RAY_DEFAULT_VASSETS_PATH AND NOT STREQUAL "unset")
  add_definitions(-DQV2RAY_DEFAULT_VASSETS_PATH="${QV2RAY_DEFAULT_VASSETS_PATH}")
endif()

if(QV2RAY_DEFAULT_VCORE_PATH AND NOT STREQUAL "unset")
  add_definitions(-DQV2RAY_DEFAULT_VCORE_PATH="${QV2RAY_DEFAULT_VCORE_PATH}")
endif()

if(QV2RAY_TRANSLATION_PATH AND NOT STREQUAL "unset")
  add_definitions(-DQV2RAY_TRANSLATION_PATH="${QV2RAY_TRANSLATION_PATH}")
endif()

set(QV2RAY_SOURCES
  src/base/Qv2rayLog.cpp
  src/common/CommandArgs.cpp
  src/common/HTTPRequestHelper.cpp
  src/common/LogHighlighter.cpp
  src/common/QJsonModel.cpp
  src/common/QvHelpers.cpp
  src/common/QvTranslator.cpp
  src/components/autolaunch/QvAutoLaunch.cpp
  src/components/geosite/QvGeositeReader.cpp
  src/components/icmping/win/ICMPPinger.cpp
  src/components/pac/QvGFWPACConverter.cpp
  src/components/pac/QvPACHandler.cpp
  src/components/plugins/toolbar/QvToolbar.cpp
  src/components/plugins/toolbar/QvToolbar_linux.cpp
  src/components/plugins/toolbar/QvToolbar_win.cpp
  src/components/proxy/QvProxyConfigurator.cpp
  src/components/speedchart/speedplotview.cpp
  src/components/speedchart/speedwidget.cpp
  src/core/connection/ConnectionIO.cpp
  src/core/connection/Generation.cpp
  src/core/connection/Serialization.cpp
  src/core/connection/Serialization_ss.cpp
  src/core/connection/Serialization_ssd.cpp
  src/core/connection/Serialization_vmess.cpp
  src/core/CoreUtils.cpp
  src/core/handler/ConfigHandler.cpp
  src/core/handler/V2rayInstanceHandler.cpp
  src/core/kernel/APIBackend.cpp
  src/core/kernel/KernelInteractions.cpp
  src/core/settings/SettingsBackend.cpp
  src/core/settings/SettingsUpgrade.cpp
  src/core/tcping/QvTCPing.cpp
  src/main.cpp
  src/ui/editors/w_InboundEditor.cpp
  src/ui/editors/w_JsonEditor.cpp
  src/ui/editors/w_OutboundEditor.cpp
  src/ui/editors/w_RoutesEditor.cpp
  src/ui/editors/w_RoutesEditor_extra.cpp
  src/ui/messaging/QvMessageBus.cpp
  src/ui/models/InboundNodeModel.cpp
  src/ui/models/OutboundNodeModel.cpp
  src/ui/models/RuleNodeModel.cpp
  src/ui/widgets/ConnectionInfoWidget.cpp
  src/ui/widgets/ConnectionItemWidget.cpp
  src/ui/widgets/QvAutoCompleteTextEdit.cpp
  src/ui/widgets/StreamSettingsWidget.cpp
  src/ui/w_ExportConfig.cpp
  src/ui/w_ImportConfig.cpp
  src/ui/w_MainWindow.cpp
  src/ui/w_MainWindow_extra.cpp
  src/ui/w_PreferencesWindow.cpp
  src/ui/w_ScreenShot_Core.cpp
  src/ui/w_SubscriptionManager.cpp
  # ui files
  src/ui/w_ExportConfig.ui
  src/ui/w_SubscriptionManager.ui
  src/ui/editors/w_OutboundEditor.ui
  src/ui/editors/w_InboundEditor.ui
  src/ui/editors/w_JsonEditor.ui
  src/ui/editors/w_RoutesEditor.ui
  src/ui/w_ImportConfig.ui
  src/ui/widgets/StreamSettingsWidget.ui
  src/ui/widgets/ConnectionInfoWidget.ui
  src/ui/widgets/ConnectionItemWidget.ui
  src/ui/w_MainWindow.ui
  src/ui/w_PreferencesWindow.ui
  src/ui/w_ScreenShot_Core.ui

  assets/qv2ray.rc
)

if(EMBED_TRANSLATIONS)
  add_definitions(-DEMBED_TRANSLATIONS)
  configure_file(translations/translations.qrc ${CMAKE_BINARY_DIR} COPYONLY)
  list(APPEND QV2RAY_SOURCES ${CMAKE_BINARY_DIR}/translations.qrc)
endif()

set(QRC_RESOURCES ${CMAKE_SOURCE_DIR}/resources.qrc)

set(QT_LIBRARY
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::Network
)

add_executable(${PROJECT_NAME}
  ${GUI_TYPE}
  ${QV2RAY_SOURCES}
  ${QNODEEDITOR_SOURCES}
  ${QNODEEDITOR_QRC_RESOURCES}
  ${SINGLEAPPLICATION_SOURCES}
  ${QZXING_SOURCES}
  ${PROTO_SRCS}
  ${PROTO_HDRS}
  ${API_GRPC_SRCS}
  ${API_PROTO_SRCS}
  ${QRC_RESOURCES}
  ${QM_FILES}
)

target_link_libraries(${PROJECT_NAME}
  ${QT_LIBRARY}
  ${MACOSX_FRAMEWORK}
  ${MACOSX_ICONV_LIBRARY}
  ${QV2RAY_PROTOBUF_LIBRARY}
  ${QV2RAY_BACKEND_LIBRARIES}
)


target_include_directories(${PROJECT_NAME} PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_BINARY_DIR}
  ${QHTTPSERVER_DIR}
  ${QNODEEDITOR_INCLUDE_PATH}
  ${QZXING_INCLUDE_PATH}
  ${SINGLEAPPLICATION_DIR}
  ${Protobuf_INCLUDE_DIRS}
  ${MACOSX_ICONV_INCLUDE_PATH}
  ${cpp-httplib_INCLUDE_DIRS}
)

if(APPLE)
    set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR})
    set(prefix "${PROJECT_NAME}.app/Contents")
    set(INSTALL_RUNTIME_DIR "${prefix}/MacOS")
    set(INSTALL_CMAKE_DIR "${prefix}/Resources")

    # Destination paths below are relative to ${CMAKE_INSTALL_PREFIX}
    install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR} )
    install(FILES ${CMAKE_SOURCE_DIR}/assets/Info.plist DESTINATION "${prefix}" )
    install(FILES ${CMAKE_SOURCE_DIR}/assets/icons/qv2ray.icns DESTINATION "${INSTALL_CMAKE_DIR}" )
    if(NOT EMBED_TRANSLATIONS)
      install(FILES ${QM_FILES} DESTINATION ${INSTALL_CMAKE_DIR}/lang)
    endif()

    # Call macdeployqt
    install(CODE "execute_process(COMMAND macdeployqt ${PROJECT_NAME}.app)")
    set(APPS "\$ENV{DESTDIR}\${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.app")
    include(cmake/deployment.cmake)
endif()

if(UNIX AND NOT APPLE AND NOT WIN32)
  install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)
  install(FILES assets/qv2ray.metainfo.xml DESTINATION share/metainfo)
  install(FILES assets/qv2ray.desktop DESTINATION share/applications)
  install(FILES assets/icons/qv2ray.png DESTINATION share/icons/hicolor/256x256/apps/)
  if(NOT EMBED_TRANSLATIONS)
    install(FILES ${QM_FILES} DESTINATION share/qv2ray/lang)
  endif()
endif()

if(WIN32)
  target_link_libraries(${PROJECT_NAME} wininet wsock32 ws2_32 user32)
  install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION .)
  if(NOT EMBED_TRANSLATIONS)
    install(FILES ${QM_FILES} DESTINATION lang)
  endif()
  
  include(cmake/FindOpenSSL.cmake)
  install(FILES ${SSL_LIBRARY} DESTINATION .)
  install(FILES ${CRYPTO_LIBRARY} DESTINATION .)

  install(CODE "execute_process(COMMAND windeployqt ${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.exe --compiler-runtime --verbose 2)")
  set(APPS "\$ENV{DESTDIR}\${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.exe")
  include(cmake/deployment.cmake)
endif()