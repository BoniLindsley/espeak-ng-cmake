cmake_minimum_required(VERSION 3.12)

# Write down where the source code of eSpeak NG is.
# This assumes that it is next to this repository,
# both inside some source directory.

# ```
# src
# |-- espeak-ng
# |-- espeak-ng-cmake
# ```
#
# This is needed to find source files.
# Change this if this CMake source file is moved.
get_filename_component(
  PROJECT_ROOT_PARENT "${CMAKE_SOURCE_DIR}" DIRECTORY
)
set(espeak-ng_ROOT "${PROJECT_ROOT_PARENT}/espeak-ng")

# Installation will try use a standard file directory structure.
# The included module provides functions for that.
include(GNUInstallDirs)

# Version number has a `-dev` suffix, but CMake does not support it.
project(espeak-ng VERSION "1.51")

# Original Autoconf provided some options.
# They are replicated here under different names.
# They do not seem to be available in Windows though...?
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set(${PROJECT_NAME}_WITH_KLATT TRUE)
  set(${PROJECT_NAME}_WITH_MBROLA TRUE)
  set(${PROJECT_NAME}_WITH_SONIC FALSE) # Not supported in Windows.
  set(${PROJECT_NAME}_WITH_ASYNC FALSE) # Not supported in Windows.
  set(${PROJECT_NAME}_WITH_PCAUDIOLIB TRUE) # Might break if false.
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
  DESTINATION "${CMAKE_INSTALL_DATADIR}/${TARGET_NAME}/cmake"
  NAMESPACE "${TARGET_NAME}::"
  FILE "${TARGET_NAME}-config.cmake"
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
set(espeak-ng_PACKAGE_VERSION "${PROJECT_VERSION}-dev")
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

if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  target_compile_options(
    ${TARGET_NAME}
    PRIVATE "-fPIC" "-fno-exceptions" "-fvisibility=hidden"
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
  target_compile_options(
    ${TARGET_NAME}
    PRIVATE
      # Disable warnings.
      "-Wno-endif-labels"
      # Enable warnings.
      "-pedantic"
  )
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

# Data generation is hard-coded.
# So input is copied first into build directory
# to not pollute source directory, before generating data.
add_custom_target(${TARGET_NAME}-make-directory ALL)
add_dependencies(${TARGET_NAME} ${TARGET_NAME}-make-directory)
set(${TARGET_NAME}_OUTPUT_DIR "${PROJECT_BINARY_DIR}/data_output")
foreach(DIRECTORY_TO_COPY dictsource;espeak-ng-data;phsource)
  add_custom_command(
    TARGET ${TARGET_NAME}-make-directory
    PRE_BUILD
    COMMAND
      "${CMAKE_COMMAND}" ARGS "-E" "copy_directory"
      "${espeak-ng_ROOT}/${DIRECTORY_TO_COPY}"
      "${${TARGET_NAME}_OUTPUT_DIR}/${DIRECTORY_TO_COPY}"
  )
endforeach()

# The actual commands to compile data.
add_custom_command(
  TARGET ${TARGET_NAME}
  PRE_BUILD
  # Input hard-coded as `<path_flag>/../phsource/intonations`
  # Output hard-coded as `<path_flag>/intonations`.
  COMMAND
    espeak-ng-executable ARGS
    # eSpeak NG wants path first.
    "--path=\"${${TARGET_NAME}_OUTPUT_DIR}/espeak-ng-data\""
    "--compile-intonation"
)

add_custom_command(
  TARGET ${TARGET_NAME}
  PRE_BUILD
  COMMAND
    espeak-ng-executable ARGS
    # eSpeak NG wants path first.
    "--path=\"${${TARGET_NAME}_OUTPUT_DIR}/espeak-ng-data\""
    "--compile-phonemes"
)

list(
  APPEND
  LANGUAGE_COMPILE_LIST
  af;am;an;as;az;bg;bn;bs;ca;cs;cy;da;de;el;en;eo;es;et;eu;fa;fi;fr;ga;gd
  gn;grc;gu;hi;hr;hu;hy;ia;id;is;it;jbo;ja;ka;kl;kn;ko;ku;ky;la;lfn;lt;lv
  mk;ml;mr;ms;mt;my;nci;ne;nl;no;om;or;pa;pap;pl;pt;ro;ru;si;sk;sl;sq;sr
  sv;sw;ta;te;tn;tr;tt;ur;vi;zh;yue
)

foreach(LANG_TO_COMPILE ${LANGUAGE_COMPILE_LIST})
  add_custom_command(
    TARGET ${TARGET_NAME}
    POST_BUILD
    COMMAND
      espeak-ng-executable ARGS
      # eSpeak NG wants path first.
      "--path=\"${${TARGET_NAME}_OUTPUT_DIR}/espeak-ng-data\""
      "--compile=${LANG_TO_COMPILE}"
    WORKING_DIRECTORY "${espeak-ng_ROOT}/dictsource"
  )
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
