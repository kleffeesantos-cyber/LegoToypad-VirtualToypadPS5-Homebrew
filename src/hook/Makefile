# LEGO Dimensions Virtual Toy Pad - PS4 GoldHEN plugin (ldtoypad.prx)
#
# TEMPLATE: the toolchain half (target triple, link script, create-fself flags) follows the
# OpenOrbis / GoldHEN plugin recipe. If your other GoldHEN plugin (ps4-skylanders-portal) builds
# in your CI, copy its CFLAGS/LDFLAGS/create-fself lines over the ones marked TOOLCHAIN below.
#
# Needs: OO_PS4_TOOLCHAIN (OpenOrbis toolchain root). Run `make` inside src/ps4.

OO      ?= $(OO_PS4_TOOLCHAIN)
OUT     := ../../build/ps4
TARGET  := $(OUT)/ldtoypad.prx

CC      := clang
LD      := ld.lld

# ---- TOOLCHAIN ------------------------------------------------------------------------------
CFLAGS  := --target=x86_64-pc-freebsd12-elf -fPIC -funwind-tables -c -O2 \
           -isysroot $(OO) -isystem $(OO)/include -isystem $(OO)/include/c++/v1 \
           -fno-stack-protector -mstackrealign -Wall -Wno-unused-parameter
LDFLAGS := -m elf_x86_64 -pie --script $(OO)/link.x --eh-frame-hdr -L$(OO)/lib -lc -lkernel
FSELF   := $(OO)/bin/linux/create-fself
# ---------------------------------------------------------------------------------------------

# hooks.c defines its own memcpy/memset/sqrtf for the freestanding PS5 blob: rename those
# definitions so they do not clash with libc (struct copies keep calling the real memcpy)
HOOKDEF := -Dmemcpy=ld_memcpy_ -Dmemset=ld_memset_ -Dsqrtf=ld_sqrtf_

OBJS := $(OUT)/plugin.o $(OUT)/hooks.o $(OUT)/menu.o $(OUT)/ldproto.o $(OUT)/webremote_ps4.o

all: $(TARGET)

$(OUT):
	mkdir -p $(OUT)

$(OUT)/plugin.o: plugin.c ../hook/ldhook.h ../injector/offsets.h | $(OUT)
	$(CC) $(CFLAGS) -I../hook $< -o $@

$(OUT)/hooks.o: ../hook/hooks.c ../hook/ldhook.h | $(OUT)
	$(CC) $(CFLAGS) $(HOOKDEF) -fvisibility=hidden -I../core $< -o $@

$(OUT)/menu.o: ../hook/menu.c ../hook/ldhook.h | $(OUT)
	$(CC) $(CFLAGS) -fvisibility=hidden -I../core $< -o $@

$(OUT)/ldproto.o: ../core/ldproto.c ../core/ldproto.h | $(OUT)
	$(CC) $(CFLAGS) -fvisibility=hidden -I../core $< -o $@

# webremote.c is the PS5 file, unchanged: a generated copy swaps pthread_* for the ldp_* shim
$(OUT)/webremote_ps4.c: ../injector/webremote.c | $(OUT)
	sed -e 's|#include <pthread.h>|#include "ldpthread.h"|' \
	    -e 's/\bpthread_/ldp_/g' -e 's/\bPTHREAD_/LDP_/g' $< > $@

$(OUT)/webremote_ps4.o: $(OUT)/webremote_ps4.c ../injector/vtp_api.h | $(OUT)
	$(CC) $(CFLAGS) -Ishim -I../injector -I../hook $< -o $@

$(OUT)/ldtoypad.oelf: $(OBJS)
	$(LD) $(OBJS) $(LDFLAGS) -o $@

$(TARGET): $(OUT)/ldtoypad.oelf
	$(FSELF) -in=$< -out=$(OUT)/ldtoypad.out --lib=$@ --paid 0x3800000000000011

clean:
	rm -rf $(OUT)
