#
# leptonica
#

################################################################################

cmake_minimum_required(VERSION 3.1.3)
cmake_policy(SET CMP0054 NEW)

project(
  ${LEPTONICA_LIBRARY}
  LANGUAGES C
  VERSION 1.84.0)
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)

find_package(PkgConfig QUIET)



#find_package(GIF 5)
#if(GIF_FOUND)
#  set(libs_private "${libs_private} -lgif")
#else()
#  message(FATAL_ERROR "Could not find GIF 5 libs.\n Use -DENABLE_GIF=OFF to disable GIF support." )
#endif(GIF_FOUND)

test_big_endian(BIG_ENDIAN)

if(BIG_ENDIAN)
  set(ENDIANNESS L_BIG_ENDIAN)
else()
  set(ENDIANNESS L_LITTLE_ENDIAN)
endif()

set(APPLE_UNIVERSAL_BUILD "defined (__APPLE_CC__)")
configure_file(
    ${PROJECT_SOURCE_DIR}/endianness.h.in
    ${PROJECT_SOURCE_DIR}/endianness.h
    #${PROJECT_BINARY_DIR}/endianness.h
    @ONLY)

ocv_include_directories("${CMAKE_CURRENT_SOURCE_DIR}")

file(GLOB hdr "*.h")
file(GLOB src "*.c")

string(REPLACE "-O3" "-O2" CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")

add_library                     (${LEPTONICA_LIBRARY} ${src} ${hdr})


set_target_properties(${LEPTONICA_LIBRARY} PROPERTIES
    VERSION 6.0.0
    SOVERSION 6
    OUTPUT_NAME ${LEPTONICA_LIBRARY}
    DEBUG_POSTFIX "${OPENCV_DEBUG_POSTFIX}"
    COMPILE_PDB_NAME ${LEPTONICA_LIBRARY}
    COMPILE_PDB_NAME_DEBUG "${LEPTONICA_LIBRARY}${OPENCV_DEBUG_POSTFIX}"
    ARCHIVE_OUTPUT_DIRECTORY ${3P_LIBRARY_OUTPUT_PATH}
)

if(ENABLE_SOLUTION_FOLDERS)
  set_target_properties(${LEPTONICA_LIBRARY} PROPERTIES FOLDER "3rdparty")
endif()

if (BUILD_SHARED_LIBS)
    target_compile_definitions  (${LEPTONICA_LIBRARY} PRIVATE -DLIBLEPT_EXPORTS)
endif()

if (GIF_LIBRARIES)
    target_include_directories  (${LEPTONICA_LIBRARY} PRIVATE ${GIF_INCLUDE_DIR})
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE ${GIF_LIBRARIES})
endif()
if (JPEG_LIBRARIES)
    target_include_directories  (${LEPTONICA_LIBRARY} PRIVATE ${JPEG_INCLUDE_DIR})
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE ${JPEG_LIBRARIES})
endif()
if (OPENJPEG_LIBRARIES)
    # openjp2 < 2.5.0 does not sets the INTERFACE_INCLUDE_DIRECTORIES property
    target_include_directories  (${LEPTONICA_LIBRARY} PRIVATE ${OPENJPEG_INCLUDE_DIRS})
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE ${OPENJPEG_LIBRARIES})
endif()
if (PNG_LIBRARIES)
    set(PNG_INCLUDE_DIRS ${PNG_INCLUDE_DIR})
    target_include_directories  (${LEPTONICA_LIBRARY} PRIVATE ${PNG_INCLUDE_DIRS})
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE ${PNG_LIBRARIES})
endif()
if (TIFF_LIBRARIES)
    target_include_directories  (${LEPTONICA_LIBRARY} PRIVATE ${TIFF_INCLUDE_DIR})
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE ${TIFF_LIBRARIES})
endif()
if (WEBP_LIBRARIES)
    if ((CMAKE_BUILD_TYPE STREQUAL "Debug") OR (CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")) 
      set(WEBP_LIBRARIES ${OpenCV_BINARY_DIR}/3rdparty/lib/libwebpd.lib)
    else()
      set(WEBP_LIBRARIES ${OpenCV_BINARY_DIR}/3rdparty/lib/libwebp.lib)
    endif()
  target_link_libraries   (${LEPTONICA_LIBRARY} PRIVATE ${WEBP_LIBRARIES})
  target_include_directories(${LEPTONICA_LIBRARY} PRIVATE ${WEBP_INCLUDE_DIR})
elseif (WebP_LIBRARIES)
    if(TARGET WebP::webp)
        target_link_libraries   (${LEPTONICA_LIBRARY} PRIVATE WebP::webp WebP::libwebpmux)
    else()
        target_link_libraries   (${LEPTONICA_LIBRARY} PRIVATE ${WebP_LIBRARIES})
        target_include_directories(${LEPTONICA_LIBRARY} PRIVATE ${WebP_INCLUDE_DIR})
    endif()
endif()
if (ZLIB_LIBRARIES)
    target_include_directories  (${LEPTONICA_LIBRARY} PRIVATE ${ZLIB_INCLUDE_DIRS})
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE ${ZLIB_LIBRARIES})
endif()

if (UNIX)
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE m)
    set(lib ${CMAKE_SHARED_LIBRARY_PREFIX})
    set(dll ${CMAKE_SHARED_LIBRARY_SUFFIX})
    set(old_target ${lib}lept${dll})
    if (SYM_LINK)
        add_custom_target(lept ALL
            ${CMAKE_COMMAND} -E create_symlink "$<TARGET_FILE:leptonica>" ${old_target}
            DEPENDS ${PROJECT_NAME}
            COMMENT "Creating lept symlink")
        install(FILES ${CMAKE_BINARY_DIR}/${old_target} DESTINATION lib)
    endif(SYM_LINK)
endif()

if (MSVC)
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE user32.lib gdi32.lib)
endif()

if (NOT SW_BUILD)
    export(TARGETS ${LEPTONICA_LIBRARY} FILE ${CMAKE_BINARY_DIR}/LeptonicaTargets.cmake)
else()
    target_link_libraries       (${LEPTONICA_LIBRARY} PRIVATE
        org.sw.demo.gif
        org.sw.demo.jpeg
        org.sw.demo.glennrp.png
        org.sw.demo.tiff
        org.sw.demo.webmproject.webp
        org.sw.demo.uclouvain.openjpeg.openjp2
    )
    file(WRITE ${CMAKE_BINARY_DIR}/LeptonicaTargets.cmake "include(${CMAKE_BINARY_DIR}/cppan.cmake)\n")
    export(TARGETS ${LEPTONICA_LIBRARY} APPEND FILE ${CMAKE_BINARY_DIR}/LeptonicaTargets.cmake)
endif()

install(TARGETS ${LEPTONICA_LIBRARY} EXPORT LeptonicaTargets RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
#install(EXPORT LeptonicaTargets DESTINATION  ${leptonica_INSTALL_CMAKE_DIR})
install(FILES
    ${hdr}
    ${CMAKE_BINARY_DIR}/endianness.h
    DESTINATION include/leptonica)

if(NOT BUILD_SHARED_LIBS)
  ocv_install_target(${LEPTONICA_LIBRARY} EXPORT OpenCVModules ARCHIVE DESTINATION ${OPENCV_3P_LIB_INSTALL_PATH} COMPONENT dev)
endif()
