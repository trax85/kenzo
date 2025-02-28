#
# arch/arm64/Makefile
#
# This file is included by the global makefile so that you can add your own
# architecture-specific flags and dependencies.
#
# This file is subject to the terms and conditions of the GNU General Public
# License.  See the file "COPYING" in the main directory of this archive
# for more details.
#
# Copyright (C) 1995-2001 by Russell King

LDFLAGS_vmlinux	:=-p --no-undefined -X
CPPFLAGS_vmlinux.lds = -DTEXT_OFFSET=$(TEXT_OFFSET)
OBJCOPYFLAGS	:=-O binary -R .note -R .note.gnu.build-id -R .comment -S
GZFLAGS		:=-9

KBUILD_DEFCONFIG := defconfig

# Optimize for big.LITTLE configuration
ARM_ARCH_OPT	:= -mcpu=cortex-a57.cortex-a53+crypto+crc \
		   -mtune=cortex-a57.cortex-a53

# If for some reason compiling with a toolchain that supports neither mcpu or
# mtune (in which case, wtf?), don't tune at all (march can break assembly)
KBUILD_CFLAGS	+= $(call cc-option,$(ARM_ARCH_OPT),)

KBUILD_CFLAGS	+= -mgeneral-regs-only
KBUILD_CFLAGS	+= -fno-pic
KBUILD_CFLAGS	+= -fno-asynchronous-unwind-tables

ifeq ($(CONFIG_CPU_BIG_ENDIAN), y)
KBUILD_CPPFLAGS	+= -mbig-endian
AS		+= -EB
LD		+= -EB
else
KBUILD_CPPFLAGS	+= -mlittle-endian
AS		+= -EL
LD		+= -EL
endif

CHECKFLAGS	+= -D__aarch64__

# Default value
head-y		:= arch/arm64/kernel/head.o

# The byte offset of the kernel image in RAM from the start of RAM.
TEXT_OFFSET := 0x00080000

export	TEXT_OFFSET GZFLAGS

core-y		+= arch/arm64/kernel/ arch/arm64/mm/
core-$(CONFIG_XEN)		+= arch/arm64/xen/
core-$(CONFIG_KVM) += arch/arm64/kvm/
core-$(CONFIG_CRYPTO) += arch/arm64/crypto/
libs-y		:= arch/arm64/lib/ $(libs-y)

# Default target when executing plain make
ifeq ($(CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE),y)
KBUILD_IMAGE	:= Image.gz-dtb
else
KBUILD_IMAGE	:= Image.gz
endif

KBUILD_DTBS	:= dtbs

all:	$(KBUILD_IMAGE) $(KBUILD_DTBS)

boot := arch/arm64/boot

Image Image.gz: vmlinux
	$(Q)$(MAKE) $(build)=$(boot) $(boot)/$@

zinstall install: vmlinux
	$(Q)$(MAKE) $(build)=$(boot) $@

%.dtb: scripts
	$(Q)$(MAKE) $(build)=$(boot)/dts $(boot)/dts/$@

.PHONY: dtbs
dtbs: scripts
	$(Q)$(MAKE) $(build)=$(boot)/dts dtbs
	$(Q)$(MAKE) $(build)=$(boot)/dts/qcom dtbs
	@true

PHONY += vdso_install
vdso_install:
	$(Q)$(MAKE) $(build)=arch/arm64/kernel/vdso $@

Image.gz-dtb: vmlinux scripts dtbs
	$(Q)$(MAKE) $(build)=$(boot) $(boot)/$@

# We use MRPROPER_FILES and CLEAN_FILES now
archclean:
	$(Q)$(MAKE) $(clean)=$(boot)

define archhelp
  echo  '* Image.gz      - Compressed kernel image (arch/$(ARCH)/boot/Image.gz)'
  echo  '  Image         - Uncompressed kernel image (arch/$(ARCH)/boot/Image)'
  echo  '* dtbs          - Build device tree blobs for enabled boards'
  echo  '  install       - Install uncompressed kernel'
  echo  '  zinstall      - Install compressed kernel'
  echo  '                  Install using (your) ~/bin/installkernel or'
  echo  '                  (distribution) /sbin/installkernel or'
  echo  '                  install to $$(INSTALL_PATH) and run lilo'
endef
