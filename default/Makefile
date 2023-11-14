###############################################################################
# Makefile for the project Controller Tester
###############################################################################

## General Flags
PROJECT = Smokey and the Bandit
GAME = smokeyAndTheBandit
MCU = atmega644
TARGET = $(GAME).elf
CC = avr-gcc
PACKROM = packrom
INFO=../gameinfo.properties
UZEBIN_DIR = ../../../bin

## Kernel settings
KERNEL_DIR = ../kernel
KERNEL_OPTIONS += -DSOUND_MIXER=1 
KERNEL_OPTIONS += -DSOUND_CHANNEL_4_ENABLE=0
KERNEL_OPTIONS += -DSOUND_CHANNEL_3_ENABLE=1
KERNEL_OPTIONS += -DSOUND_CHANNEL_2_ENABLE=1 
KERNEL_OPTIONS += -DSOUND_CHANNEL_1_ENABLE=1 
KERNEL_OPTIONS += -DVIDEO_MODE=3 
KERNEL_OPTIONS += -DSCROLLING=1
KERNEL_OPTIONS += -DINTRO_LOGO=0 
KERNEL_OPTIONS += -DOVERLAY_LINES=8

# forced to 28 when scrolling is enabled
KERNEL_OPTIONS += -DSCREEN_TILES_H=28

# 28 by default, can be defined here
KERNEL_OPTIONS += -DSCREEN_TILES_V=28

# 20 by default, can be defined here.
KERNEL_OPTIONS += -DFIRST_RENDER_LINE=20

# DEFAULT 20
KERNEL_OPTIONS += -DVRAM_TILES_V=20
KERNEL_OPTIONS += -DVRAM_TILES_H=32
# DEFAULT 12
KERNEL_OPTIONS += -DMAX_SPRITES=12
# DEFAULT 20
KERNEL_OPTIONS += -DRAM_TILES_COUNT=26
KERNEL_OPTIONS += -DFRAME_LINES=24

## Options common to compile, link and assembly rules
COMMON = -mmcu=$(MCU)

## Compile options common for all C compilation units.
CFLAGS = $(COMMON)
CFLAGS += -Wall -gdwarf-2 -std=gnu99 -DF_CPU=28636360UL -Os -fsigned-char -ffunction-sections -fno-toplevel-reorder
CFLAGS += -MD -MP -MT $(*F).o -MF dep/$(@F).d
#CFLAGS += -mcall-prologues -fno-inline
CFLAGS += $(KERNEL_OPTIONS)

## Assembly specific flags
ASMFLAGS = $(COMMON)
ASMFLAGS += $(CFLAGS)
ASMFLAGS += -x assembler-with-cpp -Wa,-gdwarf2

## Linker flags
LDFLAGS = $(COMMON)
LDFLAGS += -Wl,-Map=$(GAME).map 
LDFLAGS += -Wl,-gc-sections 
LDFLAGS += -Wl,--section-start,.noinit=0x800100 -Wl,--section-start,.data=0x800200

## Intel Hex file production flags
HEX_FLASH_FLAGS = -R .eeprom

HEX_EEPROM_FLAGS = -j .eeprom
HEX_EEPROM_FLAGS += --set-section-flags=.eeprom="alloc,load"
HEX_EEPROM_FLAGS += --change-section-lma .eeprom=0 --no-change-warnings


## Objects that must be built in order to link
OBJECTS = uzeboxVideoEngineCore.o uzeboxCore.o uzeboxSoundEngine.o uzeboxSoundEngineCore.o uzeboxVideoEngine.o $(GAME).o 

## Objects explicitly added by the user
LINKONLYOBJECTS = 

## Include Directories
INCLUDES = -I"$(KERNEL_DIR)" 

## Resources
RESOURCES = ../data/sprites.inc ../data/font1.inc ../data/font2.inc ../data/font3.inc ../data/font4.inc

## Build
all: $(RESOURCES) $(TARGET) $(GAME).hex $(GAME).eep $(GAME).lss $(GAME).uze size

## Rebuild graphics resources
../data/sprites.inc: ../data/images/sprites.png ../data/sprites.xml
	$(UZEBIN_DIR)/gconvert ../data/sprites.xml

## 2-1 mismatch is intentional
../data/font1.inc: ../data/images/fonts2.png ../data/font1.xml
	$(UZEBIN_DIR)/gconvert ../data/font1.xml

../data/font2.inc: ../data/images/fonts2.png ../data/font2.xml
	$(UZEBIN_DIR)/gconvert ../data/font2.xml

## 2-3 mismatch is intentional
../data/font3.inc: ../data/images/fonts2.png ../data/font3.xml
	$(UZEBIN_DIR)/gconvert ../data/font3.xml

## 2-4 mismatch is intentional
../data/font4.inc: ../data/images/fonts2.png ../data/font4.xml
	$(UZEBIN_DIR)/gconvert ../data/font4.xml

## Compile Kernel files
uzeboxVideoEngineCore.o: $(KERNEL_DIR)/uzeboxVideoEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxSoundEngineCore.o: $(KERNEL_DIR)/uzeboxSoundEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxCore.o: $(KERNEL_DIR)/uzeboxCore.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxSoundEngine.o: $(KERNEL_DIR)/uzeboxSoundEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxVideoEngine.o: $(KERNEL_DIR)/uzeboxVideoEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

## Compile game sources
$(GAME).o: ../smokeyAndTheBandit.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

##Link
$(TARGET): $(OBJECTS)
	 $(CC) $(LDFLAGS) $(OBJECTS) $(LINKONLYOBJECTS) $(LIBDIRS) $(LIBS) -o $(TARGET)

%.hex: $(TARGET)
	avr-objcopy -O ihex $(HEX_FLASH_FLAGS)  $< $@

%.eep: $(TARGET)
	-avr-objcopy $(HEX_EEPROM_FLAGS) -O ihex $< $@ || exit 0

%.lss: $(TARGET)
	avr-objdump -h -S $< > $@

%.uze: $(TARGET)
	-$(UZEBIN_DIR)/packrom $(GAME).hex $@ $(INFO)


UNAME := $(shell sh -c 'uname -s 2>/dev/null || echo not')
AVRSIZEFLAGS := -A ${TARGET}
ifneq (,$(findstring MINGW,$(UNAME)))
AVRSIZEFLAGS := -C --mcu=${MCU} ${TARGET}
endif

size: ${TARGET}
	@echo
	@avr-size ${AVRSIZEFLAGS}

## Clean target
.PHONY: clean
clean:
	-rm -rf $(OBJECTS) $(GAME).* dep/*


## Other dependencies
-include $(shell mkdir dep 2>/dev/null) $(wildcard dep/*)
