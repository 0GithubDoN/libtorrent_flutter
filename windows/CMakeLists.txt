# Windows CMakeLists.txt — uses prebuilt lib if available, otherwise builds from source.
cmake_minimum_required(VERSION 3.14)

set(PROJECT_NAME "libtorrent_flutter")
project(${PROJECT_NAME} LANGUAGES CXX)

# Pick the prebuilt slice that matches the *target* architecture, not the
# build-host architecture. Flutter Windows desktop sets one of these:
#   - CMAKE_GENERATOR_PLATFORM = x64 / ARM64   (Visual Studio generator)
#   - CMAKE_VS_PLATFORM_NAME   = x64 / ARM64
# When neither is set (e.g. running plain `cmake` from the prompt), fall
# back to the host arch via PROCESSOR_ARCHITECTURE.
set(_LTF_TARGET_ARCH "")
if(DEFINED CMAKE_GENERATOR_PLATFORM AND CMAKE_GENERATOR_PLATFORM)
  set(_LTF_TARGET_ARCH "${CMAKE_GENERATOR_PLATFORM}")
elseif(DEFINED CMAKE_VS_PLATFORM_NAME AND CMAKE_VS_PLATFORM_NAME)
  set(_LTF_TARGET_ARCH "${CMAKE_VS_PLATFORM_NAME}")
else()
  set(_LTF_TARGET_ARCH "$ENV{PROCESSOR_ARCHITECTURE}")
endif()

string(TOLOWER "${_LTF_TARGET_ARCH}" _LTF_TARGET_ARCH_LOWER)
if(_LTF_TARGET_ARCH_LOWER MATCHES "arm64|aarch64")
  set(LTF_WINDOWS_ARCH "arm64")
else()
  set(LTF_WINDOWS_ARCH "x64")
endif()

set(PREBUILT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../prebuilt/windows")
set(PREBUILT_DLL "${PREBUILT_DIR}/${LTF_WINDOWS_ARCH}/libtorrent_flutter.dll")

# -----------------------------------------------------------------------------
# Prebuilt binaries are NOT shipped on pub.dev. Download from the matching
# GitHub Release on first build. Set -DLIBTORRENT_FLUTTER_SKIP_DOWNLOAD=ON to
# opt out (e.g. you've dropped your own .dll into prebuilt/windows/<arch>/).
# -----------------------------------------------------------------------------
option(LIBTORRENT_FLUTTER_SKIP_DOWNLOAD "Skip downloading prebuilt native lib" OFF)

if(NOT EXISTS "${PREBUILT_DLL}" AND NOT LIBTORRENT_FLUTTER_SKIP_DOWNLOAD)
  set(_PUBSPEC "${CMAKE_CURRENT_SOURCE_DIR}/../pubspec.yaml")
  if(EXISTS "${_PUBSPEC}")
    file(READ "${_PUBSPEC}" _PUBSPEC_CONTENT)
    if("${_PUBSPEC_CONTENT}" MATCHES "version:[ \t]*([^\r\n ]+)")
      set(_LTF_VERSION "${CMAKE_MATCH_1}")
      set(_DL_URL "https://github.com/ayman708-UX/libtorrent_flutter/releases/download/v${_LTF_VERSION}/windows-native-lib-${LTF_WINDOWS_ARCH}.zip")
      set(_ZIP "${CMAKE_CURRENT_BINARY_DIR}/windows-native-lib-${LTF_WINDOWS_ARCH}.zip")
      message(STATUS "libtorrent_flutter: downloading prebuilt from ${_DL_URL}")
      file(DOWNLOAD "${_DL_URL}" "${_ZIP}" SHOW_PROGRESS STATUS _DL_STATUS TLS_VERIFY ON)
      list(GET _DL_STATUS 0 _DL_CODE)
      if(_DL_CODE EQUAL 0)
        # CI artifact root corresponds to prebuilt/windows/, so the zip
        # already contains an <arch>/ subdir holding the DLLs.
        file(MAKE_DIRECTORY "${PREBUILT_DIR}")
        execute_process(
          COMMAND "${CMAKE_COMMAND}" -E tar xf "${_ZIP}"
          WORKING_DIRECTORY "${PREBUILT_DIR}"
          RESULT_VARIABLE _UNZIP_RC
        )
        file(REMOVE "${_ZIP}")
        if(NOT _UNZIP_RC EQUAL 0)
          message(WARNING "libtorrent_flutter: failed to extract prebuilt zip (rc=${_UNZIP_RC}); will build from source")
        endif()
      else()
        message(WARNING "libtorrent_flutter: prebuilt download failed (${_DL_STATUS}); will build from source")
      endif()
    endif()
  endif()
endif()

if(EXISTS "${PREBUILT_DLL}")
  message(STATUS "Using prebuilt Windows ${LTF_WINDOWS_ARCH} native library")
  # Bundle the main DLL plus every dependency DLL sitting next to it.
  file(GLOB PREBUILT_DEPS "${PREBUILT_DIR}/${LTF_WINDOWS_ARCH}/*.dll")
  set(libtorrent_flutter_bundled_libraries
    ${PREBUILT_DEPS}
    PARENT_SCOPE
  )
else()
  message(STATUS "Building from source (no prebuilt found at ${PREBUILT_DLL})")
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/../src" "${CMAKE_CURRENT_BINARY_DIR}/shared")
  set(libtorrent_flutter_bundled_libraries
    $<TARGET_FILE:libtorrent_flutter>
    PARENT_SCOPE
  )
endif()
# Windows CMakeLists.txt — uses prebuilt lib if available, otherwise builds from source.
cmake_minimum_required(VERSION 3.14)

set(PROJECT_NAME "libtorrent_flutter")
project(${PROJECT_NAME} LANGUAGES CXX)

set(PREBUILT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../prebuilt/windows")
set(PREBUILT_DLL "${PREBUILT_DIR}/x64/libtorrent_flutter.dll")

# -----------------------------------------------------------------------------
# Prebuilt binaries are NOT shipped on pub.dev. Download from the matching
# GitHub Release on first build. Set -DLIBTORRENT_FLUTTER_SKIP_DOWNLOAD=ON to
# opt out (e.g. you've dropped your own .dll into prebuilt/windows/x64/).
# -----------------------------------------------------------------------------
option(LIBTORRENT_FLUTTER_SKIP_DOWNLOAD "Skip downloading prebuilt native lib" OFF)

if(NOT EXISTS "${PREBUILT_DLL}" AND NOT LIBTORRENT_FLUTTER_SKIP_DOWNLOAD)
  set(_PUBSPEC "${CMAKE_CURRENT_SOURCE_DIR}/../pubspec.yaml")
  if(EXISTS "${_PUBSPEC}")
    file(READ "${_PUBSPEC}" _PUBSPEC_CONTENT)
    if("${_PUBSPEC_CONTENT}" MATCHES "version:[ \t]*([^\r\n ]+)")
      set(_LTF_VERSION "${CMAKE_MATCH_1}")
      set(_DL_URL "https://github.com/ayman708-UX/libtorrent_flutter/releases/download/v${_LTF_VERSION}/windows-native-lib.zip")
      set(_ZIP "${CMAKE_CURRENT_BINARY_DIR}/windows-native-lib.zip")
      message(STATUS "libtorrent_flutter: downloading prebuilt from ${_DL_URL}")
      file(DOWNLOAD "${_DL_URL}" "${_ZIP}" SHOW_PROGRESS STATUS _DL_STATUS TLS_VERIFY ON)
      list(GET _DL_STATUS 0 _DL_CODE)
      if(_DL_CODE EQUAL 0)
        # CI artifact root corresponds to prebuilt/windows/, so the zip
        # already contains an x64/ subdir holding the DLLs.
        file(MAKE_DIRECTORY "${PREBUILT_DIR}")
        execute_process(
          COMMAND "${CMAKE_COMMAND}" -E tar xf "${_ZIP}"
          WORKING_DIRECTORY "${PREBUILT_DIR}"
          RESULT_VARIABLE _UNZIP_RC
        )
        file(REMOVE "${_ZIP}")
        if(NOT _UNZIP_RC EQUAL 0)
          message(WARNING "libtorrent_flutter: failed to extract prebuilt zip (rc=${_UNZIP_RC}); will build from source")
        endif()
      else()
        message(WARNING "libtorrent_flutter: prebuilt download failed (${_DL_STATUS}); will build from source")
      endif()
    endif()
  endif()
endif()

if(EXISTS "${PREBUILT_DLL}")
  message(STATUS "Using prebuilt Windows native library")
  # Bundle the main DLL plus every dependency DLL sitting next to it.
  file(GLOB PREBUILT_DEPS "${PREBUILT_DIR}/x64/*.dll")
  set(libtorrent_flutter_bundled_libraries
    ${PREBUILT_DEPS}
    PARENT_SCOPE
  )
else()
  message(STATUS "Building from source (no prebuilt found at ${PREBUILT_DLL})")
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/../src" "${CMAKE_CURRENT_BINARY_DIR}/shared")
  set(libtorrent_flutter_bundled_libraries
    $<TARGET_FILE:libtorrent_flutter>
    PARENT_SCOPE
  )
endif()
# Windows CMakeLists.txt — uses prebuilt lib if available, otherwise builds from source.
cmake_minimum_required(VERSION 3.18)  # need file(ARCHIVE_EXTRACT)

set(PROJECT_NAME "libtorrent_flutter")
project(${PROJECT_NAME} LANGUAGES CXX)

set(PREBUILT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../prebuilt/windows")
set(PREBUILT_DLL "${PREBUILT_DIR}/x64/libtorrent_flutter.dll")

# ─────────────────────────────────────────────────────────────────────────────
# Prebuilt binaries are NOT shipped on pub.dev. Download from the matching
# GitHub Release on first build. Set -DLIBTORRENT_FLUTTER_SKIP_DOWNLOAD=ON to
# opt out (e.g. you've dropped your own .dll into prebuilt/windows/x64/).
# ─────────────────────────────────────────────────────────────────────────────
option(LIBTORRENT_FLUTTER_SKIP_DOWNLOAD "Skip downloading prebuilt native lib" OFF)

if(NOT EXISTS "${PREBUILT_DLL}" AND NOT LIBTORRENT_FLUTTER_SKIP_DOWNLOAD)
  set(_PUBSPEC "${CMAKE_CURRENT_SOURCE_DIR}/../pubspec.yaml")
  if(EXISTS "${_PUBSPEC}")
    file(READ "${_PUBSPEC}" _PUBSPEC_CONTENT)
    if("${_PUBSPEC_CONTENT}" MATCHES "version:[ \t]*([^\r\n ]+)")
      set(_LTF_VERSION "${CMAKE_MATCH_1}")
      set(_DL_URL "https://github.com/ayman708-UX/libtorrent_flutter/releases/download/v${_LTF_VERSION}/windows-native-lib.zip")
      set(_ZIP "${CMAKE_CURRENT_BINARY_DIR}/windows-native-lib.zip")
      message(STATUS "libtorrent_flutter: downloading prebuilt from ${_DL_URL}")
      file(DOWNLOAD "${_DL_URL}" "${_ZIP}" SHOW_PROGRESS STATUS _DL_STATUS TLS_VERIFY ON)
      list(GET _DL_STATUS 0 _DL_CODE)
      if(_DL_CODE EQUAL 0)
        file(MAKE_DIRECTORY "${PREBUILT_DIR}")
        # CI artifact root corresponds to prebuilt/windows/, so the zip
        # already contains an x64/ subdir holding the DLLs.
        file(ARCHIVE_EXTRACT INPUT "${_ZIP}" DESTINATION "${PREBUILT_DIR}")
        file(REMOVE "${_ZIP}")
      else()
        message(WARNING "libtorrent_flutter: prebuilt download failed (${_DL_STATUS}); will build from source")
      endif()
    endif()
  endif()
endif()

if(EXISTS "${PREBUILT_DLL}")
  message(STATUS "Using prebuilt Windows native library")
  # Bundle the main DLL plus every dependency DLL sitting next to it.
  file(GLOB PREBUILT_DEPS "${PREBUILT_DIR}/x64/*.dll")
  set(libtorrent_flutter_bundled_libraries
    ${PREBUILT_DEPS}
    PARENT_SCOPE
  )
else()
  message(STATUS "Building from source (no prebuilt found at ${PREBUILT_DLL})")
  add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/../src" "${CMAKE_CURRENT_BINARY_DIR}/shared")
  set(libtorrent_flutter_bundled_libraries
    $<TARGET_FILE:libtorrent_flutter>
    PARENT_SCOPE
  )
endif()
