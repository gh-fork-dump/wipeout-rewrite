cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
project(wipeout-rewrite)

include(GNUInstallDirs)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")

option(RENDERER_GL "Use OpenGL as the graphics renderer" ON)
option(PLATFORM_SDL2 "Use SDL2 for platform management" ON)
option(PLATFORM_SOKOL "Use Sokol for platform management" OFF)

set(common_src
	src/wipeout/camera.c
	src/wipeout/camera.h
	src/wipeout/droid.c
	src/wipeout/droid.h
	src/wipeout/game.c
	src/wipeout/game.h
	src/wipeout/hud.c
	src/wipeout/hud.h
	src/wipeout/image.c
	src/wipeout/image.h
	src/wipeout/ingame_menus.c
	src/wipeout/ingame_menus.h
	src/wipeout/intro.c
	src/wipeout/intro.h
	src/wipeout/main_menu.c
	src/wipeout/main_menu.h
	src/wipeout/menu.c
	src/wipeout/menu.h
	src/wipeout/object.c
	src/wipeout/object.h
	src/wipeout/particle.c
	src/wipeout/particle.h
	src/wipeout/race.c
	src/wipeout/race.h
	src/wipeout/scene.c
	src/wipeout/scene.h
	src/wipeout/sfx.c
	src/wipeout/sfx.h
	src/wipeout/ship.c
	src/wipeout/ship.h
	src/wipeout/ship_ai.c
	src/wipeout/ship_ai.h
	src/wipeout/ship_player.c
	src/wipeout/ship_player.h
	src/wipeout/title.c
	src/wipeout/title.h
	src/wipeout/track.c
	src/wipeout/track.h
	src/wipeout/ui.c
	src/wipeout/ui.h
	src/wipeout/weapon.c
	src/wipeout/weapon.h
	src/input.c
	src/input.h
	src/mem.c
	src/mem.h
	src/platform.h
	src/render.h
	src/system.c
	src/system.h
	src/types.c
	src/types.h
	src/utils.c
	src/utils.h

	packaging/windows/wipegame.exe.manifest
	packaging/windows/wipegame.rc
)

add_executable(wipegame WIN32)
set_property(TARGET wipegame PROPERTY C_STANDARD 11)
target_sources(wipegame PRIVATE ${common_src})
target_include_directories(wipegame PRIVATE src)

if(WIN32)
	if(MSVC)
		target_compile_definitions(wipegame PRIVATE
			"NOMINMAX"
			"_USE_MATH_DEFINES"
			"_CRT_SECURE_NO_WARNINGS"
		)
	endif()
elseif(LINUX)
	target_link_libraries(wipegame PUBLIC m)
	set_property(TARGET wipegame PROPERTY ENABLE_EXPORTS ON)
	if (PLATFORM_SOKOL)
		find_package(Threads REQUIRED)
		find_package(X11 REQUIRED)
		find_package(ALSA REQUIRED)
		target_link_libraries(wipegame PUBLIC
			X11::X11
			X11::Xcursor
			Threads::Threads
			X11::Xi
			dl
			ALSA::ALSA
		)
	endif()
elseif(APPLE)
	target_compile_definitions(wipegame PRIVATE "_THREAD_SAFE")
	target_link_libraries(wipegame PUBLIC "-framework Foundation")
	if (PLATFORM_SOKOL)
		target_link_libraries(wipegame PUBLIC 
			"-framework Cocoa"
			"-framework QuartzCore"
			"-framework AudioToolbox"
		)
		set_source_files_properties(src/platform_sokol.c PROPERTIES COMPILE_FLAGS "-x objective-c")
	endif()
endif()

if(RENDERER_GL)
	find_package(OpenGL REQUIRED)
	find_package(GLEW)
	target_compile_definitions(wipegame PRIVATE "RENDERER_GL")
	target_sources(wipegame PRIVATE src/render_gl.c)
	target_include_directories(wipegame PUBLIC ${OPENGL_INCLUDE_DIR})
	target_link_libraries(wipegame PUBLIC OpenGL::GL)
	if(APPLE)
		target_compile_definitions(wipegame PRIVATE "GL_SILENCE_DEPRECATION")
		target_link_libraries(wipegame PUBLIC "-framework OpenGL")
	else()
		target_include_directories(wipegame PRIVATE ${GLEW_INCLUDE_DIRS})
		target_link_libraries(wipegame PRIVATE GLEW::GLEW)
	endif()
endif()

if(PLATFORM_SDL2)
	find_package(SDL2 REQUIRED)
	target_sources(wipegame PRIVATE src/platform_sdl.c)
	target_include_directories(wipegame PUBLIC ${SDL2_INCLUDE_DIR})
	target_link_libraries(wipegame PUBLIC SDL2::Main)
elseif(PLATFORM_SOKOL)
	target_sources(wipegame PRIVATE src/platform_sokol.c)
endif()
