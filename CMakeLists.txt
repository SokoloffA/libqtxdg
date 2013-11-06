cmake_minimum_required( VERSION 2.6 )

project(libxdgqt)


set(PUBLIC_H_FILES
    xdgaction.h
    xdgdesktopfile.h
    xdgdirs.h
    xdgicon.h
    xdgmenu.h
    xdgmenuwidget.h
    xdgmime.h
    xmlhelper.h
    xdgautostart.h
)

set(PUBLIC_CLASSES
    XdgAction
    XdgDesktopFile
    XdgDirs
    XdgIcon
    XdgMenu
    XdgMenuWidget
    XdgMime
    XmlHelper
    XdgAutoStart
)

set(PRIVATE_H_FILES
    xdgmenuapplinkprocessor.h
    xdgmenulayoutprocessor.h
    xdgmenu_p.h
    xdgmenureader.h
    xdgmenurules.h
    qiconfix/qiconloader_p.h
)

set(CPP_FILES
    xdgaction.cpp
    xdgdesktopfile.cpp
    xdgdirs.cpp
    xdgicon.cpp
    xdgmenuapplinkprocessor.cpp
    xdgmenu.cpp
    xdgmenulayoutprocessor.cpp
    xdgmenureader.cpp
    xdgmenurules.cpp
    xdgmenuwidget.cpp
    xdgmime.cpp
    xmlhelper.cpp
    xdgautostart.cpp
    qiconfix/qiconloader.cpp
)

set(MOCS
    xdgaction.h
    xdgmenuapplinkprocessor.h
    xdgmenu.h
    xdgmenu_p.h
    xdgmenureader.h
    xdgmenurules.h
    xdgmenuwidget.h
)




file(GLOB TS_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/translations/libxdgqt_*.ts
)


option(USE_QT5 "Force use the Qt5." $ENV{USE_QT5})
option(USE_QT4 "Force use the Qt4." $ENV{USE_QT4})

if((USE_QT4 AND USE_QT5) OR
   (NOT USE_QT4 AND NOT USE_QT5))
    # Autodetect Qt version
    find_package(Qt5Core QUIET)
    if (Qt5Core_FOUND)
        set(USE_QT4 OFF)
        set(USE_QT5 ON)
    else()
        set(USE_QT4 ON)
        set(USE_QT5 OFF)
    endif()
endif()

option(BUILD_TESTS "Builds tests" OFF)

# Standard directories for installation
include(GNUInstallDirs)

# additional cmake files
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

set (MAJOR_VERSION 0)
set (MINOR_VERSION 5)
set (PATCH_VERSION 0)
set(XDGQT_VERSION_STRING ${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION})
set(XDGQT_MAJOR_VERSION ${MAJOR_VERSION})
set(XDGQT_MINOR_VERSION ${MINOR_VERSION})
set(XDGQT_PATCH_VERSION ${PATCH_VERSION})


add_definitions(-Wall)

if(USE_QT5)
    cmake_minimum_required(VERSION 2.8.3)

    set(CMAKE_INCLUDE_CURRENT_DIR ON)
    set(CMAKE_AUTOMOC ON)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)

    find_package(Qt5Widgets REQUIRED QUIET)
    find_package(Qt5Xml REQUIRED QUIET)
    find_package(Qt5LinguistTools REQUIRED QUIET)
    if (BUILD_TESTS)
        find_package(Qt5Test REQUIRED QUIET)
    endif()

    set(XDGQT_QT_VERSION "5")
    set(XDGQT_PKG_CONFIG_REQUIRES "Qt5Widgets Qt5Xml")

    include_directories(${Qt5Widgets_INCLUDE_DIRS} ${Qt5Xml_INCLUDE_DIRS})
    add_definitions(${Qt5Widgets_DEFINITIONS})

    #set(CMAKE_CXX_FLAGS
    #    "${CMAKE_CXX_FLAGS} ${Qt5Widgets_EXECUTABLE_COMPILE_FLAGS}"
    #)

    set(QT_LIBRARIES ${Qt5Widgets_LIBRARIES} ${Qt5Xml_LIBRARIES})

    message(STATUS "Building with Qt ${Qt5Core_VERSION_STRING}")
    QT5_ADD_TRANSLATION(QM_FILES ${TS_FILES})
else()

    find_package(Qt4 REQUIRED QtCore QtGui QtXml QUIET)
    if (BUILD_TESTS)
        find_package(Qt4 REQUIRED QtTest QUIET)
    endif()

    set(XDGQT_QT_VERSION "4")
    set(XDGQT_PKG_CONFIG_REQUIRES "QtCore, QtXml")

    include(${QT_USE_FILE})
    message(STATUS "Building with Qt ${QTVERSION}")
    QT4_WRAP_CPP(CXX_FILES ${MOCS})
    QT4_ADD_TRANSLATION(QM_FILES ${TS_FILES})
endif()

set(XDGQT_LIBRARY_NAME "xdgqt${XDGQT_QT_VERSION}")
set(XDGQT_FILE_NAME "xdgqt${XDGQT_QT_VERSION}")

set(XDGQT_PKG_CONFIG_DESCRIPTION "XdgQt${XDGQT_QT_VERSION}, a Qt${XDGQT_QT_VERSION} implementation of XDG standards")



find_package(LibMagic REQUIRED QUIET)

include(create_portable_headers)
#************************************************
# Create in-tree build infrastructure
#************************************************
if(INTREE_ROOT_DIR)
    # Copy public headers
    foreach(h ${PUBLIC_H_FILES})
        configure_file(${h} ${INTREE_ROOT_DIR}/include/xdgqt/${h} COPYONLY)
    endforeach()

    # Create the portable headers
    create_portable_headers(INTREE_PORTABLE_HEADERS ${INTREE_ROOT_DIR}/include/xdgqt ${PUBLIC_CLASSES})

    # Create config.cmake & use.cmake files
    set(XDGQT_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR};${INTREE_ROOT_DIR}/include")
    configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/cmake/xdgqt-config.cmake.in
        ${INTREE_ROOT_DIR}/cmake/${XDGQT_FILE_NAME}-config.cmake
        @ONLY
    )

    configure_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/cmake/xdgqt_use.cmake
        ${INTREE_ROOT_DIR}/cmake/${XDGQT_FILE_NAME}_use.cmake
        @ONLY
    )
endif()

#************************************************
# Create instalable build infrastructure
#************************************************

# Create the portable headers
create_portable_headers(PORTABLE_HEADERS ${CMAKE_CURRENT_BINARY_DIR} ${PUBLIC_CLASSES})

# Create config.cmake & use.cmake files
set(XDGQT_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR}/${XDGQT_FILE_NAME})

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/xdgqt-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${XDGQT_FILE_NAME}-config.cmake
    @ONLY
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/xdgqt_use.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/cmake/${XDGQT_FILE_NAME}_use.cmake
    @ONLY
)

#**********************************************************

include(FindLibSuffix)

#if(USE_QT5)
#    QT5_WRAP_CPP(CXX_FILES ${MOCS})
#else()
#    QT4_WRAP_CPP(CXX_FILES ${MOCS})
#endif()


if (NOT CMAKE_BUILD_TYPE)
    set ( CMAKE_BUILD_TYPE Release )
endif (NOT CMAKE_BUILD_TYPE)


set(APP_SHARE_DIR ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/lib${XDGQT_FILE_NAME})
add_definitions(-DTRANSLATIONS_DIR=\"${APP_SHARE_DIR}\")

add_library(${XDGQT_LIBRARY_NAME} SHARED
    ${PUBLIC_H_FILES}
    ${PRIVATE_H_FILES}
    ${CPP_FILES}
    ${CXX_FILES}
    ${QM_FILES}
)

target_link_libraries(${XDGQT_LIBRARY_NAME} ${QT_LIBRARIES} ${LIBMAGIC_LIBRARY})

set_target_properties(${XDGQT_LIBRARY_NAME} PROPERTIES
  VERSION ${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}
  SOVERSION ${MAJOR_VERSION}
)

#qt5_use_modules(${XDGQT_LIBRARY_NAME} Widgets Xml)

install(TARGETS ${XDGQT_LIBRARY_NAME} DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES ${PUBLIC_H_FILES} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${XDGQT_FILE_NAME})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${XDGQT_FILE_NAME}-config.cmake DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${XDGQT_FILE_NAME})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cmake/${XDGQT_FILE_NAME}_use.cmake DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/${XDGQT_FILE_NAME})
install(FILES ${PORTABLE_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${XDGQT_FILE_NAME})

include(create_pkgconfig_file)
create_pkgconfig_file(${XDGQT_LIBRARY_NAME}
    ${XDGQT_PKG_CONFIG_DESCRIPTION}
    ${XDGQT_PKG_CONFIG_REQUIRES}
    ${XDGQT_FILE_NAME}
    ${XDGQT_VERSION_STRING}
)


if(BUILD_TESTS STREQUAL "Yes")
    enable_testing()
    add_subdirectory(test)
else()
    message(STATUS "")
    message(STATUS "For building tests use -DBUILD_TESTS=Yes option.")
    message(STATUS "")
endif()
