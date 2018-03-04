cmake_minimum_required(VERSION 3.9)
project(currency)

set(EOSIO_INSTALL_DIR $ENV{HOME}/eos/build/install)
set(BOOST_INSTALL_DIR $ENV{HOME}/opt/boost_1_66_0)
set(WASM_INSTALL_DIR $ENV{HOME}/opt/wasm)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_C_COMPILER ${WASM_INSTALL_DIR}/bin/clang)
set(CMAKE_CXX_COMPILER ${WASM_INSTALL_DIR}/bin/clang++)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --target=wasm32")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdlib")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-threadsafe-statics")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")

include_directories(.)
include_directories(${EOSIO_INSTALL_DIR}/include)
include_directories(${EOSIO_INSTALL_DIR}/include/libc++/upstream/include)
include_directories(${EOSIO_INSTALL_DIR}/include/musl/upstream/include)
include_directories(${BOOST_INSTALL_DIR}/include)

add_library(currency
        currency.cpp
        currency.hpp)
