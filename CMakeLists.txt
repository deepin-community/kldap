cmake_minimum_required(VERSION 3.16 FATAL_ERROR)
set(PIM_VERSION "5.18.1")

project(KLdap VERSION ${PIM_VERSION})

# ECM setup
set(KF5_MIN_VERSION "5.83.0")

find_package(ECM ${KF5_MIN_VERSION} CONFIG REQUIRED)
set(CMAKE_MODULE_PATH ${KLdap_SOURCE_DIR}/cmake ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDEFrameworkCompilerSettings NO_POLICY_SCOPE)

include(GenerateExportHeader)
include(ECMGenerateHeaders)
include(ECMGeneratePriFile)

include(ECMSetupVersion)
include(FeatureSummary)
include(KDEGitCommitHooks)
include(ECMQtDeclareLoggingCategory)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(KLDAP_LIB_VERSION ${PIM_VERSION})

find_package(KF5KIO ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5I18n ${KF5_MIN_VERSION} CONFIG REQUIRED)
find_package(KF5DocTools ${KF5_MIN_VERSION} CONFIG)
find_package(KF5KIO ${KF5_MIN_VERSION} CONFIG REQUIRED)

find_package(Qt5Keychain CONFIG)
set_package_properties(Qt5Keychain PROPERTIES
                                   DESCRIPTION "Provides support for secure credentials storage"
                                   URL "https://github.com/frankosterfeld/qtkeychain"
				   TYPE REQUIRED)

# tell what is missing without doctools
set_package_properties(KF5DocTools PROPERTIES DESCRIPTION "Provides tools to generate documentation in various format from DocBook files"
                       TYPE OPTIONAL
                       PURPOSE "Required to build documentation")

ecm_setup_version(PROJECT VARIABLE_PREFIX KLDAP
                        VERSION_HEADER "${CMAKE_CURRENT_BINARY_DIR}/kldap_version.h"
                        PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/KF5LdapConfigVersion.cmake"
                        SOVERSION 5
)
option(USE_UNITY_CMAKE_SUPPORT "Use UNITY cmake support (speedup compile time)" OFF)

set(COMPILE_WITH_UNITY_CMAKE_SUPPORT OFF)
if (USE_UNITY_CMAKE_SUPPORT)
    set(COMPILE_WITH_UNITY_CMAKE_SUPPORT ON)
endif()

########### Find packages ###########
find_package(KF5WidgetsAddons ${KF5_MIN_VERSION} CONFIG REQUIRED)

add_definitions(-DQT_DISABLE_DEPRECATED_BEFORE=0x050f00)
add_definitions(-DKF_DISABLE_DEPRECATED_BEFORE_AND_AT=0x055400)

find_package(Ldap)
set_package_properties(Ldap PROPERTIES
  TYPE RECOMMENDED
  PURPOSE "Needed to provide LDAP functionality in KDE"
)

find_package(Sasl2)
set_package_properties(Sasl2 PROPERTIES TYPE REQUIRED)

if (Ldap_FOUND)
   set(LDAP_FOUND 1)
endif()

add_definitions(-DTRANSLATION_DOMAIN=\"libkldap5\")


########### CMake Config Files ###########
set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/KF5Ldap")

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/KF5LdapConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/KF5LdapConfig.cmake"
  INSTALL_DESTINATION  ${CMAKECONFIG_INSTALL_DIR}
)

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/KF5LdapConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/KF5LdapConfigVersion.cmake"
  DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
  COMPONENT Devel
)

install(EXPORT KF5LdapTargets DESTINATION "${CMAKECONFIG_INSTALL_DIR}" FILE KF5LdapTargets.cmake NAMESPACE KF5::)

install(FILES
   ${CMAKE_CURRENT_BINARY_DIR}/kldap_version.h
   DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF5}
   COMPONENT Devel
)

########### Targets ###########
add_subdirectory(src)
add_subdirectory(kioslave)
if(BUILD_TESTING)
    add_subdirectory(autotests)
    add_subdirectory(tests)
endif()

ecm_qt_install_logging_categories(
        EXPORT KLDAP
        FILE kldap.categories
        DESTINATION ${KDE_INSTALL_LOGGINGCATEGORIESDIR}
        )

kde_configure_git_pre_commit_hook(CHECKS CLANG_FORMAT)
ki18n_install(po)
if(KF5DocTools_FOUND)
    kdoctools_install(po)
endif()
feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
