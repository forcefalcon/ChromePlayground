# Copyright (c) 2012 The Native Client Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

#
# GNU Make based build file.  For details on GNU Make see:
#   http://www.gnu.org/software/make/manual/make.html
#

#
# Get pepper directory for toolchain and includes.
#
# If NACL_SDK_ROOT is not set, then assume it can be found a two directories up,
# from the default example directory location.
#
THIS_MAKEFILE:=$(abspath $(lastword $(MAKEFILE_LIST)))
NACL_SDK_ROOT?=$(abspath $(dir $(THIS_MAKEFILE))../..)
CHROME_PATH?=Undefined

#
# Defaults
#
NACL_WARNINGS:=-Wno-long-long -Wall -Wswitch-enum -Werror -pedantic
NACL_CCFLAGS:=-O0 -g -pthread $(NACL_WARNINGS)
NACL_CXXFLAGS:= -O0 -g -pthread -std=gnu++98 $(NACL_WARNINGS)
NACL_LDFLAGS:=-g -pthread -lppapi_cpp -lppapi 

#
# Project Settings
#
VALID_TOOLCHAINS:=glibc
TOOLCHAIN?=glibc

DLOPEN_CXX:=dlopen.cc
DLOPEN_CXXFLAGS:=$(NACL_CXXFLAGS)
DLOPEN_LDFLAGS:=-g -ldl -lppapi_cpp -lppapi
LIBEIGHTBALL_CXX:=eightball.cc
LIBEIGHTBALL_CXXFLAGS:=$(NACL_CXXFLAGS) -fPIC
LIBEIGHTBALL_LDFLAGS:=-g -ldl -lppapi_cpp -lppapi -shared


#
# Default target
#
all: glibc/dlopen.nmf

#
# Alias for standard commands
#
CP:=python $(NACL_SDK_ROOT)/tools/oshelpers.py cp
MKDIR:=python $(NACL_SDK_ROOT)/tools/oshelpers.py mkdir
MV:=python $(NACL_SDK_ROOT)/tools/oshelpers.py mv


#
# Verify we selected a valid toolchain for this example
#
ifeq (,$(findstring $(TOOLCHAIN),$(VALID_TOOLCHAINS)))
$(warning Availbile choices are: $(VALID_TOOLCHAINS))
$(error Can not use TOOLCHAIN=$(TOOLCHAIN) on this example.)
endif


#
# Compute path to requested NaCl Toolchain
#
OSNAME:=$(shell python $(NACL_SDK_ROOT)/tools/getos.py)
TC_PATH:=$(abspath $(NACL_SDK_ROOT)/toolchain)


#
# Verify we have a valid NACL_SDK_ROOT by looking for the toolchain directory
#
ifeq (,$(wildcard $(TC_PATH)))
$(warning No valid NACL_SDK_ROOT at $(NACL_SDK_ROOT))
ifeq ($(origin NACL_SDK_ROOT), 'file')
$(error Override the default value via enviornment variable, or command-line.)
else
$(error Fix the NACL_SDK_ROOT specified in the environment or command-line.)
endif
endif


#
# Disable DOS PATH warning when using Cygwin based NaCl tools on Windows
#
CYGWIN ?= nodosfilewarning
export CYGWIN


#
# Defaults for TOOLS
#

GLIBC_CC?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-gcc -c
GLIBC_CXX?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-g++ -c
GLIBC_LINK?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-g++ -Wl,-as-needed
GLIBC_DUMP?=$(TC_PATH)/$(OSNAME)_x86_glibc/x86_64-nacl/bin/objdump
GLIBC_PATHS:=-L $(TC_PATH)/$(OSNAME)_x86_glibc/x86_64-nacl/lib32
GLIBC_PATHS+=-L $(TC_PATH)/$(OSNAME)_x86_glibc/x86_64-nacl/lib



#
# NMF Manifiest generation
#
# Use the python script create_nmf to scan the binaries for dependencies using
# objdump.  Pass in the (-L) paths to the default library toolchains so that we
# can find those libraries and have it automatically copy the files (-s) to
# the target directory for us.
NMF:=python $(NACL_SDK_ROOT)/tools/create_nmf.py


#
# Verify we can find the Chrome executable if we need to launch it.
#
.PHONY: CHECK_FOR_CHROME
CHECK_FOR_CHROME:
ifeq (,$(wildcard $(CHROME_PATH)))
	$(warning No valid Chrome found at CHROME_PATH=$(CHROME_PATH))
	$(error Set CHROME_PATH via an environment variable, or command-line.)
else
	$(warning Using chrome at: $(CHROME_PATH))
endif


#
# Rules for glibc toolchain
#
glibc:
	$(MKDIR) glibc

GLIBC_DLOPEN_x86_32_CXX_O:=$(patsubst %.cc, glibc/%_x86_32.o,$(DLOPEN_CXX))
$(GLIBC_DLOPEN_x86_32_CXX_O) : glibc/%_x86_32.o : %.cc $(THIS_MAKE) | glibc
	$(GLIBC_CXX) -o $@ $< -m32 $(DLOPEN_CXXFLAGS) -DTCNAME=glibc

glibc/dlopen_x86_32.nexe : $(GLIBC_DLOPEN_x86_32_CXX_O)
	$(GLIBC_LINK) -o $@ $^ -m32 $(DLOPEN_LDFLAGS)
GLIBC_NMF+=glibc/dlopen_x86_32.nexe 

GLIBC_DLOPEN_x86_64_CXX_O:=$(patsubst %.cc, glibc/%_x86_64.o,$(DLOPEN_CXX))
$(GLIBC_DLOPEN_x86_64_CXX_O) : glibc/%_x86_64.o : %.cc $(THIS_MAKE) | glibc
	$(GLIBC_CXX) -o $@ $< -m64 $(DLOPEN_CXXFLAGS) -DTCNAME=glibc

glibc/dlopen_x86_64.nexe : $(GLIBC_DLOPEN_x86_64_CXX_O)
	$(GLIBC_LINK) -o $@ $^ -m64 $(DLOPEN_LDFLAGS)
GLIBC_NMF+=glibc/dlopen_x86_64.nexe 

GLIBC_LIBEIGHTBALL_x86_32_CXX_O:=$(patsubst %.cc, glibc/%_x86_32.o,$(LIBEIGHTBALL_CXX))
$(GLIBC_LIBEIGHTBALL_x86_32_CXX_O) : glibc/%_x86_32.o : %.cc $(THIS_MAKE) | glibc
	$(GLIBC_CXX) -o $@ $< -m32 $(LIBEIGHTBALL_CXXFLAGS) -DTCNAME=glibc

glibc/libeightball_x86_32.so : $(GLIBC_LIBEIGHTBALL_x86_32_CXX_O)
	$(GLIBC_LINK) -o $@ $^ -m32 -shared $(LIBEIGHTBALL_LDFLAGS)
GLIBC_REMAP+= -n libeightball_x86_32.so,libeightball.so
GLIBC_NMF+=glibc/libeightball_x86_32.so

GLIBC_LIBEIGHTBALL_x86_64_CXX_O:=$(patsubst %.cc, glibc/%_x86_64.o,$(LIBEIGHTBALL_CXX))
$(GLIBC_LIBEIGHTBALL_x86_64_CXX_O) : glibc/%_x86_64.o : %.cc $(THIS_MAKE) | glibc
	$(GLIBC_CXX) -o $@ $< -m64 $(LIBEIGHTBALL_CXXFLAGS) -DTCNAME=glibc

glibc/libeightball_x86_64.so : $(GLIBC_LIBEIGHTBALL_x86_64_CXX_O)
	$(GLIBC_LINK) -o $@ $^ -m64 -shared $(LIBEIGHTBALL_LDFLAGS)
GLIBC_REMAP+= -n libeightball_x86_64.so,libeightball.so
GLIBC_NMF+=glibc/libeightball_x86_64.so

glibc/dlopen.nmf : $(GLIBC_NMF)
	$(NMF) -D $(GLIBC_DUMP) -o $@ $(GLIBC_PATHS) $^ -t glibc -s glibc $(GLIBC_REMAP)





RUN: all
	python ../httpd.py

LAUNCH_NEXE: CHECK_FOR_CHROME all
	$(CHROME_PATH) $(NEXE_ARGS) "localhost:5103/$(PROJECT).html?tool=$(TOOLCHAIN)"


