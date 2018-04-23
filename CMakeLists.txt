cmake_minimum_required(VERSION 3.9)
project(currency CXX)


set(BOOST_INSTALL_DIR $ENV{HOME}/opt/boost_1_66_0)
set(WASM_INSTALL_DIR $ENV{HOME}/opt/wasm)

set(EOSIO_INSTALL_DIR /usr/local)
set(S2WASM_PATH ${EOSIO_INSTALL_DIR}/bin/eosio-s2wasm)

set(EOSIO_CONTRACTSDK_PATH ${EOSIO_INSTALL_DIR}/usr/share/eosio/contractsdk)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -emit-llvm")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++14")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffreestanding")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdinc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdlib")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdlibinc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-threadsafe-statics")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")

include_directories(include)
include_directories(${EOSIO_INSTALL_DIR}/include)
include_directories(${EOSIO_INSTALL_DIR}/include/libc++/upstream/include)
include_directories(${EOSIO_INSTALL_DIR}/include/musl/upstream/include)
include_directories(${BOOST_INSTALL_DIR}/include)

set(CPP_FILES
        ${CMAKE_PROJECT_NAME}/${CMAKE_PROJECT_NAME}.cpp)

set(HPP_FILES)

add_library(link SHARED
        ${CPP_FILES}
        ${HPP_FILES})

target_link_libraries(link
        ${EOSIO_CONTRACTSDK_PATH}/lib/libc.bc
        ${EOSIO_CONTRACTSDK_PATH}/lib/libc++.bc
        ${EOSIO_CONTRACTSDK_PATH}/lib/eosiolib.bc)

set(CONTRACT_ASSEMBLY_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.s)
set(CONTRACT_WAST_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.wast)

add_custom_target(assembly ALL
        DEPENDS link)
add_custom_command(OUTPUT ${CONTRACT_ASSEMBLY_FILE}
        COMMAND ${WASM_INSTALL_DIR}/bin/llc
        -thread-model=single
        --asm-verbose=false
        -o ${CONTRACT_ASSEMBLY_FILE} $<TARGET_FILE:link>
        DEPENDS $<TARGET_FILE:link>)

add_custom_target(wast ALL
        DEPENDS ${CONTRACT_WAST_FILE})
add_custom_command(OUTPUT ${CONTRACT_WAST_FILE}
        COMMAND ${S2WASM_PATH}
        -o ${CONTRACT_WAST_FILE}
        -s 16384
        ${CONTRACT_ASSEMBLY_FILE}
        DEPENDS ${CONTRACT_ASSEMBLY_FILE})
