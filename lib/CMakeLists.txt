set(VORBIS_PUBLIC_HEADERS
    ../include/vorbis/codec.h
    ../include/vorbis/vorbisenc.h
    ../include/vorbis/vorbisfile.h
)

set(VORBIS_HEADERS
    envelope.h
    lpc.h
    lsp.h
    codebook.h
    misc.h
    psy.h
    masking.h
    os.h
    mdct.h
    smallft.h
    highlevel.h
    registry.h
    scales.h
    window.h
    lookup.h
    lookup_data.h
    codec_internal.h
    backends.h
    bitrate.h
)

set(VORBIS_SOURCES
    mdct.c
    smallft.c
    block.c
    envelope.c
    window.c
    lsp.c
    lpc.c
    analysis.c
    synthesis.c
    psy.c
    info.c
    floor1.c
    floor0.c
    res0.c
    mapping0.c
    registry.c
    codebook.c
    sharedbook.c
    lookup.c
    bitrate.c
    vorbisenc.c
)

set(VORBISFILE_SOURCES
    vorbisfile.c
)

if(MSVC)
    list(APPEND VORBIS_SOURCES ../win32/vorbis.def)
    list(APPEND VORBISFILE_SOURCES ../win32/vorbisfile.def)
endif()

# include_directories(../include)
# include_directories(.)
list(APPEND inc_dirs
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}/../include")

write_basic_package_version_file("${PROJECT_NAME}ConfigVersion.cmake" COMPATIBILITY SameMajorVersion)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
	DESTINATION lib/cmake)

# -> static libvorbis
list(APPEND targets "libvorbis_static")
add_library(libvorbis_static STATIC ${VORBIS_PUBLIC_HEADERS} ${VORBIS_HEADERS} ${VORBIS_SOURCES} ${VORBISFILE_SOURCES} ${VORBISENC_SOURCES})
if (BUILD_FRAMEWORK)
    set_target_properties(libvorbis_static PROPERTIES
        FRAMEWORK TRUE
        FRAMEWORK_VERSION ${PROJECT_VERSION}
        MACOSX_FRAMEWORK_IDENTIFIER org.xiph.vorbis
        MACOSX_FRAMEWORK_SHORT_VERSION_STRING ${PROJECT_VERSION}
        MACOSX_FRAMEWORK_BUNDLE_VERSION ${PROJECT_VERSION}
        XCODE_ATTRIBUTE_INSTALL_PATH "@rpath"
        PUBLIC_HEADER "${VORBIS_PUBLIC_HEADERS}")
endif()
set_target_properties(libvorbis_static PROPERTIES
    INCLUDE_DIRECTORIES "${inc_dirs}"
    INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>;$<BUILD_INTERFACE:${inc_dirs}>"
    PREFIX "")
target_link_libraries(libvorbis_static ${OGG_LIB_NAME})

# -> shared libvorbis
if(BUILD_SHARED_LIBS)
    list(APPEND targets "libvorbis")
    add_library(libvorbis SHARED ${VORBIS_HEADERS} ${VORBIS_SOURCES})
    get_version_info(VORBIS_VERSION_INFO "V_LIB_CURRENT" "V_LIB_AGE" "V_LIB_REVISION")
    set_target_properties(libvorbis PROPERTIES
        INCLUDE_DIRECTORIES "${inc_dirs}"
        INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>;$<BUILD_INTERFACE:${inc_dirs}>"
        SOVERSION ${VORBIS_VERSION_INFO}
        IMPORT_SUFFIX "_imp${CMAKE_IMPORT_LIBRARY_SUFFIX}"
        PREFIX "")
    if (BUILD_FRAMEWORK)
        set_target_properties(libvorbis PROPERTIES
            FRAMEWORK TRUE
            FRAMEWORK_VERSION ${PROJECT_VERSION}
            MACOSX_FRAMEWORK_IDENTIFIER org.xiph.vorbis
            MACOSX_FRAMEWORK_SHORT_VERSION_STRING ${PROJECT_VERSION}
            MACOSX_FRAMEWORK_BUNDLE_VERSION ${PROJECT_VERSION}
            XCODE_ATTRIBUTE_INSTALL_PATH "@rpath"
            PUBLIC_HEADER "${VORBIS_PUBLIC_HEADERS}")
    endif()
    target_link_libraries(libvorbis ${OGG_LIB_NAME})
endif()

# -> shared libvorbisfile
if (NOT BUILD_FRAMEWORK AND BUILD_SHARED_LIBS)
    list(APPEND targets
        "libvorbisfile")

    add_library(libvorbisfile SHARED ${VORBISFILE_SOURCES})

    get_version_info(VORBISFILE_VERSION_INFO "VF_LIB_CURRENT" "VF_LIB_AGE" "VF_LIB_REVISION")

    set_target_properties(libvorbisfile PROPERTIES
        INCLUDE_DIRECTORIES "${inc_dirs}"
        INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>;$<BUILD_INTERFACE:${inc_dirs}>"
        IMPORT_SUFFIX "_imp${CMAKE_IMPORT_LIBRARY_SUFFIX}"
        PREFIX ""
        SOVERSION ${VORBISFILE_VERSION_INFO})

    target_link_libraries(libvorbisfile libvorbis ${OGG_LIB_NAME})
endif()

# -> static libvorbisfile
if (NOT BUILD_FRAMEWORK)
    list(APPEND targets
        "libvorbisfile_static")

    add_library(libvorbisfile_static STATIC ${VORBISFILE_SOURCES})

    get_version_info(VORBISFILE_VERSION_INFO "VF_LIB_CURRENT" "VF_LIB_AGE" "VF_LIB_REVISION")

    set_target_properties(libvorbisfile_static PROPERTIES
        INCLUDE_DIRECTORIES "${inc_dirs}"
        INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>;$<BUILD_INTERFACE:${inc_dirs}>"
        PREFIX ""
        SOVERSION ${VORBISFILE_VERSION_INFO})

    target_link_libraries(libvorbisfile_static libvorbis_static ${OGG_LIB_NAME})
endif()

install(FILES ${VORBIS_PUBLIC_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/vorbis)

install(TARGETS ${targets}
    EXPORT ${PROJECT_NAME}Config
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR})
