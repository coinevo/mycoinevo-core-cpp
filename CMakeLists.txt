
cmake_minimum_required(VERSION 3.4.1)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

project(TEST)
enable_testing()

#Find openSSL
find_package(OpenSSL COMPONENTS
	crypto REQUIRED
	ssl REQUIRED
)

#Prep ourselves for compiling boost
find_package(Boost COMPONENTS 
    unit_test_framework REQUIRED
    atomic REQUIRED
    chrono REQUIRED
    context REQUIRED
    date_time REQUIRED
    exception REQUIRED
    filesystem REQUIRED
    graph REQUIRED
    iostreams REQUIRED
    math_c99 REQUIRED
    math_c99f REQUIRED
    math_c99l REQUIRED
    math_tr1 REQUIRED
    math_tr1f REQUIRED
    math_tr1l REQUIRED
    prg_exec_monitor REQUIRED
    program_options REQUIRED
    random REQUIRED
    regex REQUIRED
    serialization REQUIRED
    signals REQUIRED
    system REQUIRED
    test_exec_monitor REQUIRED
    thread REQUIRED
    timer REQUIRED
    wave REQUIRED
    wserialization REQUIRED
)

set(MONERO_SRC "contrib/monero-core-custom")

include_directories(${Boost_INCLUDE_DIRS})
include_directories("src")
include_directories(${MONERO_SRC})
include_directories("${MONERO_SRC}/epee/include")
include_directories("${MONERO_SRC}/common")
include_directories("${MONERO_SRC}/vtlogger")
include_directories("${MONERO_SRC}/crypto")
include_directories("${MONERO_SRC}/cryptonote_basic")
include_directories("${MONERO_SRC}/multisig")
include_directories("${MONERO_SRC}/cryptonote_core")
include_directories("${MONERO_SRC}/cryptonote_protocol")
include_directories("${MONERO_SRC}/wallet")
include_directories("${MONERO_SRC}/rpc")
include_directories("${MONERO_SRC}/mnemonics")
include_directories("test")

# keeping test files in a separate source directory 
file(GLOB TEST_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} test/test_*.cpp)

set(
    SRC_FILES
    #
    src/monero_address_utils.hpp
    src/monero_address_utils.cpp
    src/monero_paymentID_utils.hpp
    src/monero_paymentID_utils.cpp
    src/monero_key_image_utils.hpp
    src/monero_key_image_utils.cpp
    src/monero_transfer_utils.hpp
    src/monero_transfer_utils.cpp
    src/monero_fork_rules.hpp
    src/monero_fork_rules.cpp
    src/monero_wallet_utils.hpp
    src/monero_wallet_utils.cpp
    src/tools__ret_vals.hpp
    src/tools__ret_vals.cpp
    #
    ${MONERO_SRC}/cryptonote_basic/cryptonote_basic_impl.cpp
    ${MONERO_SRC}/cryptonote_basic/account.cpp
    ${MONERO_SRC}/cryptonote_basic/cryptonote_format_utils.cpp
    ${MONERO_SRC}/crypto/crypto.cpp
    ${MONERO_SRC}/crypto/hash.c
    ${MONERO_SRC}/crypto/slow-hash.c
    ${MONERO_SRC}/crypto/oaes_lib.c
    ${MONERO_SRC}/crypto/crypto-ops.c
    ${MONERO_SRC}/crypto/crypto-ops-data.c
    ${MONERO_SRC}/crypto/keccak.c
    ${MONERO_SRC}/crypto/random.c
    ${MONERO_SRC}/crypto/aesb.c
    ${MONERO_SRC}/crypto/tree-hash.c
    ${MONERO_SRC}/crypto/hash-extra-blake.c
    ${MONERO_SRC}/crypto/blake256.c
    ${MONERO_SRC}/crypto/hash-extra-groestl.c
    ${MONERO_SRC}/crypto/hash-extra-jh.c
    ${MONERO_SRC}/crypto/hash-extra-skein.c
    ${MONERO_SRC}/crypto/groestl.c
    ${MONERO_SRC}/crypto/jh.c
    ${MONERO_SRC}/crypto/skein.c
    ${MONERO_SRC}/cryptonote_core/cryptonote_tx_utils.cpp
    ${MONERO_SRC}/common/base58.cpp
    ${MONERO_SRC}/common/threadpool.cpp
    ${MONERO_SRC}/epee/src/hex.cpp
    ${MONERO_SRC}/epee/src/string_tools.cpp
    ${MONERO_SRC}/epee/src/memwipe.c
    ${MONERO_SRC}/epee/src/wipeable_string.cpp
    ${MONERO_SRC}/device/device.cpp
    ${MONERO_SRC}/device/device_default.cpp
    ${MONERO_SRC}/ringct/rctOps.cpp
    ${MONERO_SRC}/ringct/rctTypes.cpp
    ${MONERO_SRC}/ringct/rctCryptoOps.c
    ${MONERO_SRC}/ringct/rctSigs.cpp
    ${MONERO_SRC}/ringct/bulletproofs.cc
    ${MONERO_SRC}/mnemonics/electrum-words.cpp
    ${MONERO_SRC}/vtlogger/logger.cpp
)
# needed for slow-hash
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -maes")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes")

#Run through each source
foreach(testSrc ${TEST_SRCS})
    # extract the filename without an extension (NAME_WE)
    get_filename_component(testName ${testSrc} NAME_WE)
    add_executable(
        ${testName}
        ${testSrc}
        ${SRC_FILES}
    )
    target_link_libraries(
        ${testName}
        #
        OpenSSL::SSL
        ${Boost_LIBRARIES} 
    )
    set_target_properties(
        ${testName} PROPERTIES 
        RUNTIME_OUTPUT_DIRECTORY
        ${CMAKE_CURRENT_SOURCE_DIR}/build/products
    )
    add_test(
        NAME ${testName} 
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/build/products 
        COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/build/products/${testName} --catch_system_error=yes
    )
endforeach(testSrc)