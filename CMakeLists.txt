cmake_minimum_required(VERSION 3.12)

# Make a cursory attempt to find eSpeak NG sources.
# Allow for the following scenarios:
#   * The eSpeak NG repository is checked out into a child directory:
#     ```
#     espeak-ng-cmake
#     |-- CMakeLists.txt  (This file)
#     |-- espeak-ng  (eSpeak NG sources)
#         |-- src
#             |-- espeak-ng.c
#     ```
#   * The eSpeak NG repository is checked out into the same directory,
#     or equivalently this file is copied into the directory:
#     ```
#     |-- espeak-ng  (eSpeak NG sources)
#         |-- CMakeLists.txt  (This file)
#         |-- src
#             |-- espeak-ng.c
#     ```
#   * The eSpeak NG repository is checked out into a sibling directory:
#     ```
#     |-- espeak-ng-cmake
#     |   |-- CMakeLists.txt  (This file)
#     |-- espeak-ng  (eSpeak NG sources)
#         |-- src
#             |-- espeak-ng.c
#     ```
find_path(
  espeak-ng_ROOT
  NAMES
    "src/espeak-ng.c"
    "src/include/espeak-ng/espeak_ng.h"
    "src/libespeak-ng/speech.c"
  PATHS "." ".."
  PATH_SUFFIXES "espeak-ng"
  DOC "eSpeak NG repository root "
  NO_DEFAULT_PATH
)

# Installation will try use a standard file directory structure.
# The included module provides functions for that.
include(GNUInstallDirs)
# Installation will create packages.
# The included module makes it easier to create them.
include(CMakePackageConfigHelpers)

# Version number has a `-dev` suffix, but CMake does not support it.
project(espeak-ng VERSION "1.51")
set(espeak-ng_PACKAGE_VERSION "${PROJECT_VERSION}-dev")

# Original Autoconf provided some options.
# They are replicated here under different names.
# They do not seem to be available in Windows though...?
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  set(${PROJECT_NAME}_WITH_KLATT TRUE)
  set(${PROJECT_NAME}_WITH_MBROLA TRUE)
  set(${PROJECT_NAME}_WITH_SONIC FALSE) # Not supported in Windows.
  set(${PROJECT_NAME}_WITH_ASYNC FALSE) # Not supported in Windows.
  set(${PROJECT_NAME}_WITH_PCAUDIOLIB TRUE)
  # The extra language are in the Windows installer.
  # So they are added here,
  # even though they are disabled by default in Autoconf.
  set(${PROJECT_NAME}_WITH_EXTDICT_RU TRUE)
  set(${PROJECT_NAME}_WITH_EXTDICT_ZH TRUE)
  set(${PROJECT_NAME}_WITH_EXTDICT_ZHY TRUE)
else()
  set(
    ${PROJECT_NAME}_WITH_KLATT
    TRUE
    CACHE BOOL "Enable Klatt formant synthesis."
  )
  set(
    ${PROJECT_NAME}_WITH_MBROLA
    TRUE
    CACHE BOOL "Enable MBROLA voice support."
  )
  set(
    ${PROJECT_NAME}_WITH_SONIC
    TRUE
    CACHE BOOL "Use the sonic library to support higher WPM."
  )
  set(
    ${PROJECT_NAME}_WITH_ASYNC
    TRUE
    CACHE BOOL "Enable asynchronous commands."
  )
  set(
    ${PROJECT_NAME}_WITH_PCAUDIOLIB
    TRUE
    CACHE BOOL "Enable audio output support via pcaudiolib."
  )
  set(
    ${PROJECT_NAME}_WITH_EXTDICT_RU
    FALSE
    CACHE BOOL "Build Russian extended dictionary."
  )
  set(
    ${PROJECT_NAME}_WITH_EXTDICT_ZH
    FALSE
    CACHE BOOL "Build Mandarin Chinese extended dictionary."
  )
  set(
    ${PROJECT_NAME}_WITH_EXTDICT_ZHY
    FALSE
    CACHE BOOL "Build Cantonese extended dictionary."
  )
endif()

#
# ### CMake compatibility.
#
# Autoconf creates a `config.h`, and source code expects to find it.
# The header is not necessary with CMake,
# though not all functionalities are ironed out yet.
set(FAKE_CONFIG_H_DIR "${CMAKE_CURRENT_BINARY_DIR}/fake_include")
set(FAKE_CONFIG_H "${FAKE_CONFIG_H_DIR}/config.h")
add_custom_command(
  OUTPUT "${FAKE_CONFIG_H}"
  COMMAND
    "${CMAKE_COMMAND}" ARGS "-E" "make_directory" "${FAKE_CONFIG_H_DIR}"
  COMMAND "${CMAKE_COMMAND}" ARGS "-E" "touch" "${FAKE_CONFIG_H}"
)
add_custom_target(create_fake_config_h DEPENDS "${FAKE_CONFIG_H}")

#
#
#
# ## The eSpeak NG library

set(TARGET_NAME ${PROJECT_NAME})
add_library(
  ${TARGET_NAME} SHARED
  "${espeak-ng_ROOT}/src/ucd-tools/src/case.c"
  "${espeak-ng_ROOT}/src/ucd-tools/src/categories.c"
  "${espeak-ng_ROOT}/src/ucd-tools/src/ctype.c"
  "${espeak-ng_ROOT}/src/ucd-tools/src/proplist.c"
  "${espeak-ng_ROOT}/src/ucd-tools/src/scripts.c"
  "${espeak-ng_ROOT}/src/ucd-tools/src/tostring.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/compiledata.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/compiledict.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/compilembrola.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/dictionary.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/encoding.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/error.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/espeak_api.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/ieee80.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/intonation.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/mnemonics.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/numbers.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/readclause.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/phoneme.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/phonemelist.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/setlengths.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/spect.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/speech.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/ssml.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/synthdata.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/synthesize.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/synth_mbrola.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/translate.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/tr_languages.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/voices.c"
  "${espeak-ng_ROOT}/src/libespeak-ng/wavegen.c"
)

# For most other operating systems, `lib` is prepended to the output.
# And eSpeak NG distributes a MSVC library binary
# that is called `libespeak-ng.dll`.
# However, in Windows,
# the default library name from CMake is `epseak-ng.dll`.
# The output name is changed here to match it.
# Though... the API is likely incompatible.
# So perhaps it is better not to do this?
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set_target_properties(
    ${TARGET_NAME} PROPERTIES OUTPUT_NAME "libespeak-ng"
  )
endif()

# Tell CMake that the library built should be installed when asked to.
install(TARGETS ${TARGET_NAME} EXPORT ${TARGET_NAME})

# Let other CMake users find this library after it is installed.
install(
  EXPORT ${TARGET_NAME}
  DESTINATION "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake"
  NAMESPACE "${PROJECT_NAME}::"
  FILE "${TARGET_NAME}-target.cmake"
)

#
# ### Headers

# For `config.h`.
add_dependencies(${TARGET_NAME} create_fake_config_h)
target_include_directories(${TARGET_NAME} PRIVATE "${FAKE_CONFIG_H_DIR}")

# Tell compiler where to find headers.
target_include_directories(
  ${TARGET_NAME}
  PUBLIC
    # This tells compiler where to find the library headers
    # when compiling the library.
    $<BUILD_INTERFACE:${espeak-ng_ROOT}/src/include>
    # When the library is installed,
    # a CMake module is also installed
    # to tell other CMake users where to find this library.
    # This tells other users of the CMake module how to find the headers.
    #
    # This needs to be specified because the in-source hierarchy
    # is incompatible with the hierarchy for installation
    #
    # The actual installation of those headers are below.
    $<INSTALL_INTERFACE:include>
  PRIVATE "${espeak-ng_ROOT}/src/include/compat"
  PRIVATE "${espeak-ng_ROOT}/src/ucd-tools/src/include"
)
# Install headers of this library along with the library.
install(
  DIRECTORY "${espeak-ng_ROOT}/src/include/espeak-ng" TYPE "INCLUDE"
)

#
# ### Macros and compiler options

set(espeak-ng_DATA "espeak-ng-data")
target_compile_definitions(
  ${TARGET_NAME}
  PRIVATE "PACKAGE_VERSION=\"${espeak-ng_PACKAGE_VERSION}\""
)

# These are found in Automake but not in Visual Studio projects.
if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_definitions(
    ${TARGET_NAME}
    PRIVATE
      "_BSD_SOURCE"
      "_DEFAULT_SOURCE"
      "_POSIX_C_SOURCE=200112L"
      "PATH_ESPEAK_DATA=\"${espeak-ng_DATA}\""
  )
endif()

## Something to do with versioning in filename extension.
## Unsure how to make this work...
#if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
#  set(espeak-ng_SHARED_VERSION "2:49:1")
#  target_link_options(${TARGET_NAME}
#    PRIVATE "-version-info ${espeak-ng_SHARED_VERSION}"
#  )
#endif()

if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_options(
    ${TARGET_NAME}
    PRIVATE "-fPIC" "-fno-exceptions" "-fvisibility=hidden"
  )
endif()

#
# ### Link dependencies

find_package(Threads)
target_link_libraries(${TARGET_NAME} Threads::Threads)

## Autoconf also searches for libm.
## Assume the standard library used will not require it.
#target_link_libraries(${TARGET_NAME} m)

#
# ### Extra features based on user configuration

if("${${PROJECT_NAME}_WITH_KLATT}")
  target_sources(
    ${TARGET_NAME} PRIVATE "${espeak-ng_ROOT}/src/libespeak-ng/klatt.c"
  )
  target_compile_definitions(${TARGET_NAME} PRIVATE "INCLUDE_KLATT")
endif()

if("${${PROJECT_NAME}_WITH_MBROLA}")
  target_sources(
    ${TARGET_NAME}
    PRIVATE "${espeak-ng_ROOT}/src/libespeak-ng/mbrowrap.c"
  )
  target_compile_definitions(${TARGET_NAME} PRIVATE "INCLUDE_MBROLA")
endif()

if("${${PROJECT_NAME}_WITH_ASYNC}")
  target_sources(
    ${TARGET_NAME}
    PRIVATE
      "${espeak-ng_ROOT}/src/libespeak-ng/espeak_command.c"
      "${espeak-ng_ROOT}/src/libespeak-ng/event.c"
      "${espeak-ng_ROOT}/src/libespeak-ng/fifo.c"
  )
  target_compile_definitions(${TARGET_NAME} PRIVATE "USE_ASYNC")
endif()

if("${${PROJECT_NAME}_WITH_PCAUDIOLIB}")
  target_compile_definitions(
    ${TARGET_NAME} PUBLIC "HAVE_PCAUDIOLIB_AUDIO_H"
  )

  if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
    find_package(pcaudiolib REQUIRED)
    target_link_libraries(${TARGET_NAME} pcaudiolib::pcaudiolib)
  else()
    # Assume the library can be found on other systems.
    target_link_libraries(${TARGET_NAME} pcaudio)
  endif()
endif()

#
# ### Windows compatibility.

target_compile_definitions(${TARGET_NAME} PUBLIC "LIBESPEAK_NG_EXPORT")

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_sources(
    ${TARGET_NAME}
    PRIVATE
      # Not sure what these are for.
      "${espeak-ng_ROOT}/src/windows/com/comentrypoints.c"
      "${espeak-ng_ROOT}/src/windows/com/ttsengine.cpp"
  )
endif()

#
# ### Enable warnings and silence known warnings

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_definitions(
    ${TARGET_NAME}
    PRIVATE
      # Disable warnings.
      "_CRT_SECURE_NO_WARNINGS"
      "_CRT_NONSTDC_NO_DEPRECATE"
  )
else()
  target_compile_options(
    ${TARGET_NAME}
    PRIVATE
      # Disable warnings.
      "-Wno-endif-labels"
      # Enable warnings.
      "-pedantic"
      "-Wimplicit"
      "-Wmissing-prototypes"
      "-Wreturn-type"
      "-Wuninitialized"
      "-Wunused"
      "-Wunused-parameter"
  )
endif()

#
#
#
# ## Executable CLI binary.

set(TARGET_NAME "${PROJECT_NAME}-executable")
add_executable(${TARGET_NAME} "${espeak-ng_ROOT}/src/espeak-ng.c")
# The default output name is based on the target name.
# So it needs to be changed to `espeak-ng`.
set_target_properties(${TARGET_NAME} PROPERTIES OUTPUT_NAME "espeak-ng")
install(TARGETS ${TARGET_NAME})

#
# ### Headers

# For `config.h`.
add_dependencies(${TARGET_NAME} create_fake_config_h)
target_include_directories(${TARGET_NAME} PRIVATE "${FAKE_CONFIG_H_DIR}")

# Found in Automake, but not sure if this is actually used.
if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_include_directories(
    ${TARGET_NAME} PRIVATE "${espeak-ng_ROOT}/src/libespeak-ng"
  )
endif()

target_include_directories(
  ${TARGET_NAME}
  PRIVATE "${espeak-ng_ROOT}/src/include/compat"
          "${espeak-ng_ROOT}/src/ucd-tools/src/include"
)

#
# ### Macros and compiler options

target_compile_definitions(
  ${TARGET_NAME}
  PRIVATE "PACKAGE_VERSION=\"${espeak-ng_PACKAGE_VERSION}\""
)

# These are found in Automake but not in Visual Studio projects.
if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_definitions(
    ${TARGET_NAME}
    PRIVATE "_BSD_SOURCE" "_DEFAULT_SOURCE" "_POSIX_C_SOURCE=200112L"
  )
endif()

#
# ### Link dependencies

target_link_libraries(${TARGET_NAME} espeak-ng)

#
# Specified in Automake.
if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_link_options(${TARGET_NAME} PRIVATE "-static")
endif()

#
# ### Enable warnings and silence known warnings

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_definitions(
    ${TARGET_NAME}
    PRIVATE
      # Disable warnings.
      "_CRT_SECURE_NO_WARNINGS"
      "_CRT_NONSTDC_NO_DEPRECATE"
  )
else()
  # Disable warnings.
  target_compile_options(${TARGET_NAME} PRIVATE "-Wno-endif-labels")
endif()

#
# ### Windows compatibility.

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_sources(
    ${TARGET_NAME} PRIVATE "${espeak-ng_ROOT}/src/compat/getopt.c"
  )
endif()

#
#
#
# ## Generate data files.

set(TARGET_NAME "${PROJECT_NAME}-data")
add_custom_target(${TARGET_NAME} ALL)

# Data generation is hard-coded by eSpeak NG, relative input source data.
# So input is copied first into build directory
# to not pollute source directory, before generating data.
set(${TARGET_NAME}_OUTPUT_DIR "${PROJECT_BINARY_DIR}/data_output")
foreach(DIRECTORY_TO_COPY dictsource;espeak-ng-data;phsource)
  add_custom_command(
    TARGET ${TARGET_NAME}
    PRE_BUILD
    COMMAND
      "${CMAKE_COMMAND}" ARGS "-E" "copy_directory"
      "${espeak-ng_ROOT}/${DIRECTORY_TO_COPY}"
      "${${TARGET_NAME}_OUTPUT_DIR}/${DIRECTORY_TO_COPY}"
  )
endforeach()

# CMake code to execute on install.
# This is the beginning part of CMake code
# that can execute the eSpeak NG binary.
string(
  JOIN
  " "
  ESPEAK_NG_EXECUTE_PROCESS
  "execute_process(COMMAND"
  "\"${CMAKE_INSTALL_FULL_BINDIR}/espeak-ng\""
  "\"--path=${${TARGET_NAME}_OUTPUT_DIR}/espeak-ng-data\""
  # Requires ")" to be appended later before using this.
)

# The actual commands to compile data.
# Since the eSpeak NG binary is used to compile the data,
# the compilation is done in install time
# because shared libraries are guaranteed to be found after installing
# (or at least they should be in a normal setup).
# If the compilation is done at build time,
# then it is possible that the binary cannot find required libraries.
# This is particularly apparent in Windows without `$RPATH`.
install(CODE "${ESPEAK_NG_EXECUTE_PROCESS} \"--compile-intonation\")")
install(CODE "${ESPEAK_NG_EXECUTE_PROCESS} \"--compile-phonemes\")")

list(
  APPEND
  LANGUAGE_COMPILE_LIST
  af;am;an;as;az;bg;bn;bs;ca;cs;cy;da;de;el;en;eo;es;et;eu;fa;fi;fr;ga;gd
  gn;grc;gu;hi;hr;hu;hy;ia;id;is;it;jbo;ja;ka;kl;kn;ko;ku;ky;la;lfn;lt;lv
  mk;ml;mr;ms;mt;my;nci;ne;nl;no;om;or;pa;pap;pl;pt;ro;ru;si;sk;sl;sq;sr
  sv;sw;ta;te;tn;tr;tt;ur;vi;zh;yue
)

foreach(LANG_TO_COMPILE ${LANGUAGE_COMPILE_LIST})
  string(
    JOIN
    " "
    ESPEAK_NG_INSTALL_CODE
    "${ESPEAK_NG_EXECUTE_PROCESS}"
    "\"--compile=${LANG_TO_COMPILE}\""
    "\"WORKING_DIRECTORY\""
    "\"${${TARGET_NAME}_OUTPUT_DIR}/dictsource\")"
  )
  install(CODE "${ESPEAK_NG_INSTALL_CODE}")
endforeach()

#
# ### Figure out which files to install.
#
# The Windows install seem to install a different set of files
# when compared to the files from Debian.
# The file hierachy is a little different.
# Special casing is required?

# The `yue` language name becomes `zhy` in the output.
set(LANGUAGE_INSTALL_LIST ${LANGUAGE_COMPILE_LIST})
list(REMOVE_ITEM LANGUAGE_INSTALL_LIST yue)
list(APPEND LANGUAGE_INSTALL_LIST zhy)
# The actual language file to instead has "_dict" at the end.
list(TRANSFORM LANGUAGE_INSTALL_LIST APPEND "_dict")
# There are some extra files to install.
list(
  APPEND
  LANGUAGE_INSTALL_LIST
  "intonations"
  "phondata"
  "phonindex"
  "phontab"
)
# Turn the filenames into a full path.
list(
  TRANSFORM LANGUAGE_INSTALL_LIST
  PREPEND "${${TARGET_NAME}_OUTPUT_DIR}/${espeak-ng_DATA}/"
)

install(
  FILES ${LANGUAGE_INSTALL_LIST}
  DESTINATION "${CMAKE_INSTALL_DATADIR}/${espeak-ng_DATA}"
)

# Not sure why the source and install directory name are different.
# But that is what their names are.
install(
  DIRECTORY "${${TARGET_NAME}_OUTPUT_DIR}/${espeak-ng_DATA}/lang/"
  DESTINATION "${CMAKE_INSTALL_DATADIR}/${espeak-ng_DATA}/voices"
)

#
# ### Data that depends on options.

# MBROLA build doe not seem to be built at all on Windows.
if("${${PROJECT_NAME}_WITH_MBROLA}")
  if(NOT "${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
    file(
      MAKE_DIRECTORY
      # Make sure MBROLA compile output directory exists.
      "${${TARGET_NAME}_OUTPUT_DIR}/espeak-ng-data/mbrola_ph"
    )

    list(
      APPEND
      MBROLA_COMPILE_LIST
      af1;ar1;ar2;ca;cr1;cs;de2;de4;de6;de8;ee1;en1;es;es3;es4;fr;gr1;gr2
      grc-de6;hn1;hu1;ic1;id1;in;ir1;it1;it3;jp;la1;lt;ma1;mx1;mx2;nl;nz1
      pl1;pt1;ptbr;ptbr4;ro1;sv;sv2;tl1;tr1;us;us3;vz;zh
    )
    foreach(LANG_TO_COMPILE ${MBROLA_COMPILE_LIST})
      string(
        JOIN
        " "
        ESPEAK_NG_INSTALL_CODE
        "${ESPEAK_NG_EXECUTE_PROCESS}"
        "\"--compile-mbrola=phsource/mbrola/${LANG_TO_COMPILE}\""
        "\"WORKING_DIRECTORY\""
        "\"${${TARGET_NAME}_OUTPUT_DIR}\")"
      )
      install(CODE "${ESPEAK_NG_INSTALL_CODE}")
    endforeach()

    # Determine which files to install.
    install(
      DIRECTORY
        "${${TARGET_NAME}_OUTPUT_DIR}/${espeak-ng_DATA}/mbrola_ph"
      DESTINATION "${CMAKE_INSTALL_DATADIR}/${espeak-ng_DATA}"
    )
  endif()
endif()

#
#
#
# ## Create a CMake package that imports all targets in this project.

# Generate the template needed to for creating the package file.
# This is done at configuration time.
set(TEMPLATE_DIR "${CMAKE_CURRENT_BINARY_DIR}/cmake_modules")
set(CONFIG_TEMPLATE "${TEMPLATE_DIR}/package_config_template.cmake.in")
set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/cmake")
# Add targets to install into this.
# Every target in this list must include `${TARGET_NAME}-target.cmake`.
set(INSTALLED_BUILDSYSTEM_TARGETS "${PROJECT_NAME}")
file(
  WRITE "${CONFIG_TEMPLATE}"
  [=[
@PACKAGE_INIT@

include(CMakeFindDependencyMacro)

foreach(TARGET_TO_INCLUDE @INSTALLED_BUILDSYSTEM_TARGETS@)
  include("${CMAKE_CURRENT_LIST_DIR}/${TARGET_TO_INCLUDE}-target.cmake")
endforeach()

if(@espeak-ng_WITH_PCAUDIOLIB@)
  find_dependency(pcaudiolib)
endif()
find_dependency(Threads)

check_required_components(@INSTALLED_BUILDSYSTEM_TARGETS@)
]=]
)

# Tell CMake to generate the package.
set(CONFIG_FILENAME "${PROJECT_NAME}-config.cmake")
configure_package_config_file(
  # Tell CMake to use the template generated above.
  "${CONFIG_TEMPLATE}"
  # The file to contain the generated package file at build time.
  "${TEMPLATE_DIR}/${CONFIG_FILENAME}"
  # Directory to copy the package to at install time.
  # Note that it does not actually install the file created.
  INSTALL_DESTINATION
  "${CONFIG_INSTALL_DIR}"
)
# So install the file manually.
install(
  FILES "${TEMPLATE_DIR}/${CONFIG_FILENAME}"
  DESTINATION "${CONFIG_INSTALL_DIR}"
)
