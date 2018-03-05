cmake_minimum_required(VERSION 3.9)
project(hello)

set(EOSIO_INSTALL_DIR $ENV{HOME}/eos/build/install)
set(BOOST_INSTALL_DIR $ENV{HOME}/opt/boost_1_66_0)
set(WASM_INSTALL_DIR $ENV{HOME}/opt/wasm)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_C_COMPILER ${WASM_INSTALL_DIR}/bin/clang)
set(CMAKE_CXX_COMPILER ${WASM_INSTALL_DIR}/bin/clang++)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -emit-llvm")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --target=wasm32")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffreestanding")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdinc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -nostdlib")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-threadsafe-statics")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")

include_directories(include)
include_directories(${EOSIO_INSTALL_DIR}/include)
include_directories(${EOSIO_INSTALL_DIR}/include/libc++/upstream/include)
include_directories(${EOSIO_INSTALL_DIR}/include/musl/upstream/include)
include_directories(${BOOST_INSTALL_DIR}/include)

set(CPP_FILES
        src/${CMAKE_PROJECT_NAME}.cpp)

set(HPP_FILES
        include/${CMAKE_PROJECT_NAME}.hpp)

add_library(objects OBJECT
        ${CPP_FILES}
        ${HPP_FILES})

set(CONTRACT_BYTECODE_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.bc)
set(CONTRACT_ASSEMBLY_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.s)
set(CONTRACT_WAST_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.wast)
set(CONTRACT_ABI_FILE ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}.abi)

add_custom_target(link
        DEPENDS ${CONTRACT_BYTECODE_FILE})
add_custom_command(OUTPUT ${CONTRACT_BYTECODE_FILE}
        COMMAND ${WASM_INSTALL_DIR}/bin/llvm-link
        -o ${CONTRACT_BYTECODE_FILE} $<TARGET_OBJECTS:objects>
        DEPENDS $<TARGET_OBJECTS:objects>)

add_custom_target(assembly
        DEPENDS ${CONTRACT_ASSEMBLY_FILE})
add_custom_command(OUTPUT ${CONTRACT_ASSEMBLY_FILE}
        COMMAND ${WASM_INSTALL_DIR}/bin/llc
        --asm-verbose=false
        -o ${CONTRACT_ASSEMBLY_FILE} ${CONTRACT_BYTECODE_FILE}
        DEPENDS ${CONTRACT_BYTECODE_FILE})

add_custom_target(wast ALL
        DEPENDS ${CONTRACT_WAST_FILE})
add_custom_command(OUTPUT ${CONTRACT_WAST_FILE}
        COMMAND ${EOSIO_INSTALL_DIR}/bin/eosio-s2wasm
        -o ${CONTRACT_WAST_FILE}
        -s 16384
        ${CONTRACT_ASSEMBLY_FILE}
        DEPENDS ${CONTRACT_ASSEMBLY_FILE})

foreach(FILE IN LISTS HPP_FILES)
    get_filename_component(FILE_ABSOLUTE ${FILE} ABSOLUTE)
    list(APPEND HPP_FILES_ABSOLUTE ${FILE_ABSOLUTE})
endforeach()

add_custom_target(abi ALL
        DEPENDS ${CONTRACT_ABI_FILE})
add_custom_command(OUTPUT ${CONTRACT_ABI_FILE}
        COMMAND ${EOSIO_INSTALL_DIR}/bin/eosio-abigen
        -extra-arg=-c
        -extra-arg=--std=c++14
        -extra-arg=--target=wasm32
        -extra-arg=-nostdinc
        -extra-arg=-nostdinc++
        -extra-arg=-DABIGEN
        -extra-arg=-I${EOSIO_INSTALL_DIR}/include/libc++/upstream/include
        -extra-arg=-I${EOSIO_INSTALL_DIR}/include/musl/upstream/include
        -extra-arg=-I${EOSIO_INSTALL_DIR}/include
        -extra-arg=-I${BOOST_INSTALL_DIR}/include
        -extra-arg=-I${CMAKE_HOME_DIRECTORY}/include
        -extra-arg=-fparse-all-comments
        -destination-file=${CONTRACT_ABI_FILE}
        -verbose=0
        ${HPP_FILES_ABSOLUTE} --
        DEPENDS ${HPP_FILES})
