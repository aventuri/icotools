
BR_DIR := /home/andrea/src/buildroot-2016.05/output
CC := $(BR_DIR)/host/usr/bin/arm-linux-gnueabihf-g++
CFLAGS :=-Wall -I$(BR_DIR)/host/opt/ext-toolchain/arm-linux-gnueabihf/include/c++/5.3.1/ -I$(BR_DIR)/host/opt/ext-toolchain/arm-linux-gnueabihf/include/c++/5.3.1/arm-linux-gnueabihf/


USBMODE ?= 0
RASPI ?= 0
SUNXI ?= 1

ifeq ($(shell bash -c 'type -p icoprog'),)
SSH_RASPI ?= ssh root@raspi
else
SSH_RASPI ?= sh -c
endif

ifeq ($(USBMODE),1)
icoprog: icoprog.cc
	$(CC) -o icoprog -Wall -Os icoprog.cc -D USBMODE -lftdi -lrt -lstdc++
else
ifeq ($(RASPI),1)
icoprog: icoprog.cc
	$(CC) -o icoprog -Wall -Os icoprog.cc -lwiringPi -lrt -lstdc++
else
ifeq ($(SUNXI),1)
icoprog: libSunxi.o icoprog.cc
	$(CC) -o icoprog -Wall -Os icoprog.cc -D SUNXI libSunxi.o -lrt -lstdc++

libSunxi.o: libSunxi.cc libSunxi.h
	$(CC) -c -Wall -Os libSunxi.cc -lrt -lstdc++

endif
endif
endif

example.blif: example.v
	yosys -p 'synth_ice40 -blif example.blif' example.v

example.asc: example.blif example.pcf
	arachne-pnr -d 8k -p example.pcf -o example.asc example.blif

example.bin: example.asc
	icepack example.asc example.bin

example_sram: example.bin
	$(SSH_RASPI) '/usr/local/bin/icoprog -p' < example.bin

example_flash: example.bin
	$(SSH_RASPI) '/usr/local/bin/icoprog -f' < example.bin
	$(SSH_RASPI) '/usr/local/bin/icoprog -b'

reset:
	$(SSH_RASPI) '/usr/local/bin/icoprog -f' < example.pcf
	$(SSH_RASPI) '/usr/local/bin/icoprog -b'

install: icoprog
	sudo install icoprog /usr/local/bin/
	sudo chmod u+s /usr/local/bin/icoprog

uninstall:
	sudo rm -f /usr/local/bin/icoprog

clean:
	rm -f icoprog *.o example.blif example.asc example.bin

.PHONY: example_sram example_flash reset install clean

