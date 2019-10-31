# Select the board to build for: if not given on the command line,
# then default to EFM32GG_STK3700.
BOARD ?= EFM32GG_STK3700

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

BOARD_DIR ?= boards/$(BOARD)
ifeq ($(wildcard $(BOARD_DIR)/.),)
$(error Invalid BOARD specified: $(BOARD_DIR))
endif

include ../../py/mkenv.mk
include $(BOARD_DIR)/mpconfigboard.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

STARTUP_FILE ?= devices/EFM32GG/Source/startup_efm32gg.o
SYSTEM_FILE ?= devices/EFM32GG/Source/system_efm32gg.o

# Select the cross compile prefix
CROSS_COMPILE = arm-none-eabi-

INC += -I.
INC += -I$(TOP)
INC += -I$(TOP)/lib/cmsis/inc
INC += -I$(BUILD)
INC += -Iemlib/inc
INC += -Iusb/inc

CFLAGS_CORTEX_M3 = -mthumb -mtune=cortex-m3 -mcpu=cortex-m3 -msoft-float -fsingle-precision-constant -Wdouble-promotion

CFLAGS = $(INC) -Wall -Wpointer-arith -Werror -std=gnu99 -nostdlib $(CFLAGS_MOD) $(CFLAGS_EXTRA)
CFLAGS += -D$(DEVICE)
CFLAGS += $(CFLAGS_CORTEX_M3)
CFLAGS += $(COPT)
CFLAGS += -I$(BOARD_DIR)

LDFLAGS = -nostdlib -L $(LD_DIR) $(addprefix -T,$(LD_FILES)) -Map=$(@:.elf=.map) --cref

# Remove uncalled code from the final image.
CFLAGS += -fdata-sections -ffunction-sections
LDFLAGS += --gc-sections

# Tune for Debugging or Optimization
ifeq ($(DEBUG), 1)
CFLAGS += -O0 -ggdb
else
CFLAGS += -Os -DNDEBUG
endif

# Options for mpy-cross
MPY_CROSS_FLAGS += -march=armv7m

CSUPEROPT = -Os # save some code space

LIBS = -Wl,--start-group -lgcc -lc -lnosys -Wl,--end-group

SRC_EMLIB = $(addprefix emlib/src/, \
	em_acmp.c \
	em_adc.c \
	em_aes.c \
	em_assert.c \
	em_burt.c \
	em_can.c \
	em_cmu.c \
	em_core.c \
	em_cryotimer.c \
	em_crypto.c \
	em_csen.c \
	em_dac.c \
	em_dbg.c \
	em_dma.c \
	em_ebi.c \
	em_emu.c \
	em_eusart.c \
	em_gpcrc.c \
	em_gpio.c \
	em_i2c.c \
	em_iadc.c \
	em_idac.c \
	em_int.c \
	em_lcd.c \
	em_ldma.c \
	em_lesense.c \
	em_letimer.c \
	em_leuart.c \
	em_mpu.c \
	em_msc.c \
	em_opamp.c \
	em_pcnt.c \
	em_pdm.c \
	em_prs.c \
	em_qspi.c \
	em_rmu.c \
	em_rtc.c \
	em_rtcc.c \
	em_se.c \
	em_system.c \
	em_timer.c \
	em_usart.c \
	em_vcmp.c \
	em_vdac.c \
	em_wdog.c \
)

SRC_USB = $(addprefix usb/src/, \
	em_usbd.c \
	em_usbdch9.c \
	em_usbdep.c \
	em_usbdint.c \
	em_usbh.c \
	em_usbhal.c \
	em_usbhep.c \
	em_usbhint.c \
	em_usbtimer.c \
)

SRC_C = \
	main.c \
	uart_core.c \
	lib/utils/printf.c \
	lib/utils/stdout_helpers.c \
	lib/utils/pyexec.c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	$(BUILD)/_frozen_mpy.c \

SRC_O = \
	$(STARTUP_FILE) \
	$(SYSTEM_FILE)

OBJ = $(PY_CORE_O)
OBJ += $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_O))
OBJ += $(addprefix $(BUILD)/, $(SRC_EMLIB:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_USB:.c=.o))

all: $(BUILD)/firmware.elf

$(BUILD)/_frozen_mpy.c: frozentest.mpy $(BUILD)/genhdr/qstrdefs.generated.h
	$(ECHO) "MISC freezing bytecode"
	$(Q)$(TOP)/tools/mpy-tool.py -f -q $(BUILD)/genhdr/qstrdefs.preprocessed.h -mlongint-impl=none $< > $@

$(BUILD)/firmware.elf: $(OBJ)
	$(ECHO) "LINK $@"
	$(Q)$(LD) $(LDFLAGS) -o $@ $^ $(LIBS)
	$(Q)$(SIZE) $@

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(Q)$(OBJCOPY) -O binary -j .isr_vector -j .text -j .data $^ $(BUILD)/firmware.bin

include $(TOP)/py/mkrules.mk
