# Select the board to build for.
ifeq ($(BOARD),)
  $(error You must provide a BOARD parameter)
else
  ifeq ($(wildcard boards/$(BOARD)/.),)
    $(error Invalid BOARD specified)
  endif
endif


# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

include ../../py/mkenv.mk
include boards/$(BOARD)/mpconfigboard.mk
-include mpconfigport.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

include $(TOP)/supervisor/supervisor.mk

CROSS_COMPILE = arm-none-eabi-

BOSSAC := tools/bossac_osx

HAL_DIR=hal/$(MCU_SERIES)

INC += -I. \
       -I../.. \
       -I../lib/mp-readline \
       -I../lib/timeutils \
       -Iasf4/$(CHIP_FAMILY) \
       -Iasf4/$(CHIP_FAMILY)/hal/include \
       -Iasf4/$(CHIP_FAMILY)/hal/utils/include \
       -Iasf4/$(CHIP_FAMILY)/hri \
       -Iasf4/$(CHIP_FAMILY)/hpl/core \
       -Iasf4/$(CHIP_FAMILY)/hpl/gclk \
       -Iasf4/$(CHIP_FAMILY)/hpl/pm \
       -Iasf4/$(CHIP_FAMILY)/hpl/port \
       -Iasf4/$(CHIP_FAMILY)/hpl/rtc \
       -Iasf4/$(CHIP_FAMILY)/hpl/tc \
       -Iasf4/$(CHIP_FAMILY)/include \
       -Iasf4/$(CHIP_FAMILY)/CMSIS/Include \
       -Iasf4_conf/$(CHIP_FAMILY) \
       -Iboards/$(BOARD) \
       -Iboards/ \
       -Iperipherals/ \
       -Ifreetouch \
       -I../../lib/tinyusb/src \
       -I../../supervisor/shared/usb \
       -I$(BUILD)

BASE_CFLAGS = \
	-fsingle-precision-constant \
	-fno-strict-aliasing \
	-Wdouble-promotion \
	-Wno-endif-labels \
	-Wstrict-prototypes \
	-Werror-implicit-function-declaration \
	-Wfloat-equal \
	-Wundef \
	-Wshadow \
	-Wwrite-strings \
	-Wsign-compare \
	-Wmissing-format-attribute \
	-Wno-deprecated-declarations \
	-Wnested-externs \
	-Wunreachable-code \
	-Wcast-align \
	-Wno-error=lto-type-mismatch \
	-D__$(CHIP_VARIANT)__ \
	-ffunction-sections \
	-fdata-sections \
	-fshort-enums \
	-DCIRCUITPY_SOFTWARE_SAFE_MODE=0x0ADABEEF \
	-DCIRCUITPY_CANARY_WORD=0xADAF00 \
	-DCIRCUITPY_SAFE_RESTART_WORD=0xDEADBEEF \
	--param max-inline-insns-single=500


# NDEBUG disables assert() statements. This reduces code size pretty dramatically, per tannewt.

ifeq ($(CHIP_FAMILY), samd21)
CFLAGS += -Os -DNDEBUG
# TinyUSB defines
CFLAGS += -DCFG_TUSB_MCU=OPT_MCU_SAMD21 -DCFG_TUD_CDC_RX_BUFSIZE=128 -DCFG_TUD_CDC_TX_BUFSIZE=128 -DCFG_TUD_MSC_BUFSIZE=512
endif

ifeq ($(CHIP_FAMILY), samd51)
CFLAGS += -Os -DNDEBUG
# TinyUSB defines
CFLAGS += -DCFG_TUSB_MCU=OPT_MCU_SAMD51 -DCFG_TUD_CDC_RX_BUFSIZE=256 -DCFG_TUD_CDC_TX_BUFSIZE=256 -DCFG_TUD_MSC_BUFSIZE=1024
endif

#Debugging/Optimization
ifeq ($(DEBUG), 1)
  # Turn on Python modules useful for debugging (e.g. uheap, ustack).
  CFLAGS += -ggdb
  # You may want to disable -flto if it interferes with debugging.
  CFLAGS += -flto
  # You may want to enable these flags to make setting breakpoints easier.
  # CFLAGS += -fno-inline -fno-ipa-sra
  ifeq ($(CHIP_FAMILY), samd21)
    CFLAGS += -DENABLE_MICRO_TRACE_BUFFER
  endif
else
  # -finline-limit can shrink the image size.
  # -finline-limit=80 or so is similar to not having it on.
  # There is no simple default value, though.
  ifdef INTERNAL_FLASH_FILESYSTEM
    CFLAGS += -finline-limit=50
  endif
  ifdef CFLAGS_INLINE_LIMIT
    CFLAGS += -finline-limit=$(CFLAGS_INLINE_LIMIT)
  endif
  CFLAGS += -flto
endif

CFLAGS += $(INC) -Wall -Werror -std=gnu11 -nostdlib $(BASE_CFLAGS) $(CFLAGS_MOD) $(COPT)

ifeq ($(CHIP_FAMILY), samd21)
CFLAGS += \
	-mthumb \
	-mabi=aapcs-linux \
	-mcpu=cortex-m0plus \
	-msoft-float \
	-mfloat-abi=soft \
	-DSAMD21
endif
ifeq ($(CHIP_FAMILY), samd51)
CFLAGS += \
	-mthumb \
	-mabi=aapcs-linux \
	-mcpu=cortex-m4 \
	-mfloat-abi=hard \
	-mfpu=fpv4-sp-d16 \
	-DSAMD51
endif

ifneq ($(FROZEN_DIR),)
# To use frozen source modules, put your .py files in a subdirectory (eg scripts/)
# and then invoke make with FROZEN_DIR=scripts (be sure to build from scratch).
CFLAGS += -DMICROPY_MODULE_FROZEN_STR
CFLAGS += -Wno-error=lto-type-mismatch
endif

# To use frozen bytecode, put your .py files in a subdirectory (eg frozen/) and
# then invoke make with FROZEN_MPY_DIR=frozen or FROZEN_MPY_DIRS="dir1 dir2"
# (be sure to build from scratch).

ifneq ($(FROZEN_MPY_DIRS),)
CFLAGS += -DMICROPY_QSTR_EXTRA_POOL=mp_qstr_frozen_const_pool
CFLAGS += -DMICROPY_MODULE_FROZEN_MPY
CFLAGS += -Wno-error=lto-type-mismatch
endif

LDFLAGS = $(CFLAGS) -nostartfiles -fshort-enums -Wl,-nostdlib -Wl,-T,$(LD_FILE) -Wl,-Map=$@.map -Wl,-cref -Wl,-gc-sections -specs=nano.specs
LIBS := -lgcc -lc

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

# Propagate longint choice from .mk to C. There's no easy string comparison
# in cpp coniditionals, so we #define separate names for each.
ifeq ($(LONGINT_IMPL),NONE)
CFLAGS += -DLONGINT_IMPL_NONE
endif

ifeq ($(LONGINT_IMPL),MPZ)
CFLAGS += -DLONGINT_IMPL_MPZ
endif

ifeq ($(LONGINT_IMPL),LONGLONG)
CFLAGS += -DLONGINT_IMPL_LONGLONG
endif

ifeq ($(CHIP_FAMILY), samd21)
LDFLAGS += -mthumb -mcpu=cortex-m0plus -Lasf/thirdparty/CMSIS/Lib/GCC/
BOOTLOADER_SIZE := 0x2000
else ifeq ($(CHIP_FAMILY), samd51)
LDFLAGS += -mthumb -mcpu=cortex-m4
BOOTLOADER_SIZE := 0x4000
endif

SRC_ASF := \
	gcc/gcc/startup_$(CHIP_FAMILY).c \
	gcc/system_$(CHIP_FAMILY).c \
	hal/src/hal_adc_sync.c \
	hal/src/hal_atomic.c \
	hal/src/hal_calendar.c \
	hal/src/hal_dac_sync.c \
	hal/src/hal_delay.c \
	hal/src/hal_flash.c \
	hal/src/hal_i2c_m_sync.c \
	hal/src/hal_io.c \
	hal/src/hal_sleep.c \
	hal/src/hal_spi_m_sync.c \
	hal/src/hal_timer.c \
	hal/src/hal_usart_async.c \
	hpl/adc/hpl_adc.c \
	hpl/core/hpl_init.c \
	hpl/dac/hpl_dac.c \
	hpl/gclk/hpl_gclk.c \
	hpl/nvmctrl/hpl_nvmctrl.c \
	hpl/pm/hpl_pm.c \
	hpl/rtc/hpl_rtc.c \
	hpl/sercom/hpl_sercom.c \
	hpl/systick/hpl_systick.c \
	hal/utils/src/utils_list.c \
	hal/utils/src/utils_ringbuffer.c \

ifeq ($(CHIP_FAMILY), samd21)
SRC_ASF += \
	hpl/core/hpl_core_m0plus_base.c \
	hpl/sysctrl/hpl_sysctrl.c \

else ifeq ($(CHIP_FAMILY), samd51)
SRC_ASF += \
    hal/src/hal_rand_sync.c \
	hpl/core/hpl_core_m4.c \
	hpl/mclk/hpl_mclk.c \
	hpl/osc32kctrl/hpl_osc32kctrl.c \
	hpl/oscctrl/hpl_oscctrl.c \
	hpl/trng/hpl_trng.c \

endif

SRC_ASF := $(addprefix asf4/$(CHIP_FAMILY)/, $(SRC_ASF))

SRC_C = \
	audio_dma.c \
	board_busses.c \
	background.c \
	fatfs_port.c \
	mphalport.c \
	reset.c \
	peripherals/samd/clocks.c \
	peripherals/samd/dma.c \
	peripherals/samd/events.c \
	peripherals/samd/external_interrupts.c \
	peripherals/samd/sercom.c \
	peripherals/samd/timers.c \
	peripherals/samd/$(CHIP_FAMILY)/adc.c \
	peripherals/samd/$(CHIP_FAMILY)/cache.c \
	peripherals/samd/$(CHIP_FAMILY)/clocks.c \
	peripherals/samd/$(CHIP_FAMILY)/dma.c \
	peripherals/samd/$(CHIP_FAMILY)/events.c \
	peripherals/samd/$(CHIP_FAMILY)/external_interrupts.c \
	peripherals/samd/$(CHIP_FAMILY)/pins.c \
	peripherals/samd/$(CHIP_FAMILY)/sercom.c \
	peripherals/samd/$(CHIP_FAMILY)/timers.c \
	tick.c \
	bindings/samd/__init__.c \
	bindings/samd/Clock.c \
	boards/$(BOARD)/board.c \
	boards/$(BOARD)/pins.c \
	lib/oofatfs/ff.c \
	lib/oofatfs/option/ccsbcs.c \
	lib/timeutils/timeutils.c \
	lib/tinyusb/src/portable/microchip/$(CHIP_FAMILY)/dcd_$(CHIP_FAMILY).c \
	lib/tinyusb/src/portable/microchip/$(CHIP_FAMILY)/hal_$(CHIP_FAMILY).c \
	lib/utils/buffer_helper.c \
	lib/utils/context_manager_helpers.c \
	lib/utils/interrupt_char.c \
	lib/utils/pyexec.c \
	lib/utils/stdout_helpers.c \
	lib/utils/sys_stdio_mphal.c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	freetouch/adafruit_ptc.c \
	supervisor/shared/memory.c

ifeq ($(MICROPY_PY_NETWORK),1)
CFLAGS += -DMICROPY_PY_NETWORK=1

SRC_MOD += lib/netutils/netutils.c

ifneq ($(MICROPY_PY_WIZNET5K),0)
WIZNET5K_DIR=drivers/wiznet5k
INC += -I$(TOP)/$(WIZNET5K_DIR)
CFLAGS_MOD += -DMICROPY_PY_WIZNET5K=$(MICROPY_PY_WIZNET5K) -D_WIZCHIP_=$(MICROPY_PY_WIZNET5K)
SRC_MOD += $(addprefix $(WIZNET5K_DIR)/,\
        ethernet/w$(MICROPY_PY_WIZNET5K)/w$(MICROPY_PY_WIZNET5K).c \
        ethernet/wizchip_conf.c \
        ethernet/socket.c \
        internet/dns/dns.c \
        internet/dhcp/dhcp.c \
        )

endif # MICROPY_PY_WIZNET5K
endif # MICROPY_PY_NETWORK

SRC_COMMON_HAL = \
	board/__init__.c \
	busio/__init__.c \
	busio/I2C.c \
	busio/SPI.c \
	busio/UART.c \
	digitalio/__init__.c \
	digitalio/DigitalInOut.c \
	displayio/FourWire.c \
	i2cslave/__init__.c \
	i2cslave/I2CSlave.c \
	microcontroller/__init__.c \
	microcontroller/Pin.c \
	microcontroller/Processor.c \
	neopixel_write/__init__.c \
	os/__init__.c \
	rotaryio/__init__.c \
	rotaryio/IncrementalEncoder.c \
	rtc/__init__.c \
	rtc/RTC.c \
	supervisor/__init__.c \
	supervisor/Runtime.c \
	time/__init__.c \
	analogio/__init__.c \
	analogio/AnalogIn.c \
	analogio/AnalogOut.c \
	nvm/__init__.c \
	nvm/ByteArray.c \
	pulseio/__init__.c \
	pulseio/PulseIn.c \
	pulseio/PulseOut.c \
	pulseio/PWMOut.c \
	touchio/__init__.c \
	touchio/TouchIn.c

ifeq ($(INTERNAL_LIBM),1)
SRC_LIBM = $(addprefix lib/,\
	libm/math.c \
	libm/roundf.c \
	libm/fmodf.c \
	libm/nearbyintf.c \
	libm/ef_sqrt.c \
	libm/kf_rem_pio2.c \
	libm/kf_sin.c \
	libm/kf_cos.c \
	libm/kf_tan.c \
	libm/ef_rem_pio2.c \
	libm/sf_sin.c \
	libm/sf_cos.c \
	libm/sf_tan.c \
	libm/sf_frexp.c \
	libm/sf_modf.c \
	libm/sf_ldexp.c \
	libm/asinfacosf.c \
	libm/atanf.c \
	libm/atan2f.c \
	)
endif

# These don't have corresponding files in each port but are still located in
# shared-bindings to make it clear what the contents of the modules are.
SRC_BINDINGS_ENUMS = \
	digitalio/Direction.c \
	digitalio/DriveMode.c \
	digitalio/Pull.c \
	microcontroller/RunMode.c \
	help.c \
	math/__init__.c \
	supervisor/__init__.c \
	util.c


SRC_SHARED_MODULE = \
	bitbangio/__init__.c \
	bitbangio/I2C.c \
	bitbangio/OneWire.c \
	bitbangio/SPI.c \
	busio/OneWire.c \
	displayio/__init__.c \
	displayio/Bitmap.c \
	displayio/ColorConverter.c \
	displayio/Group.c \
	displayio/OnDiskBitmap.c \
	displayio/Palette.c \
	displayio/Sprite.c \
	gamepad/__init__.c \
	gamepad/GamePad.c \
	_stage/__init__.c \
	_stage/Layer.c \
	_stage/Text.c \
	storage/__init__.c \
	os/__init__.c \
	random/__init__.c \
	struct/__init__.c \
	uheap/__init__.c \
	ustack/__init__.c \
	usb_hid/__init__.c \
	usb_hid/Device.c

	# usb_midi/__init__.c
	# usb_midi/Port.c

ifeq ($(MICROPY_PY_NETWORK),1)
SRC_SHARED_MODULE += socket/__init__.c network/__init__.c
ifneq ($(MICROPY_PY_WIZNET5K),0)
SRC_SHARED_MODULE += wiznet/__init__.c wiznet/wiznet5k.c
endif
endif

# SAMRs don't have a DAC
ifneq ($(CHIP_VARIANT),SAMR21G18A)
	SRC_COMMON_HAL += \
		audioio/__init__.c \
		audioio/AudioOut.c
	SRC_SHARED_MODULE += \
		audioio/__init__.c \
		audioio/Mixer.c \
		audioio/RawSample.c \
		audioio/WaveFile.c
endif

# The smallest SAMD51 packages don't have I2S. Everything else does.
ifneq ($(CHIP_VARIANT),SAMD51G18A)
	ifneq ($(CHIP_VARIANT),SAMD51G19A)
		ifneq ($(CHIP_VARIANT),SAMR21G18A)
			SRC_COMMON_HAL += \
				audiobusio/__init__.c \
				audiobusio/I2SOut.c \
				audiobusio/PDMIn.c
			SRC_C += peripherals/samd/i2s.c peripherals/samd/$(CHIP_FAMILY)/i2s.c
		endif
	endif
endif

SRC_COMMON_HAL_EXPANDED = $(addprefix shared-bindings/, $(SRC_COMMON_HAL)) \
                          $(addprefix shared-bindings/, $(SRC_BINDINGS_ENUMS)) \
                          $(addprefix common-hal/, $(SRC_COMMON_HAL))

SRC_SHARED_MODULE_EXPANDED = $(addprefix shared-bindings/, $(SRC_SHARED_MODULE)) \
                             $(addprefix shared-module/, $(SRC_SHARED_MODULE))

SRC_S = supervisor/$(CHIP_FAMILY)_cpu.s

OBJ = $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_ASF:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_EXPANDED:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_SHARED_MODULE_EXPANDED:.c=.o))
ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))

SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED) $(STM_SRC_C)
# Sources that only hold QSTRs after pre-processing.
SRC_QSTR_PREPROCESSOR += peripherals/samd/$(CHIP_FAMILY)/clocks.c

all: $(BUILD)/firmware.bin $(BUILD)/firmware.uf2

$(BUILD)/firmware.elf: $(OBJ)
	$(STEPECHO) "LINK $@"
	$(Q)$(CC) -o $@ $(LDFLAGS) $^ -Wl,--start-group $(LIBS) -Wl,--end-group
	$(Q)$(SIZE) $@ | $(PYTHON3) $(TOP)/tools/build_memory_info.py $(LD_FILE)

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O binary -j .vectors -j .text -j .data $^ $@

$(BUILD)/firmware.uf2: $(BUILD)/firmware.bin
	$(STEPECHO) "Create $@"
	$(Q)$(PYTHON3) $(TOP)/tools/uf2/utils/uf2conv.py -b $(BOOTLOADER_SIZE) -c -o $@ $^

deploy: $(BUILD)/firmware.bin
	$(ECHO) "Writing $< to the board"
	$(BOSSAC) -u $<

# Run emulation build on a POSIX system with suitable terminal settings
run:
	stty raw opost -echo
	build/firmware.elf
	@echo Resetting terminal...
# This sleep is useful to spot segfaults
	sleep 1
	reset

test: $(BUILD)/firmware.elf
	$(Q)/bin/echo -e "print('hello world!', list(x+1 for x in range(10)), end='eol\\\\n')\\r\\n\\x04" | $(BUILD)/firmware.elf | tail -n2 | grep "^hello world! \\[1, 2, 3, 4, 5, 6, 7, 8, 9, 10\\]eol"

include $(TOP)/py/mkrules.mk
