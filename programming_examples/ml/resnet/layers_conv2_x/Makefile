##===- Makefile -----------------------------------------------------------===##
# 
# This file licensed under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
#
# Copyright (C) 2024, Advanced Micro Devices, Inc.
# 
##===----------------------------------------------------------------------===##

srcdir := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

include ${srcdir}/../../../makefile-common

mlirFileName = aie

VPATH := ${srcdir}/../../../../aie_kernels/aie2

aie_py_src=resnet.py
use_placed?=0

ifeq (${use_placed}, 1)
aie_py_src=resnet_placed.py
endif

all: build/conv2dk1_i8.o build/conv2dk1_skip_init.o build/conv2dk3.o build/conv2dk1_skip.o build/conv2dk1_ui8.o build/final.xclbin

build/${mlirFileName}.mlir: ${srcdir}/${aie_py_src}
	mkdir -p ${@D}
	python3 $< > $@

build/conv2dk1_i8.o: conv2dk1.cc
	mkdir -p ${@D}
	cd ${@D} && ${PEANO_INSTALL_DIR}/bin/clang++ ${PEANOWRAP2_FLAGS} -DINT8_ACT -c $< -o ${@F}

build/conv2dk3.o: conv2dk3.cc
	mkdir -p ${@D}
	cd ${@D} && ${PEANO_INSTALL_DIR}/bin/clang++ ${PEANOWRAP2_FLAGS} -DUINT8_ACT -c $< -o ${@F}

build/conv2dk1_skip_init.o: conv2dk1_skip_init.cc
	mkdir -p ${@D}
	cd ${@D} && ${PEANO_INSTALL_DIR}/bin/clang++ ${PEANOWRAP2_FLAGS} -DINT8_ACT -c $< -o ${@F}

build/conv2dk1_ui8.o: conv2dk1.cc
	mkdir -p ${@D}
	cd ${@D} && ${PEANO_INSTALL_DIR}/bin/clang++ ${PEANOWRAP2_FLAGS} -DUINT8_ACT -c $< -o ${@F}

build/conv2dk1_skip.o: conv2dk1_skip.cc
	mkdir -p ${@D}
	cd ${@D} && ${PEANO_INSTALL_DIR}/bin/clang++ ${PEANOWRAP2_FLAGS} -DUINT8_ACT -c $< -o ${@F}

build/final.xclbin: build/${mlirFileName}.mlir build/conv2dk1_i8.o build/conv2dk1_skip_init.o build/conv2dk3.o build/conv2dk1_skip.o build/conv2dk1_ui8.o
	mkdir -p ${@D}
	cd ${@D} &&	aiecc.py --alloc-scheme=basic-sequential --aie-generate-xclbin --aie-generate-npu-insts --no-compile-host \
		--no-xchesscc --no-xbridge \
		--xclbin-name=${@F} --npu-insts-name=insts.bin $(<:%=../%)

clean:
	rm -rf build log

run_py: build/final.xclbin
	${powershell} python3 ${srcdir}/test.py -x build/final.xclbin -i build/insts.bin -k MLIR_AIE
