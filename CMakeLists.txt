cmake_minimum_required(VERSION 3.16)

set(BOOST_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/boost_1_72_0/include)
set(WEBRTC_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/webrtc/include)

set(BOOST_LIBRARY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/boost_1_72_0/lib)

if("${TARGET}" STREQUAL "ubuntu")
    set(CMAKE_C_COMPILER clang)
    set(CMAKE_CXX_COMPILER clang++)
    set(LIBCXX_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libcxx/include)
endif()

if("${TARGET}" STREQUAL "windows")
set(WEBRTC_LIBRARY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/webrtc/release)
elseif("${TARGET}" STREQUAL "macos")
set(WEBRTC_LIBRARY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/webrtc/lib)
else()
set(WEBRTC_LIBRARY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/webrtc/lib)
endif()

project(native-webrtc-loopback C CXX)

add_executable(native-webrtc-loopback main.cpp)
target_compile_features(native-webrtc-loopback PUBLIC cxx_std_20)

if("${TARGET}" STREQUAL "ubuntu")
target_compile_options(native-webrtc-loopback
PRIVATE
  "$<$<COMPILE_LANGUAGE:CXX>:-nostdinc++>"
  "$<$<AND:$<COMPILE_LANGUAGE:CXX>,$<BOOL:LIBCXX_INCLUDE_DIR>>:-isystem${LIBCXX_INCLUDE_DIR}>"
)
endif()

include_directories(${WEBRTC_INCLUDE_DIR})
include_directories(${WEBRTC_INCLUDE_DIR}/third_party/abseil-cpp)
include_directories(${BOOST_INCLUDE_DIR})
link_directories(${WEBRTC_LIBRARY_DIR})

if("${TARGET}" STREQUAL "windows")
    set_target_properties(native-webrtc-loopback PROPERTIES MSVC_RUNTIME_LIBRARY "MultiThreaded")

    target_link_libraries(native-webrtc-loopback
        winmm.lib
        dmoguids.lib
        msdmo.lib
        Secur32.lib
        wmcodecdspuuid.lib
    )
    target_compile_definitions(native-webrtc-loopback PUBLIC WEBRTC_WIN)
    target_link_libraries(native-webrtc-loopback ${WEBRTC_LIBRARY_DIR}/webrtc.lib)
elseif("${TARGET}" STREQUAL "macos")
    target_link_libraries(native-webrtc-loopback
        "-framework AVFoundation"
        "-framework AudioUnit"
        "-framework AudioToolbox"
        "-framework CoreAudio"
        "-framework CoreGraphics"
        "-framework CoreMedia"
        "-framework CoreVideo"
        "-framework VideoToolbox"
        "-framework AppKit"
        "-framework OpenGL"
    )

    target_compile_definitions(native-webrtc-loopback PUBLIC
        WEBRTC_POSIX
        WEBRTC_MAC
    )
    target_link_libraries(native-webrtc-loopback ${WEBRTC_LIBRARY_DIR}/libwebrtc.a)
elseif("${TARGET}" STREQUAL "ubuntu")
    find_package(Threads REQUIRED)
    target_link_libraries(native-webrtc-loopback
        X11
        dl
        Threads::Threads
    )

    target_compile_definitions(native-webrtc-loopback PUBLIC
        WEBRTC_POSIX
        _LIBCPP_DISABLE_AVAILABILITY
        _LIBCPP_ABI_UNSTABLE
    )
    target_link_libraries(native-webrtc-loopback ${WEBRTC_LIBRARY_DIR}/libwebrtc.a)
endif()
