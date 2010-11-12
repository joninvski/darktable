cmake_minimum_required(VERSION 2.6)

#
# Add files for libdarktable
#
FILE(GLOB SOURCE_FILES
  "common/collection.c" 
  "common/colorlabels.c" 
  "common/colormatrices.c" 
  "common/colorspaces.c" 
  "common/darktable.c" 
  "common/exif.cc" 
  "common/film.c" 
  "common/fswatch.c" 
  "common/history.c" 
  "common/image.c" 
  "common/image_cache.c" 
  "common/image_compression.c" 
  "common/imageio.c" 
  "common/imageio_jpeg.c" 
  "common/imageio_module.c" 
  "common/imageio_pfm.c" 
  "common/imageio_rgbe.c" 
  "common/imageio_tiff.c" 
  "common/styles.c" 
  "common/tags.c" 
  "common/variables.c" 
  "common/pwstorage/pwstorage.c" 
  "common/pwstorage/backend_gconf.c"
  "control/control.c"  
  "control/jobs/control_jobs.c"  
  "control/jobs/develop_jobs.c"  
  "control/jobs/film_jobs.c"  
  "control/jobs/image_jobs.c"  
  "develop/develop.c"
  "develop/imageop.c"
  "develop/pixelpipe.c"
  "dtgtk/*.c" 
  "gui/background_jobs.c"  
  "gui/contrast.c"  
  "gui/filmview.c"  
  "gui/gtk.c"  
  "gui/histogram.c"  
  "gui/iop_history.c"  
  "gui/iop_modulegroups.c"  
  "gui/metadata.c"  
  "gui/navigation.c"  
  "gui/preferences.c"  
  "gui/presets.c"  
  "gui/styles_dialog.c"  
  "gui/tool_colorlabels.c"  
  "libs/lib.c"
  "views/view.c"
  )
FILE(GLOB HEADER_FILES "*.h" "common/*.h" "control/*.h" "iop/*.h" "libs/*.h", "views/*.h")

set(SOURCES ${SOURCE_FILES} ${HEADER_FILES})

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/LibRaw)

# initial compiler flags
add_definitions("-DHAVE_CONFIG_H")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")

# Check for openmp
find_package(OpenMP)
if(OpenMP_FOUND)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
  add_definitions("-DHAVE_OPENMP")
endif(OpenMP_FOUND)

#
# Find all other required libraries for building
#
find_package(GTK2 2.6 REQUIRED gtk glade)
include_directories(${GTK2_INCLUDE_DIRS})
list(APPEND LIBS ${GTK2_LIBRARIES})
add_definitions(${GTK2_DEFINITIONS})

# Check for libxml2 / broken cmake module can't be included in the foreach() below
find_package(LibXml2 2.6 REQUIRED)
include_directories(${LIBXML2_INCLUDE_DIR})
list(APPEND LIBS ${LIBXML2_LIBRARIES})
add_definitions(${LIBXML2_DEFINITIONS})


# Check for gconf2 / broken cmake module can't be included in the foreach() below
if(USE_GCONF_BACKEND)
  find_package(GConf2)
  include_directories(${GConf2_INCLUDE_DIRS})
  list(APPEND LIBS ${GConf2_LIBRARIES})
  add_definitions(${GConf2_DEFINITIONS})
endif(USE_GCONF_BACKEND)



if(USE_CAMERA_SUPPORT)
  find_package(Gphoto2)
  include_directories(${Gphoto2_INCLUDE_DIRS})
  list(APPEND LIBS ${Gphoto2_LIBRARIES})
  add_definitions(${Gphoto2_DEFINITIONS})
endif(USE_CAMERA_SUPPORT)

# Find all the libs that don't require extra parameters
foreach(lib ${OUR_LIBS} LensFun GThread GModule PangoCairo PThread Rsvg2 GDK-PixBuf LibXml2 INotify Sqlite3 Exiv2  CURL PNG JPEG TIFF OpenEXR LCMS)
  find_package(${lib} REQUIRED)
  include_directories(${${lib}_INCLUDE_DIRS})
  list(APPEND LIBS ${${lib}_LIBRARIES})
  add_definitions(${${lib}_DEFINITIONS})
endforeach(lib)

find_package(Gettext)
if(Gettext_FOUND)
  include_directories(${Gettext_INCLUDE_DIRS})
  list(APPEND LIBS ${Gettext_LIBRARIES})
  add_definitions("-DUSE_GETTEXT")
  message(STATUS "Internationalization: Enabled")
else()
  message(STATUS "Internationalization: Disabled (libintl not found)")
endif()

#
# Detect compile of optional pwstorage backends
#
find_package(DBUSGLIB)
if(DBUSGLIB_FOUND)
  list(APPEND SOURCES "common/pwstorage/backend_kwallet.c")
  include_directories(${DBUSGLIB_INCLUDE_DIRS})
  list(APPEND LIBS ${DBUSGLIB_LIBRARIES})
  add_definitions("-DHAVE_KWALLET")
endif(DBUSGLIB_FOUND)

find_package(GnomeKeyring)
if(GNOMEKEYRING_FOUND)
  set(SOURCES ${SOURCES} "common/pwstorage/backend_gkeyring.c")
  include_directories(${GnomeKeyring_INCLUDE_DIRS})
  list(APPEND LIBS ${GnomeKeyring_LIBRARIES})
  add_definitions("-DHAVE_GKEYRING")
endif(GNOMEKEYRING_FOUND)

find_package(OpenCL)
if(OPENCL_FOUND)
  include_directories($OPENCL_INCLUDE_DIR)
  list(APPEND LIBS $OPENCL_LIBRARIES)
  list(APPEND SOURCE_FILES "common/opencl.c")
  add_definitions("-DHAVE_OPENCL")
endif(OPENCL_FOUND)



#
# Add HAVE_xxx defines used by darktable
#
if(INOTIFY_FOUND)
  add_definitions("-DHAVE_INOTIFY")
endif(INOTIFY_FOUND)

if(LENSFUN_FOUND)
  add_definitions("-DHAVE_LENSFUN")
endif(LENSFUN_FOUND)

if(GCONF2_FOUND)
  add_definitions("-DHAVE_GCONF")
endif(GCONF2_FOUND)

if(OPENEXR_FOUND)
  add_definitions("-DHAVE_OPENEXR")
endif(OPENEXR_FOUND)


if(GPHOTO2_FOUND)
  set(SOURCES ${SOURCES} "common/camera_control.c" "control/jobs/camera_jobs.c" "gui/camera_import_dialog.c" "gui/devices.c" )
  add_definitions("-DHAVE_GPHOTO2")
endif(GPHOTO2_FOUND)


#
# Set default compile flags for GCC
#
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-missing-field-initializers -fno-strict-aliasing")
# c++ code doesn't like -Werror (libraw)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-missing-field-initializers -fno-strict-aliasing")

if(NOT APPLE)
  # apple build fails on deprecated warnings..
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror")
endif(NOT APPLE)

if(NOT WIN32)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pthread")
endif(NOT WIN32)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -msse2 -mfpmath=sse -march=native -g")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -ffast-math -fexpensive-optimizations")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0 -g -DDEBUG")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse2 -mfpmath=sse -march=native -g")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -ffast-math -fexpensive-optimizations")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g -DDEBUG")
  

#
# Generate config.h
#
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.h" "${CMAKE_CURRENT_BINARY_DIR}/config.h" @ONLY)
include_directories("${CMAKE_CURRENT_BINARY_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")


#
# Setup generation of preference dialog files..
#
add_custom_command(
   OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/preferences_gen.h
   COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/../tools/create_preferences.sh ${CMAKE_CURRENT_SOURCE_DIR}/..//darktable.schemas.in > ${CMAKE_CURRENT_BINARY_DIR}/preferences_gen.h
)

#
# build libdarktable
#
add_library(lib_darktable SHARED ${CMAKE_CURRENT_BINARY_DIR}/preferences_gen.h ${SOURCES})
if(APPLE)
  set_target_properties(lib_darktable PROPERTIES INSTALL_NAME_DIR "@executable_path/../lib/darktable")
endif(APPLE)
set_target_properties(lib_darktable PROPERTIES OUTPUT_NAME darktable)
set_target_properties(lib_darktable PROPERTIES LINKER_LANGUAGE C)


#
# the libraw part is a bit of a hack:
# the static linking didn't work since it was pulling -lstdc++ and -lm into linker flags.
# so we do a custom dependency and pretend an imported liblibraw_r.a so no other -l are
# appended.
#
add_dependencies(lib_darktable libraw_r)
add_library(libraw_static STATIC IMPORTED)
set_target_properties(libraw_static PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/LibRaw/liblibraw_r.a)
target_link_libraries(lib_darktable ${LIBS} libraw_static)
install(TARGETS lib_darktable DESTINATION lib/darktable)
list(APPEND LIBS lib_darktable)

# Compile views modules
add_subdirectory(LibRaw)

# Compile views modules
add_subdirectory(views)

# Compile lighttable modules
add_subdirectory(libs)

# Compile darkroom modules
add_subdirectory(iop)

# Compile darkroom modules
add_subdirectory(imageio)




#
# build darktable executable
#
add_executable(darktable ${SUBSYSTEM_MACOSX} ${SUBSYSTEM_WIN32} main.c ${RESOURCE_OBJECT})
set_target_properties(darktable PROPERTIES INSTALL_RPATH ${CMAKE_INSTALL_PREFIX}/lib/darktable)
set_target_properties(darktable PROPERTIES LINKER_LANGUAGE C)
target_link_libraries(darktable ${LIBS})
install(TARGETS darktable DESTINATION bin)



