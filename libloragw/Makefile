# putting the configuration in a separate file
include library.cfg

# constant symbols
CC=gcc
CFLAGS=-O2 -Wall -Wextra -Iinc -I.
C99FLAGS=-O2 -Wall -Wextra -std=c99 -Iinc -I.

# configuration-dependant symbols
ifeq ($(LGW_PHY),native)
LDFLAGS=-lrt
endif
ifeq ($(LGW_PHY),ftdi)
LDFLAGS=-lrt -lmpsse
endif

# general build targets

all: libloragw.a test_loragw_spi test_loragw_reg test_loragw_hal

clean:
	rm -f *.a
	rm -f test_*
	rm -f obj/*.o
	rm -f .conf_ok

.conf_ok: library.cfg
	@echo "*** Checking Lora gateway HAL library config ***"
	@rm -f .conf_ok
ifeq ($(LGW_PHY),native)
	@echo "Selected SPI interface type: Linux native driver"
else
ifeq ($(LGW_PHY),ftdi)
	@echo "Selected SPI interface type: FTDI SPI-over-USB bridge"
else
	$(error No SPI physical layer selected)
endif
endif
	@echo "*** Config seems ok ***"
	@echo ""
	@touch .conf_ok

# static library

libloragw.a: obj/loragw_hal.o obj/loragw_reg.o obj/loragw_spi.o obj/loragw_aux.o
	ar rcs libloragw.a obj/loragw_hal.o obj/loragw_reg.o obj/loragw_spi.o obj/loragw_aux.o

# library module target

obj/loragw_aux.o: .conf_ok src/loragw_aux.c inc/loragw_aux.h
	$(CC) -c $(CFLAGS) src/loragw_aux.c -o obj/loragw_aux.o $(FLAG_AUX)

obj/loragw_spi.o: .conf_ok src/loragw_spi.native.c src/loragw_spi.ftdi.c inc/loragw_spi.h
ifeq ($(LGW_PHY),native)
	$(CC) -c $(C99FLAGS) src/loragw_spi.native.c -o obj/loragw_spi.o $(FLAG_SPI)
endif
ifeq ($(LGW_PHY),ftdi)
	$(CC) -c $(C99FLAGS) src/loragw_spi.ftdi.c -o obj/loragw_spi.o $(FLAG_SPI)
endif

obj/loragw_reg.o: .conf_ok src/loragw_reg.c inc/loragw_reg.h inc/loragw_spi.h
	$(CC) -c $(C99FLAGS) src/loragw_reg.c -o obj/loragw_reg.o $(FLAG_REG)

obj/loragw_hal.o: .conf_ok VERSION src/loragw_hal.c src/arb_fw.var src/agc_fw.var inc/loragw_hal.h inc/loragw_reg.h inc/loragw_spi.h inc/loragw_aux.h
	$(CC) -c $(C99FLAGS) src/loragw_hal.c -o obj/loragw_hal.o -D LGW_PHY="\"$(LGW_PHY)\"" $(FLAG_HAL)

# test programs

test_loragw_spi: tst/test_loragw_spi.c obj/loragw_spi.o
	$(CC) $(C99FLAGS) tst/test_loragw_spi.c obj/loragw_spi.o -o test_loragw_spi $(LDFLAGS)

test_loragw_reg: tst/test_loragw_reg.c obj/loragw_reg.o obj/loragw_spi.o
	$(CC) $(C99FLAGS) tst/test_loragw_reg.c obj/loragw_reg.o obj/loragw_spi.o -o test_loragw_reg $(LDFLAGS)
	
test_loragw_hal: tst/test_loragw_hal.c obj/loragw_hal.o obj/loragw_reg.o obj/loragw_spi.o obj/loragw_aux.o
	$(CC) $(C99FLAGS) tst/test_loragw_hal.c obj/loragw_hal.o obj/loragw_reg.o obj/loragw_spi.o obj/loragw_aux.o -o test_loragw_hal $(LDFLAGS)
