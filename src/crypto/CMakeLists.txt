# Copyright (c) 2014-2019, The Monero Project
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification, are
# permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this list of
#    conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice, this list
#    of conditions and the following disclaimer in the documentation and/or other
#    materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its contributors may be
#    used to endorse or promote products derived from this software without specific
#    prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL
# THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
# STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF
# THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

set(crypto_sources
  chacha.c
  crypto-ops-data.c
  crypto-ops.c
  crypto.cpp
  hash.c
  hmac-keccak.c
  keccak.c
  groestl.c
  pow_hash/aux_hash.c
  pow_hash/cn_heavy_hash_soft.cpp
  pow_hash/cn_heavy_hash_hard_intel.cpp
  pow_hash/cn_heavy_hash_intel_avx2.cpp
  pow_hash/cn_heavy_hash_hard_arm.cpp
  random.c
  rx-slow-hash.c
  tree-hash.c
  sha512-blocks.c
  sha512.c)

include_directories(${RANDOMXEQ_INCLUDE})

if(ARCH_ID STREQUAL "i386" OR ARCH_ID STREQUAL "x86_64" OR ARCH_ID STREQUAL "x86-64" OR ARCH_ID STREQUAL "amd64")
list(APPEND crypto_sources CryptonightR_template.S)
endif()

if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
	if (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64" OR ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
		set_source_files_properties(pow_hash/cn_heavy_hash_intel_avx2.cpp PROPERTIES COMPILE_FLAGS "-mavx2")
		set_source_files_properties(pow_hash/cn_heavy_hash_hard_intel.cpp PROPERTIES COMPILE_FLAGS "-msse2 -maes")
	elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "aarch64" AND NOT APPLE)
		set_source_files_properties(pow_hash/cn_heavy_hash_hard_arm.cpp PROPERTIES COMPILE_FLAGS "-march=armv8-a+crypto")
	endif()
endif()


set(crypto_headers)

set(crypto_private_headers
  blake256.h
  chacha.h
  crypto-ops.h
  crypto.h
  generic-ops.h
  groestl.h
  groestl_tables.h
  hash-ops.h
  hash.h
  jh.h
  hmac-keccak.h
  initializer.h
  keccak.h
  skein_port.h
  pow_hash/cn_heavy_hash.hpp
  random.h
  sha512.h)

monero_private_headers(cncrypto
  ${crypto_private_headers})
monero_add_library(cncrypto
  ${crypto_sources}
  ${crypto_headers}
  ${crypto_private_headers})
target_link_libraries(cncrypto
  PUBLIC
    epee
    randomx
    ${Boost_SYSTEM_LIBRARY}
    ${SODIUM_LIBRARY}
  PRIVATE
    ${EXTRA_LIBRARIES})

# cheat because cmake and ccache hate each other
set_property(SOURCE CryptonightR_template.S PROPERTY LANGUAGE C)
