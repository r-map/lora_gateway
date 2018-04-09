### get external defined data

LIBLORAGW_VERSION := `cat ../VERSION`
include library.cfg

### constant symbols

PREFIX := $(INSTALLDIR)/usr
ARCH ?=
CROSS_COMPILE ?=
CC := $(CROSS_COMPILE)gcc
AR := $(CROSS_COMPILE)ar

LBITS := $(shell getconf LONG_BIT)
ifeq ($(LBITS),64)
   LIBDIR = lib64
else
   LIBDIR = lib
endif

CFLAGS := -O2 -Wall -Wextra -std=c99 -Iinc -I. -I/usr/include/libftdi1 -I/usr/include/wiringPi -fPIC

OBJDIR = obj
INCLUDES = $(wildcard inc/*.h)

### linking options

#LIBS := -lloragw -lmpsse -lrt -lm
LIBS := -lloragw -lwiringPi -lmpsse -lrt -lm

### general build targets

all: libloragw.a libloragw.so test_loragw_spi test_loragw_reg test_loragw_hal test_loragw_gps test_loragw_cal

clean:
	rm -f libloragw.a libloragw.so
	rm -f test_loragw_*
	rm -f $(OBJDIR)/*.o
	rm -f inc/config.h

### transpose library.cfg into a C header file : config.h

inc/config.h: ../VERSION library.cfg
	@echo "*** Checking libloragw library configuration ***"
	@rm -f $@
	#File initialization
	@echo "#ifndef _LORAGW_CONFIGURATION_H" >> $@
	@echo "#define _LORAGW_CONFIGURATION_H" >> $@
	# Release version
	@echo "Release version   : $(LIBLORAGW_VERSION)"
	@echo "	#define LIBLORAGW_VERSION	"\"$(LIBLORAGW_VERSION)\""" >> $@
	# Debug options
	@echo "	#define DEBUG_AUX	$(DEBUG_AUX)" >> $@
	@echo "	#define DEBUG_SPI	$(DEBUG_SPI)" >> $@
	@echo "	#define DEBUG_REG	$(DEBUG_REG)" >> $@
	@echo "	#define DEBUG_HAL	$(DEBUG_HAL)" >> $@
	@echo "	#define DEBUG_GPS	$(DEBUG_GPS)" >> $@
	@echo "	#define DEBUG_GPIO	$(DEBUG_GPIO)" >> $@
	@echo "	#define DEBUG_LBT	$(DEBUG_LBT)" >> $@

	#ftdi config
	@echo "	#define VID 0x0403" >> $@
	@echo "	#define PID 0x6010" >> $@

        # end of file
	@echo "#endif" >> $@
	@echo "*** Configuration seems ok ***"

### library module target

$(OBJDIR):
	mkdir -p $(OBJDIR)

$(OBJDIR)/%.o: src/%.c $(INCLUDES) inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@

$(OBJDIR)/loragw_spi.o: src/loragw_spi.c $(INCLUDES) inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@

$(OBJDIR)/loragw_hal.o: src/loragw_hal.c $(INCLUDES) src/arb_fw.var src/agc_fw.var src/cal_fw.var inc/config.h | $(OBJDIR)
	$(CC) -c $(CFLAGS) $< -o $@

### static library

libloragw.a: $(OBJDIR)/loragw_hal.o $(OBJDIR)/loragw_gps.o $(OBJDIR)/loragw_reg.o $(OBJDIR)/loragw_spi.o $(OBJDIR)/loragw_aux.o $(OBJDIR)/loragw_radio.o $(OBJDIR)/loragw_fpga.o $(OBJDIR)/loragw_lbt.o
	$(AR) rcs $@ $^

### shared library

libloragw.so: $(OBJDIR)/loragw_hal.o $(OBJDIR)/loragw_gps.o $(OBJDIR)/loragw_reg.o $(OBJDIR)/loragw_spi.o $(OBJDIR)/loragw_aux.o $(OBJDIR)/loragw_radio.o $(OBJDIR)/loragw_fpga.o $(OBJDIR)/loragw_lbt.o
	$(CC) $(CFLAGS)	-shared	-o $@ $^


.PHONY: install
install: all
	install -d $(PREFIX)/$(LIBDIR)
	install -m 0755 libloragw.so $(PREFIX)/$(LIBDIR)
	install -m 0755 libloragw.a $(PREFIX)/$(LIBDIR)
	install -d $(PREFIX)/include/libloragw/
	install -D inc/*.h $(PREFIX)/include/libloragw/

dist: clean
	tar  cvfz ../libloragw-`cat ../VERSION`.tar.gz --directory=.. libloragw VERSION

### test programs

test_loragw_spi: tst/test_loragw_spi.c libloragw.so
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_reg: tst/test_loragw_reg.c libloragw.so
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_hal: tst/test_loragw_hal.c libloragw.so
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_gps: tst/test_loragw_gps.c libloragw.so
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

test_loragw_cal: tst/test_loragw_cal.c libloragw.so src/cal_fw.var
	$(CC) $(CFLAGS) -L. $< -o $@ $(LIBS)

### EOF
