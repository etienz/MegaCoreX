# Makefile for AVR Mega-0 (4809), Tiny-0, and Tiny-1 version of Optiboot
# Bill Westfield, 2019
# $Id$
#
# Edit History
# Sep-2019 refactor from the normal AVR Makefile.
# * Copyright 2013-2019 by Bill Westfield.  Part of Optiboot.
# * This software is licensed under version 2 of the Gnu Public Licence.
# * See optiboot.c for details.

HELPTEXT = "\n"
#----------------------------------------------------------------------
#
# program name should not be changed...
PROGRAM    = optiboot_x
MF:= $(MAKEFILE_LIST)

# export symbols to recursive makes (for ISP)
export



# Memory configuration
# BOOTEND_FUSE * 256 must be above Bootloader Program Memory Usage,
# this is 194 bytes at optimization level -O3, so BOOTEND_FUSE = 0x01

BOOTEND_FUSE = 0x02
BOOT_SIZE    = $(BOOTEND_FUSE)00

LDSECTIONS  = -Wl,-section-start=.text=0x00 \
	      -Wl,--section-start=.application=$(BOOT_SIZE) \
	      -Wl,--section-start=.version=0x$(shell echo $$(( $(BOOTEND_FUSE) - 1 )))FE

BOOTFLAGS = -DBOOTEND_FUSE=$(BOOTEND_FUSE) -DBOOT_SIZE=$(BOOT_SIZE)

BAUD_RATE=115200

# Place your avr-gcc tool root here
GCCROOT = "/home/kjb/Public Workspace/arduino-1.8.19/hardware/tools/avr/bin/"#/opt/microchip/xc8/v2.32/avr/bin/
INCLUDE = /opt/microchip/xc8/v2.32/avr/avr/include/
DFP_PATH = /opt/microchip/mplabx/v5.50/packs/Microchip/ATmega_DFP/2.3.126


#
# End of build environment code.


CC         = $(GCCROOT)avr-gcc
RCC        = $(abspath $(CC))
#$(info wildcard ("$(wildcard $(CC))",""))
ifndef PRODUCTION
$(info Using Compiler at: ${RCC})
endif




OPTIMIZE = -Os -fno-split-wide-types -mrelax

# Override is only needed by avr-lib build system.

override CFLAGS  = -g -Wall $(OPTIMIZE)
override LDFLAGS = $(LDSECTIONS) -Wl,--relax -nostartfiles -nostdlib



#AVRDUDE_PATH   = "/home/kjb/Public Workspace/arduino-1.8.19/hardware/tools/avr/"

OBJCOPY        = $(GCCROOT)avr-objcopy
OBJDUMP        = $(GCCROOT)avr-objdump
SIZE           = $(GCCROOT)avr-size
AVRDUDE		   = $(AVRDUDE_PATH)avrdude

AVRDUDE_PROGRAMMER = atmelice_updi
AVRDUDE_PORT = /dev/ttyUSB0
AVRDUDE_BAUD = $(BAUD_RATE)
AVRDUDE_MCU	 = m3208
AVRDUDE_CONF = "/home/kjb/Public Workspace/MegaCoreX/megaavr/avrdude.conf"

#AVRDUDE_NO_VERIFY = -V

AVRDUDE_FLAGS = -C $(AVRDUDE_CONF) -p $(AVRDUDE_MCU) -P $(AVRDUDE_PORT) -b $(AVRDUDE_BAUD) $(AVRDUDE_NO_VERIFY)

include parse_options.mk

.PRECIOUS: %.elf
.PRECIOUS: %.lst
.PRECIOUS: %.map

ifndef PRODUCTION
LISTING= $(OBJDUMP) -S 
else
LISTING= @true
endif

ifeq ($(SKIP_BOOTLOADER_ON_POR), 0)
START_APP_ON_POR=0
else
START_APP_ON_POR=1
endif

ifeq ($(UARTTX), A0)
UART=0
UARTMUX=DEF
endif
ifeq ($(UARTTX), B0)
UART=3
UARTMUX=DEF
endif
ifeq ($(UARTTX), C0)
UART=1
UARTMUX=DEF
endif
ifeq ($(UARTTX), F0)
UART=2
UARTMUX=DEF
endif
ifeq ($(UARTTX), A4)
UART=0
UARTMUX=ALT
endif
ifeq ($(UARTTX), B4)
UART=3
UARTMUX=ALT
endif
ifeq ($(UARTTX), C4)
UART=1
UARTMUX=ALT
endif
ifeq ($(UARTTX), F4)
UART=2
UARTMUX=ALT
endif


#---------------------------------------------------------------------------
# "Chip-level Platform" targets.
# A "Chip-level Platform" compiles for a particular chip, but probably does
# not have "standard" values for things like clock speed, LED pin, etc.
# Makes for chip-level platforms should usually explicitly define their
# options like: "make atmega4809 UARTTX=A4 LED=D0"
#---------------------------------------------------------------------------
#
# Mega0, tiny0, tiny1 don't really have any chip-specific requirements.
#
# Note about fuses:
#  The fuses are defined in the source code.  There are 9!
#  Be sure to use a programmer that will program the fuses from the object file.
#
#---------------------------------------------------------------------------
#



optiboot_%.hex: optiboot_%.elf
	$(OBJCOPY) -j .text -j .data -j .version --set-section-flags .version=alloc,load -O ihex $< $@

%.hex: %.elf
	$(OBJCOPY) -j .text -j .data -j .version --set-section-flags .version=alloc,load -O ihex $< $@	
	$(SIZE) $@

%_fuses.hex: %.elf
	$(OBJCOPY) -j --section=.fuse -O ihex $< $@

%.elf:	optiboot_x_mod.c FORCE
	@echo "Compile elf"
	$(CC) $(CFLAGS) $(CPU_OPTIONS) $(LED_OPTIONS) $(UART_OPTIONS) $(COMMON_OPTIONS) $(POR) $(BOOTFLAGS) $(LDFLAGS) $(PACK_OPT) -Wl,-Map="optiboot_x.map" -mmcu=$(MCU) -o $@ $<

flash: %.hex
	$(AVRDUDE) $(AVRDUDE_FLAGS) -U flash:w:$<.hex:i
	
writebootend: 
	$(AVRDUDE) $(AVRDUDE_FLAGS) -U fuses:w:$(BOOTEND_FUSE):m


#---------------------------------------------------------------------------
# "Board-level Platform" targets.
# A "Board-level Platform" implies a manufactured platform with a particular
# AVR_FREQ, LED, and so on.  Parameters are not particularly changable from
# the "make" command line.
# Most of the board-level platform builds should envoke make recursively
#  appropriate specific options
#---------------------------------------------------------------------------

.PHONY: mega0 install

mega0: MCU=atmega4809
mega0: bootloaders/mega0/$(BAUD_RATE)/Optiboot_mega0_UART$(UART)_$(UARTMUX)_$(BAUD_RATE)_$(LED).hex 
mega0: bootloaders/mega0/$(BAUD_RATE)/Optiboot_mega0_UART$(UART)_$(UARTMUX)_$(BAUD_RATE)_$(LED)_fuses.hex 
mega0: bootloaders/mega0/$(BAUD_RATE)/Optiboot_mega0_UART$(UART)_$(UARTMUX)_$(BAUD_RATE)_$(LED).lst 
#$(TARGET).lst

install: mega0
	avrdude -p m3208 -c atmelice_updi -U flash:w:bootloaders/mega0/$(BAUD_RATE)/Optiboot_mega0_UART$(UART)_$(UARTMUX)_$(BAUD_RATE)_$(LED).hex:i


#---------------------------------------------------------------------------
#
# Generic build instructions
#

FORCE:

#windows "rm" is dumb and objects to wildcards that don't exist
clean:
	@touch  __temp_.o __temp_.elf __temp_.lst __temp_.map
	@touch  __temp_.sym __temp_.lss __temp_.eep __temp_.srec
	@touch __temp_.bin __temp_.hex __temp_.tmp.sh
	rm -rf *.o *.elf *.lst *.map *.sym *.lss *.eep *.srec *.bin *.hex *.tmp.sh

clean_asm:
	rm -rf *.lst

%.lst: %.elf FORCE
	$(OBJDUMP) -h -S $< > $@

%.srec: %.elf FORCE
	$(OBJCOPY) -j .text -j .data -j .version --set-section-flags .version=alloc,load -O srec $< $@

%.bin: %.elf FORCE
	$(OBJCOPY) -j .text -j .data -j .version --set-section-flags .version=alloc,load -O binary $< $@

