include ../../py/mkenv.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

CROSS_COMPILE = arm-none-eabi-

INC += -I.
INC += -I$(TOP)
INC += -I$(BUILD)

CFLAGS_CORTEX_M4 = -mthumb -mtune=cortex-m4 -mcpu=cortex-m4 -msoft-float -fsingle-precision-constant -Wdouble-promotion
CFLAGS = $(INC) -Wall -Werror -std=c99 -nostdlib $(CFLAGS_CORTEX_M4) $(COPT)
LDFLAGS = -nostdlib -T stm32f405.ld -Map=$@.map --cref --gc-sections

CSUPEROPT = -Os # save some code space

# Tune for Debugging or Optimization
ifeq ($(DEBUG), 1)
CFLAGS += -O0 -ggdb
else
CFLAGS += -Os -DNDEBUG
CFLAGS += -fdata-sections -ffunction-sections
endif

LIBS =

SRC_EMLIB = \
	emlib/src/em_acmp.c \
	emlib/src/em_adc.c \
	emlib/src/em_aes.c \
	emlib/src/em_assert.c \
	emlib/src/em_burt.c \
	emlib/src/em_can.c \
	emlib/src/em_cmu.c \
	emlib/src/em_core.c \
	emlib/src/em_cryotimer.c \
	emlib/src/em_crypto.c \
	emlib/src/em_csen.c \
	emlib/src/em_dac.c \
	emlib/src/em_dbg.c \
	emlib/src/em_dma.c \
	emlib/src/em_ebi.c \
	emlib/src/em_emu.c \
	emlib/src/em_eusart.c \
	emlib/src/em_gpcrc.c \
	emlib/src/em_gpio.c \
	emlib/src/em_i2c.c \
	emlib/src/em_iadc.c \
	emlib/src/em_idac.c \
	emlib/src/em_int.c \
	emlib/src/em_lcd.c \
	emlib/src/em_ldma.c \
	emlib/src/em_lesense.c \
	emlib/src/em_letimer.c \
	emlib/src/em_leuart.c \
	emlib/src/em_mpu.c \
	emlib/src/em_msc.c \
	emlib/src/em_opamp.c \
	emlib/src/em_pcnt.c \
	emlib/src/em_pdm.c \
	emlib/src/em_prs.c \
	emlib/src/em_qspi.c \
	emlib/src/em_rmu.c \
	emlib/src/em_rtc.c \
	emlib/src/em_rtcc.c \
	emlib/src/em_se.c \
	emlib/src/em_system.c \
	emlib/src/em_timer.c \
	emlib/src/em_usart.c \
	emlib/src/em_vcmp.c \
	emlib/src/em_vdac.c \
	emlib/src/em_wdog.c \

SRC_USB = \
	usb/src/em_usbd.c \
	usb/src/em_usbdch9.c \
	usb/src/em_usbdep.c \
	usb/src/em_usbdint.c \
	usb/src/em_usbh.c \
	usb/src/em_usbhal.c \
	usb/src/em_usbhep.c \
	usb/src/em_usbhint.c \
	usb/src/em_usbtimer.c \

SRC_C = \
	main.c \
	uart_core.c \
	lib/utils/printf.c \
	lib/utils/stdout_helpers.c \
	lib/utils/pyexec.c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	$(BUILD)/_frozen_mpy.c \

OBJ = $(PY_CORE_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))

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
