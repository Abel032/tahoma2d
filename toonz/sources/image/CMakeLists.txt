set(HEADERS
    sgi/tiio_sgi.h
    pli/pli_io.h
    pli/tiio_pli.h
    tga/tiio_tga.h
    compatibility/inforegion.h
    compatibility/tfile_io.h
    compatibility/tnz4.h
    quantel/filequantel.h
    quantel/filequantelP.h
    quantel/tiio_quantel.h
    png/tiio_png.h
    psd/tiio_psd.h
    svg/tiio_svg.h
    tzl/tiio_tzl.h
    ../include/tnzimage.h
    ffmpeg/tiio_gif.h
    ffmpeg/tiio_webm.h
    ffmpeg/tiio_apng.h
    ffmpeg/tiio_mp4.h
    ffmpeg/tiio_ffmpeg.h
    ffmpeg/tiio_ff_mov.h
    sprite/tiio_sprite.h
    mesh/tiio_mesh.h
    exr/tinyexr_otmod.h
    exr/tiio_exr.h
)

set(SOURCES
    tiio.cpp
    sgi/filesgi.cpp
    pli/pli_io.cpp
    pli/tags.cpp
    pli/tiio_pli.cpp
    tga/tiio_tga.cpp
    compatibility/inforegion.c
    compatibility/tfile_io.c
    compatibility/tnz4.c
    compatibility/tnz4_cpp.cpp
    quantel/filequantel.c
    quantel/tiio_quantel.cpp
    png/tiio_png.cpp
    psd/tiio_psd.cpp
    svg/tiio_svg.cpp
    tzl/tiio_tzl.cpp
    ffmpeg/tiio_gif.cpp
    ffmpeg/tiio_webm.cpp
    ffmpeg/tiio_apng.cpp
    ffmpeg/tiio_mp4.cpp
    ffmpeg/tiio_ffmpeg.cpp
    ffmpeg/tiio_ff_mov.cpp
    sprite/tiio_sprite.cpp
    mesh/tiio_mesh.cpp
    exr/tiio_exr.cpp
)


# Can't build this on FreeBSD: libtiff requires internal API access.
# Probably some one get luck to fix this.
if(NOT BUILD_TARGET_BSD)
    set(HEADERS ${HEADERS}
        tif/tiio_tif.h
        tzp/tiio_plt.h
        tzp/tiio_tzp.h
        tzp/toonztags.h
    )
    set(SOURCES ${SOURCES}
        tif/tiio_tif.cpp
        tzp/tiio_plt.cpp
        tzp/tiio_tzp.cpp
        tzp/toonzrle.cpp
        tzp/avl.c
    )
endif()


if(BUILD_TARGET_WIN)
    set(HEADERS ${HEADERS}
        avi/tiio_avi.h
    )
    set(SOURCES ${SOURCES}
        avi/tiio_avi.cpp
    )
endif()

if(WITH_TRANSLATION)
    add_translation(image ${HEADERS} ${SOURCES})
endif()

add_library(image SHARED ${HEADERS} ${SOURCES})
add_definitions(
    -DTNZIMAGE_DLL
    -D_USRDLL
    -DIMAGE_EXPORTS
)

if(BUILD_ENV_APPLE)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-install_name,@executable_path/libimage.dylib")
    add_dependencies(image tnzcore tnzbase toonzlib)
endif()

message("subdir: image")
message("Bin: " ${CMAKE_CURRENT_BINARY_DIR})

include_directories(
    SYSTEM
    ${TIFF_INCLUDE_DIR}
)

if(NOT BUILD_TARGET_BSD)
    include_directories(
        ${SDKROOT}/LibJPEG/jpeg-9
    )
endif()


if(BUILD_ENV_MSVC)
    include_directories(
        SYSTEM
        ${SDKROOT}/libpng-1.6.21
    )
endif()
if(BUILD_TARGET_WIN AND PLATFORM EQUAL 32)
    include_directories(
        SYSTEM
    )
endif()

include_directories(
    ${SDKROOT}/tinyexr
)


_find_toonz_library(TNZLIBS "tnzcore;tnzbase;toonzlib")


if(BUILD_ENV_MSVC)
    set(EXTRA_LIBS
        ${TNZLIBS}
        vfw32.lib
    )
elseif(BUILD_ENV_APPLE)
    if(PLATFORM EQUAL 32)
        find_library(CORE_SERVICES_LIB CoreServices)
        set(OSX_10_6_SDK_PATH "")
        if(NOT OSX_10_6_SDK_PATH)
            message(FATAL_ERROR "Set variable 'OSX_10_6_SDK_PATH'")
        endif()
        find_library(QD_LIB
            NAMES QD
            PATHS OSX10_6_SDK_PATH
        )
        set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -F ${OSX_10_6_SDK_PATH}")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -F ${OSX_10_6_SDK_PATH}")
    endif()
    set(EXTRA_LIBS
        ${TNZLIBS}
        ${QT_LIB}
        ${CARBON_LIB}
        ${CORE_SERVICES_LIB}
        ${QD_LIB}
        ${DEFLATE_LIB}		
    )
elseif(BUILD_ENV_UNIXLIKE)
    # Generic Unix
    set(EXTRA_LIBS
        ${TNZLIBS}
        ${DEFLATE_LIB}
    )

    if(BUILD_TARGET_WIN)
        set(EXTRA_LIBS ${EXTRA_LIBS} -lvfw32)
    endif()
endif()

target_link_libraries(image Qt5::Core Qt5::Widgets Qt5::Gui Qt5::Network ${Z_LIB} ${GLUT_LIB} ${GL_LIB} ${JPEG_LIB} ${TIFF_LIB} ${PNG_LIB} ${EXTRA_LIBS})
