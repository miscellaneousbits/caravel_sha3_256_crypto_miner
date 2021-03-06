FIRMWARE_PATH = ../..
RTL_PATH = ../../../../rtl
IP_PATH = ../../../../ip
BEHAVIOURAL_MODELS = ../../ 

GCC_PATH?=/opt/riscv32imc/bin
GCC_PREFIX?=riscv32-unknown-elf
PDK_PATH?=~/openlane/pdks/sky130A
GCC_OPTS=-fverbose-asm -Wa,-adhlns="$@.lst" -Wl,-Map=$@.map
GCC_OPTS+=-march=rv32imc -mabi=ilp32 -Wl,-Bstatic,-T,$(FIRMWARE_PATH)/sections.lds,--strip-debug
GCC_OPTS+=-ffreestanding -nostdlib
GCC_OPTS+=-O3

SIM?=RTL

.SUFFIXES:

PATTERN = miner_test

all:  ${PATTERN:=.vcd}

hex:  ${PATTERN:=.hex}

%.vvp: %_tb.v %.hex
ifeq ($(SIM),RTL)
	iverilog -DFUNCTIONAL -DSIM -I $(BEHAVIOURAL_MODELS) \
	-I $(PDK_PATH) -I $(IP_PATH) -I $(RTL_PATH) \
	$< -o $@
else
	iverilog -DFUNCTIONAL -DSIM -DGL -I $(BEHAVIOURAL_MODELS) \
	-I $(PDK_PATH) -I $(IP_PATH) -I $(RTL_PATH) \
	$< -o $@
endif

%.vcd: %.vvp
	vvp $<

%.elf: %.c $(FIRMWARE_PATH)/sections.lds $(FIRMWARE_PATH)/start.s
	${GCC_PATH}/${GCC_PREFIX}-gcc ${GCC_OPTS} -o $@ $(FIRMWARE_PATH)/start.s $<

%.hex: %.elf
	${GCC_PATH}/${GCC_PREFIX}-objcopy -O verilog $< $@ 
	# to fix flash base address
	sed -i 's/@10000000/@00000000/g' $@

%.bin: %.elf
	${GCC_PATH}/${GCC_PREFIX}-objcopy -O binary $< /dev/stdout | tail -c +1048577 > $@

# ---- Clean ----

clean:
	rm -f *.elf *.hex *.bin *.vvp *.vcd *.log *.lst *.map

.PHONY: clean hex all
