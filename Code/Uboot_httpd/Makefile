#
# (C) Copyright 2000-2004
# Wolfgang Denk, DENX Software Engineering, wd@denx.de.
#
# See file CREDITS for list of people who contributed to this
# project.
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License as
# published by the Free Software Foundation; either version 2 of
# the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston,
# MA 02111-1307 USA
#



HOSTARCH := $(shell uname -m | \
	sed -e s/i.86/i386/ \
	    -e s/sun4u/sparc64/ \
	    -e s/arm.*/arm/ \
	    -e s/sa110/arm/ \
	    -e s/powerpc/ppc/ \
	    -e s/macppc/ppc/)

HOSTOS := $(shell uname -s | tr A-Z a-z | \
	    sed -e 's/\(cygwin\).*/cygwin/')

export	HOSTARCH

# Deal with colliding definitions from tcsh etc.
VENDOR=

#########################################################################

TOPDIR	:= $(shell if [ "$$PWD" != "" ]; then echo $$PWD; else pwd; fi)
export	TOPDIR

CONFIG_CROSS_COMPILER_PATH ?= /opt/buildroot-gcc342/bin

ifeq (include/config.mk,$(wildcard include/config.mk))
# load ARCH, BOARD, and CPU configuration
include include/config.mk
export	ARCH CPU BOARD VENDOR SOC
# load other configuration
include $(TOPDIR)/config.mk

ifndef CROSS_COMPILE
ifeq ($(HOSTARCH),ppc)
CROSS_COMPILE =
else
ifeq ($(ARCH),ppc)
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/ppc_8xx-
endif
ifeq ($(ARCH),arm)
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/arm-linux-
endif
ifeq ($(ARCH),i386)
ifeq ($(HOSTARCH),i386)
CROSS_COMPILE =
else
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/i386-linux-
endif
endif
ifeq ($(ARCH),mips)
# CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/mips_4KCle-
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/mipsel-linux-
endif
ifeq ($(ARCH),nios)
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/nios-elf-
endif
ifeq ($(ARCH),nios2)
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/nios2-elf-
endif
ifeq ($(ARCH),m68k)
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/m68k-elf-
endif
ifeq ($(ARCH),microblaze)
CROSS_COMPILE = $(CONFIG_CROSS_COMPILER_PATH)/mb-
endif
endif
endif

ifeq ($(ON_BOARD_DDR1),y)
DRAM_TYPE=DDR
endif
ifeq ($(ON_BOARD_DDR2),y)
DRAM_TYPE=DDR
endif
ifeq ($(ON_BOARD_SDR),y)
DRAM_TYPE=SDR
endif

DRAM_SIZE=2
ifeq ($(ON_BOARD_64M_DRAM_COMPONENT),y)
DRAM_SIZE=8
endif
ifeq ($(ON_BOARD_128M_DRAM_COMPONENT),y)
DRAM_SIZE=16
endif
ifeq ($(ON_BOARD_256M_DRAM_COMPONENT),y)
DRAM_SIZE=32
endif
ifeq ($(ON_BOARD_512M_DRAM_COMPONENT),y)
DRAM_SIZE=64
endif
ifeq ($(ON_BOARD_1024M_DRAM_COMPONENT),y)
DRAM_SIZE=128
endif

# DRAM_WIDTH: SDR(DDR2)
# 0:16(8)
# 1:32(16)
ifeq ($(ON_BOARD_DDR_WIDTH_16),y)
DRAM_WIDTH=32
else
DRAM_WIDTH=16
endif

ifeq ($(ON_BOARD_32BIT_DRAM_BUS),y)
DRAM_TOTAL_WIDTH=32
else
DRAM_TOTAL_WIDTH=16
endif

#default value for syscfg0/syscfg1
ifeq ($(DRAM_CFG0),)
DRAM_SYSCFG0=0xFF
else
DRAM_SYSCFG0=$(DRAM_CFG0)
endif

ifeq ($(DRAM_CFG1),)
DRAM_SYSCFG1=0xFF
else
DRAM_SYSCFG1=$(DRAM_CFG1)
endif

ifeq ($(CPUPLLCFG),)
CPU_PLL_CFG=0xFF
else
CPU_PLL_CFG=$(CPUPLLCFG)
endif
export	CROSS_COMPILE

#########################################################################
# U-Boot objects....order is important (i.e. start must be first)

OBJS  = cpu/$(CPU)/start.o
ifeq ($(CPU),i386)
OBJS += cpu/$(CPU)/start16.o
OBJS += cpu/$(CPU)/reset.o
endif
ifeq ($(CPU),ppc4xx)
OBJS += cpu/$(CPU)/resetvec.o
endif
ifeq ($(CPU),mpc85xx)
OBJS += cpu/$(CPU)/resetvec.o
endif

LIBS  = lib_generic/libgeneric.a
LIBS += board/$(BOARDDIR)/lib$(BOARD).a
LIBS += cpu/$(CPU)/lib$(CPU).a
ifdef SOC
LIBS += cpu/$(CPU)/$(SOC)/lib$(SOC).a
endif
LIBS += lib_$(ARCH)/lib$(ARCH).a
LIBS += net/libnet.a

LIBS += rtc/librtc.a
#LIBS += dtt/libdtt.a
LIBS += drivers/libdrivers.a

ifdef RALINK_USB
LIBS += fs/fat/libfat.a
LIBS += disk/libdisk.a
endif

#LIBS += post/libpost.a post/cpu/libcpu.a
LIBS += common/libcommon.a
#add by aggress for RFDemo
LIBS += httpd/libhttpd.a
.PHONY : $(LIBS)

# Add GCC lib
PLATFORM_LIBS += -L $(shell dirname `$(CC) $(CFLAGS)  -print-libgcc-file-name`)

# The "tools" are needed early, so put this first
# Don't include stuff already done in $(LIBS)
ifneq ($(CFG_ENV_IS), IN_FLASH)
SUBDIRS	= tools
else
SUBDIRS	=
endif
.PHONY : $(SUBDIRS)

#########################################################################
#########################################################################

ALL = u-boot.srec uboot.bin System.map
ifneq ($(CFG_ENV_IS), IN_FLASH)
ALL += uboot.img
endif

all:		clean $(ALL)

u-boot.hex:	u-boot
		$(OBJCOPY) ${OBJCFLAGS} -O ihex $< $@


u-boot.srec:	u-boot
		$(OBJCOPY) ${OBJCFLAGS} -O srec $< $@

uboot.bin:	u-boot
		$(OBJCOPY) ${OBJCFLAGS} -O binary $< $@

uboot.img:	uboot.bin
ifeq ($(CFG_ENV_IS), IN_SPI)
		./tools/mkimage -A $(ARCH) -T standalone -C none \
		-a $(TEXT_BASE) -e $(shell readelf -h u-boot | grep "Entry" | awk '{print $$4}') \
		-n "$(shell echo $(CFG_ENV_IS) | sed -e 's/IN_//') Flash Image" \
		-r $(DRAM_TYPE) -s $(DRAM_TOTAL_WIDTH) -t $(DRAM_SIZE) -u $(DRAM_WIDTH) \
		-y $(DRAM_SYSCFG0) -z $(DRAM_SYSCFG1) -w $(CPU_PLL_CFG) -d $< $@
endif
#ifeq ($(CFG_ENV_IS), IN_NAND)
#		$(MAKE) -C stage1
#endif
ifeq ($(CFG_ENV_IS), IN_NAND)
		./tools/mkimage -A $(ARCH) -T standalone -C none \
		-a $(TEXT_BASE) -e $(shell readelf -h u-boot | grep "Entry" | awk '{print $$4}') \
		-n "$(shell echo $(CFG_ENV_IS) | sed -e 's/IN_//') Flash Image" \
		-r $(DRAM_TYPE) -s $(DRAM_TOTAL_WIDTH) -t $(DRAM_SIZE) -u $(DRAM_WIDTH) \
		-y $(DRAM_SYSCFG0) -z $(DRAM_SYSCFG1) -d $< $@
endif
		@echo ""
		@echo "===============<<IMPORTANT>>=================="
		@echo "Notes: Uboot firmware is uboot.img NOT uboot.bin"
		@echo "================================================"
		@echo ""

u-boot.dis:	u-boot
		$(OBJDUMP) -d $< > $@

#u-boot:		depend $(SUBDIRS) $(OBJS) $(LIBS) $(LDSCRIPT)
#		UNDEF_SYM=`$(OBJDUMP) -x $(LIBS) |sed  -n -e 's/.*\(__u_boot_cmd_.*\)/-u\1/p'|sort|uniq`;\
#		$(LD) $(LDFLAGS) $$UNDEF_SYM $(OBJS) \
#			--start-group $(LIBS) --end-group -L $(shell dirname) $(CC) $(CFLAGS) -print-libgcc-file-name -lgcc \
#			-Map u-boot.map -o u-boot 
			
u-boot:		depend $(SUBDIRS) $(OBJS) $(LIBS) $(LDSCRIPT)
		UNDEF_SYM=`$(OBJDUMP) -x $(LIBS) |sed  -n -e 's/.*\(__u_boot_cmd_.*\)/-u\1/p'|sort|uniq`;\
		$(LD) $(LDFLAGS) $$UNDEF_SYM $(OBJS) \
			--start-group $(LIBS) --end-group $(PLATFORM_LIBS) \
			-Map u-boot.map -o u-boot
			
show_path:
	echo $(OBJDUMP)

$(LIBS):
		$(MAKE) -C `dirname $@`
		echo $(MAKE) -C `dirname $@`

$(SUBDIRS):
		$(MAKE) -C $@ all
		echo $(MAKE) -C $@ all

gdbtools:
		$(MAKE) -C tools/gdb || exit 1

depend dep:
		@for dir in $(SUBDIRS) ; do $(MAKE) -C $$dir .depend ; done

tags:
		ctags -w `find $(SUBDIRS) include \
				lib_generic board/$(BOARDDIR) cpu/$(CPU) lib_$(ARCH) \
				fs/cramfs fs/fat fs/fdos fs/jffs2 \
				net disk rtc dtt drivers drivers/sk98lin common \
			\( -name CVS -prune \) -o \( -name '*.[ch]' -print \)`

etags:
		etags -a `find $(SUBDIRS) include \
				lib_generic board/$(BOARDDIR) cpu/$(CPU) lib_$(ARCH) \
				fs/cramfs fs/fat fs/fdos fs/jffs2 \
				net disk rtc dtt drivers drivers/sk98lin common \
			\( -name CVS -prune \) -o \( -name '*.[ch]' -print \)`

System.map:	u-boot
		@$(NM) $< | \
		grep -v '\(compiled\)\|\(\.o$$\)\|\( [aUw] \)\|\(\.\.ng$$\)\|\(LASH[RL]DI\)' | \
		sort > System.map

#########################################################################
else
all install u-boot u-boot.srec depend dep:
	@echo "System not configured - see README" >&2
	@ exit 1
endif

#########################################################################

unconfig:
	@rm -f board/*/config.tmp .config .tmp_config System.map autoconf.h

#########################################################################

clean:
	find . -type f \
		\( -name 'core' -o -name '*.bak' -o -name '*~' \
		-o -name '*.o'  -o -name '*.a'  \) -print \
		| xargs rm -f
	find ./ -name '.depend' -print | xargs rm -f

	rm -f examples/hello_world examples/timer \
	      examples/eepro100_eeprom examples/sched \
	      examples/mem_to_mem_idma2intr examples/82559_eeprom
	rm -f tools/img2srec tools/mkimage tools/envcrc tools/gen_eth_addr
	rm -f tools/mpc86x_clk tools/ncb
	rm -f tools/easylogo/easylogo tools/bmp_logo
	rm -f tools/gdb/astest tools/gdb/gdbcont tools/gdb/gdbsend
	rm -f tools/env/fw_printenv tools/env/fw_setenv
	rm -f board/cray/L1/bootscript.c board/cray/L1/bootscript.image
	rm -f board/trab/trab_fkt
	rm -f stage1/stage2.bin stage1/stage1n2.elf stage1/stage1n2.map
	rm -f ./uboot.bin ./uboot.img ./u-boot ./u-boot.* ./httpd/fsdata.c
	rm -f scripts/lxdialog/lxdialog

clobber:	clean
	find . -type f \( -name .depend \
		-o -name '*.srec' -o -name '*.bin' -o -name u-boot.img \) \
		-print0 \
		| xargs -0 rm -f
	rm -f $(OBJS) *.bak tags TAGS
	rm -fr *.*~
	rm -f u-boot u-boot.map u-boot.hex $(ALL)

mrproper \
distclean:	clobber unconfig

backup:
	F=`basename $(TOPDIR)` ; cd .. ; \
	gtar --force-local -zcvf `date "+$$F-%Y-%m-%d-%T.tar.gz"` $$F

menuconfig: clean config.in
	$(MAKE) -C scripts/lxdialog all
	$(CONFIG_SHELL) scripts/Menuconfig config.in

#########################################################################
