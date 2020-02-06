
cmake_minimum_required(VERSION 3.4.1)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

project(TEST)
enable_testing()

#Prep ourselves for compiling boost
find_package(Boost COMPONENTS 
    unit_test_framework REQUIRED
    system REQUIRED
    thread REQUIRED
)

set(COINEVO_SRC "contrib/coinevo-core-custom")

include_directories(${Boost_INCLUDE_DIRS})
include_directories("src")
include_directories(${COINEVO_SRC})
include_directories("${COINEVO_SRC}/epee/include")
include_directories("${COINEVO_SRC}/common")
include_directories("${COINEVO_SRC}/vtlogger")
include_directories("${COINEVO_SRC}/crypto")
include_directories("${COINEVO_SRC}/cryptonote_basic")
include_directories("${COINEVO_SRC}/multisig")
include_directories("${COINEVO_SRC}/cryptonote_core")
include_directories("${COINEVO_SRC}/cryptonote_protocol")
include_directories("${COINEVO_SRC}/wallet")
include_directories("${COINEVO_SRC}/rpc")
include_directories("${COINEVO_SRC}/mnemonics")
include_directories("${COINEVO_SRC}/contrib/libsodium/include") # support sodium/… paths
include_directories("${COINEVO_SRC}/contrib/libsodium/include/sodium")
include_directories("test")

# keeping test files in a separate source directory 
file(GLOB TEST_SRCS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} test/test_*.cpp)

set(
    SRC_FILES
    #
    src/coinevo_address_utils.hpp
    src/coinevo_address_utils.cpp
    src/coinevo_paymentID_utils.hpp
    src/coinevo_paymentID_utils.cpp
    src/coinevo_key_image_utils.hpp
    src/coinevo_key_image_utils.cpp
    src/coinevo_fee_utils.hpp
    src/coinevo_fee_utils.cpp
    src/coinevo_transfer_utils.hpp
    src/coinevo_transfer_utils.cpp
    src/coinevo_send_routine.hpp
    src/coinevo_send_routine.cpp
    src/coinevo_fork_rules.hpp
    src/coinevo_fork_rules.cpp
    src/coinevo_wallet_utils.hpp
    src/coinevo_wallet_utils.cpp
    src/serial_bridge_index.hpp
    src/serial_bridge_index.cpp
    src/serial_bridge_utils.hpp
    src/serial_bridge_utils.cpp
    src/tools__ret_vals.hpp
    src/tools__ret_vals.cpp
    #
    ${COINEVO_SRC}/cryptonote_basic/cryptonote_basic_impl.cpp
    ${COINEVO_SRC}/cryptonote_basic/account.cpp
    ${COINEVO_SRC}/cryptonote_basic/cryptonote_format_utils.cpp
    ${COINEVO_SRC}/crypto/crypto.cpp
    ${COINEVO_SRC}/crypto/hash.c
    ${COINEVO_SRC}/crypto/slow-hash-dummied.cpp
    ${COINEVO_SRC}/crypto/oaes_lib.c
    ${COINEVO_SRC}/crypto/crypto-ops.c
    ${COINEVO_SRC}/crypto/crypto-ops-data.c
    ${COINEVO_SRC}/crypto/keccak.c
    ${COINEVO_SRC}/crypto/chacha.c
    ${COINEVO_SRC}/crypto/random.c
    ${COINEVO_SRC}/crypto/aesb.c
    ${COINEVO_SRC}/crypto/tree-hash.c
    ${COINEVO_SRC}/crypto/hash-extra-blake.c
    ${COINEVO_SRC}/crypto/blake256.c
    ${COINEVO_SRC}/crypto/hash-extra-groestl.c
    ${COINEVO_SRC}/crypto/hash-extra-jh.c
    ${COINEVO_SRC}/crypto/hash-extra-skein.c
    ${COINEVO_SRC}/crypto/groestl.c
    ${COINEVO_SRC}/crypto/jh.c
    ${COINEVO_SRC}/crypto/skein.c
    ${COINEVO_SRC}/cryptonote_core/cryptonote_tx_utils.cpp
    ${COINEVO_SRC}/common/base58.cpp
    ${COINEVO_SRC}/common/threadpool.cpp
    ${COINEVO_SRC}/common/aligned.c
    ${COINEVO_SRC}/common/util.cpp
    ${COINEVO_SRC}/epee/src/hex.cpp
    ${COINEVO_SRC}/epee/src/string_tools.cpp
    ${COINEVO_SRC}/epee/src/memwipe.c
    ${COINEVO_SRC}/epee/src/mlocker.cpp
    ${COINEVO_SRC}/epee/src/wipeable_string.cpp
    ${COINEVO_SRC}/device/device.cpp
    ${COINEVO_SRC}/device/device_default.cpp
    ${COINEVO_SRC}/ringct/rctOps.cpp
    ${COINEVO_SRC}/ringct/rctTypes.cpp
    ${COINEVO_SRC}/ringct/rctCryptoOps.c
    ${COINEVO_SRC}/ringct/rctSigs.cpp
    ${COINEVO_SRC}/ringct/bulletproofs.cc
    ${COINEVO_SRC}/ringct/multiexp.cc
    ${COINEVO_SRC}/mnemonics/electrum-words.cpp
    ${COINEVO_SRC}/vtlogger/logger.cpp
    ${COINEVO_SRC}/contrib/libsodium/src/crypto_verify/verify.c
)
# needed for slow-hash
add_compile_options(-maes)

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
