
cmake_minimum_required(VERSION 2.6)

include_directories("${CMAKE_CURRENT_BINARY_DIR}/../" "${CMAKE_CURRENT_SOURCE_DIR}")

# The modules
set(MODULES import export copy_history styles tagging image select collect recentcollect metadata metadata_view navigation histogram history snapshots modulelist modulegroups backgroundjobs colorpicker similarity keywords hinter)

# The tools
set(MODULES ${MODULES} viewswitcher)
set(MODULES ${MODULES} darktable_label)
set(MODULES ${MODULES} ratings)
set(MODULES ${MODULES} colorlabels)
set(MODULES ${MODULES} filter)
set(MODULES ${MODULES} lighttable_mode)
set(MODULES ${MODULES} view_toolbox)
set(MODULES ${MODULES} module_toolbox)
set(MODULES ${MODULES} filmstrip)
set(MODULES ${MODULES} hinter)
set(MODULES ${MODULES} global_toolbox)

# modules
add_library(import MODULE "import.c")
add_library(export MODULE "export.c")
add_library(copy_history MODULE "copy_history.c")
add_library(styles MODULE "styles.c")
add_library(image MODULE "image.c")
add_library(tagging MODULE "tagging.c")
add_library(select MODULE "select.c")
add_library(collect MODULE "collect.c")
add_library(recentcollect MODULE "recentcollect.c")
add_library(metadata MODULE "metadata.c")
add_library(similarity MODULE "similarity.c")
add_library(metadata_view MODULE "metadata_view.c")
add_library(navigation MODULE "navigation.c")
add_library(histogram MODULE "histogram.c")
add_library(history MODULE "history.c")
add_library(snapshots MODULE "snapshots.c")
add_library(modulelist MODULE "modulelist.c")
add_library(modulegroups MODULE "modulegroups.c")
add_library(backgroundjobs MODULE "backgroundjobs.c")
add_library(colorpicker MODULE "colorpicker.c")
add_library(keywords MODULE "keywords.c")

# tools
add_library(viewswitcher MODULE "tools/viewswitcher.c")
add_library(darktable_label MODULE "tools/darktable.c")
add_library(colorlabels MODULE "tools/colorlabels.c")
add_library(ratings MODULE "tools/ratings.c")
add_library(filter MODULE "tools/filter.c")
add_library(lighttable_mode MODULE "tools/lighttable.c")
add_library(view_toolbox MODULE "tools/view_toolbox.c")
add_library(module_toolbox MODULE "tools/module_toolbox.c")
add_library(filmstrip MODULE "tools/filmstrip.c")
add_library(hinter MODULE "tools/hinter.c")
add_library(global_toolbox MODULE "tools/global_toolbox.c")

if(APRIL_FOOLS)
	if(VTE_FOUND)
		add_library(file_manager MODULE "file_manager.c")
		set(MODULES ${MODULES} file_manager)
		if(VTE_VERSION LESS "0.26.0")
			add_definitions(-DVTE_DEPRECATED=1)
			message("--- vte < 0.26 - using deprecated function")
		else(VTE_VERSION LESS "0.26.0")
			message("--- vte >= 0.26 - using recent function")
		endif(VTE_VERSION LESS "0.26.0")
	endif(VTE_FOUND)
endif(APRIL_FOOLS)

# check if gphoto2 found and add build of camera modules...
if(GPHOTO2_FOUND)
	add_library(capture MODULE "capture.c")
	add_library(camera MODULE "camera.c")
	set(MODULES ${MODULES} capture camera)
endif(GPHOTO2_FOUND)

# Add libs references
foreach(module ${MODULES})
	target_link_libraries(${module} ${LIBS})
endforeach(module)

# Add installation target for each module
# TODO: Modules are now not only lighttable related and should be placed
#       in a better matching structure then just in lighttable directory. 
foreach(module ${MODULES})
	install(TARGETS  ${module} DESTINATION ${LIB_INSTALL}/darktable/plugins/lighttable)
endforeach(module)

