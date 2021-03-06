project(KGtk)
cmake_minimum_required(VERSION 2.8.12)

set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README")
set(CPACK_GENERATOR "TBZ2")
set(CPACK_SOURCE_GENERATOR "TBZ2")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "11")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(KGTK_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}")
set(KGTK_VERSION_FULL "${KGTK_VERSION}.${CPACK_PACKAGE_VERSION_PATCH}")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${KGTK_VERSION_FULL}")
include(CPack)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

find_package(ECM 1.3.0 REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR})
include(ECMSetupVersion)
include(FeatureSummary)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)

find_package(Qt5 REQUIRED Core Widgets X11Extras)
find_package(KF5 REQUIRED
    KIO
    I18n
    WindowSystem
    DBusAddons
    JobWidgets
    )

include(CheckFunctionExists)
check_function_exists(getpeereid HAVE_GETPEEREID)

find_library(LIBDLVSYM_LIBRARY
                NAMES dl
                PATHS /lib /usr/lib /usr/X11R6/lib /usr/local/lib)

if(LIBDLVSYM_LIBRARY)
    set(HAVE_DLVSYM 1)
    # Determine version of dlsym...
    find_library(LIBDL_LIBRARY
                    NAMES dl
                    PATHS /lib /usr/lib /usr/X11R6/lib /usr/local/lib)

    if(LIBDL_LIBRARY)
        execute_process(COMMAND objdump --dynamic-syms ${LIBDL_LIBRARY}
                        COMMAND grep dlsym
                        OUTPUT_VARIABLE DLSYM_DATA)
        separate_arguments(DLSYM_DATA)
        if (${CMAKE_MAJOR_VERSION} GREATER 2 OR ${CMAKE_MINOR_VERSION} GREATER 4)
            cmake_policy(SET CMP0007 OLD)
        endif (${CMAKE_MAJOR_VERSION} GREATER 2 OR ${CMAKE_MINOR_VERSION} GREATER 4)
        list(GET DLSYM_DATA 4 KGTK_DLSYM_VERSION)
    endif(LIBDL_LIBRARY)
else(LIBDLVSYM_LIBRARY)
    message("** INFORMATION: Your libdl does not contain dlvsym - SWT apps will not be supported")
endif(LIBDLVSYM_LIBRARY)

add_subdirectory(gtk2)
add_subdirectory(gtk3)
add_subdirectory(kdialogd5)

message("** INFORMATION: Using installation prefix: ${CMAKE_INSTALL_PREFIX}")
configure_file (config.h.cmake ${CMAKE_BINARY_DIR}/config.h)
configure_file (kgtk-wrapper.cmake ${CMAKE_CURRENT_BINARY_DIR}/kgtk-wrapper @ONLY)

install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/kgtk-wrapper DESTINATION bin)

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
