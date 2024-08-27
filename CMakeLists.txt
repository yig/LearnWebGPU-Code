cmake_minimum_required(VERSION 3.14...3.25)
project(
	LearnWebGPU
	VERSION 0.1.0
	LANGUAGES CXX C
)

if (NOT EMSCRIPTEN)
	add_subdirectory(glfw)
else()
	add_library(glfw INTERFACE)
	target_link_options(glfw INTERFACE -sUSE_GLFW=3)
endif()

include(FetchContent)
set(FETCHCONTENT_QUIET FALSE)
# set(WEBGPU_BACKEND "wgpu" CACHE STRING "WebGPU backend (wgpu or dawn)")
# FetchContent_Declare(
#   webgpu
#   GIT_REPOSITORY https://github.com/yig/WebGPU-distribution
#   ## Can change `wgpu` to `dawn`
#   GIT_TAG        ${WEBGPU_BACKEND}
#   GIT_SHALLOW TRUE
#   GIT_PROGRESS TRUE
# )
# FetchContent_MakeAvailable( webgpu )

FetchContent_Declare(
    wgpurs
    URL https://github.com/gfx-rs/wgpu-native/releases/download/v22.1.0.1/wgpu-macos-aarch64-release.zip
)
FetchContent_MakeAvailable( wgpurs )
add_library( webgpu STATIC IMPORTED )
file( MAKE_DIRECTORY ${wgpurs_SOURCE_DIR}/webgpu )
file( COPY ${wgpurs_SOURCE_DIR}/webgpu.h ${wgpurs_SOURCE_DIR}/wgpu.h DESTINATION ${wgpurs_SOURCE_DIR}/webgpu )
set_target_properties( webgpu PROPERTIES IMPORTED_LOCATION ${wgpurs_SOURCE_DIR}/libwgpu_native.a )
target_include_directories( webgpu INTERFACE ${wgpurs_SOURCE_DIR} )

add_subdirectory(glfw3webgpu)

add_executable(App
	main.cpp
	webgpu-utils.h
	webgpu-utils.cpp
)

target_link_libraries(App PRIVATE glfw webgpu glfw3webgpu)
target_link_libraries( App PRIVATE "-framework Metal" )

# target_copy_webgpu_binaries(App)

set_target_properties(App PROPERTIES
	CXX_STANDARD 17
	CXX_STANDARD_REQUIRED ON
	CXX_EXTENSIONS OFF
	COMPILE_WARNING_AS_ERROR ON
)

if (MSVC)
	target_compile_options(App PRIVATE /W4)
else()
	target_compile_options(App PRIVATE -Wall -Wextra -pedantic)
endif()

if (XCODE)
	set_target_properties(App PROPERTIES
		XCODE_GENERATE_SCHEME ON
		XCODE_SCHEME_ENABLE_GPU_FRAME_CAPTURE_MODE "Metal"
	)
endif()

if (EMSCRIPTEN)
	set_target_properties(App PROPERTIES SUFFIX ".html")
	target_link_options(App PRIVATE -sASYNCIFY)
endif()
