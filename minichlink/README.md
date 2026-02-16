# minichlink

A free, open mechanism to use the CH-LinkE $4 programming dongle for the CH32V003.

If on Linux, make sure to install the provided udev rules rule to `/etc/udev/rules.d/`.
This can be done by issuing `make install_udev_rules` as root. The makefile target will also tell the udev daemon to reload the new rules and re-evaluate them on already attached devices, essentially saving you a reboot/replug.

On Windows, if you need to you can install the WinUSB driver over the WCH interface 1.

The exe here is about 12kB and contains everything except for the libusb driver.  In Linux you need `libusb-1.0-dev`.

## UIAPduino Pro Micro CH32V003 V1.4macOS Support

install the RISC-V toolchain with homebrew following these instructions

- [RISC-V Homebrew Repository](https://github.com/riscv-software-src/homebrew-riscv)

### Modified files

- Makefile:

```makefile
CFLAGS := $(ARCHFLAG) -O0 -Wall -Wno-asm-operand-widths -Wno-deprecated-declarations -Wno-deprecated-non-prototype -D__MACOSX__ -DMINICHLINK -DCH32V003 -I. $(LIBUSB_INCS) -DDEFAULT_CHLINK_PID=0xb803
```

- minichlink:

code changed to support CH-Link PID 0xb803 on UIAPduino Pro Micro CH32V003 V1.4

line 55:
```c
// original
else if( strcmp( specpgm, "b003boot" ) == 0 )
    dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b003));

// modified for UIAPduino Pro Micro CH32V003 V1.4 with CH-Link PID 0xb803
else if( strcmp( specpgm, "b003boot" ) == 0 )
    dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b803));
```

line 74:

```c
// original
else if ((dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b003))))
{
    fprintf( stderr, "Found B003Fun Bootloader\n" );
}

// modified for UIAPduino Pro Micro CH32V003 V1.4 with CH-Link PID 0xb803
else if ((dev = TryInit_B003Fun(SimpleReadNumberInt(init_hints->serial_port, 0x1209b803))))
{
    fprintf( stderr, "Found B003Fun Bootloader\n" );
}
```

### Notes

- Copy minichlink to your Arduino IDE "tools" folder:
  `/Arduino/Arduino15/packages/UIAP/tools/minichlink-2982dfd/1.0.0/`


### Key Improvements

1. **Enhanced CH-Link Support**: The `DEFAULT_CHLINK_PID=0xb803` definition improves compatibility with CH-Link programming dongles on macOS
2. **Warning Suppression**: macOS-specific compiler warnings are suppressed for cleaner builds
3. **Universal Binary Support**: ARCH flag allows building for specific architectures (x86_64/arm64)

## Usage

```
Usage: minichlink [args]
 single-letter args may be combined, i.e. -3r
 multi-part args cannot.
 -3 Enable 3.3V
 -5 Enable 5V
 -t Disable 3.3V
 -f Disable 5V
 -u Clear all code flash - by power off (also can unbrick)
 -b Reboot out of Halt
 -e Resume from halt
 -a Place into Halt
 -D Configure NRST as GPIO
 -d Configure NRST as NRST
 -s [debug register] [value]
 -g [debug register]
 -w [binary image to write] [address, decimal or 0x, try0x08000000]
 -r [output binary image] [memory address, decimal or 0x, try 0x08000000] [size, decimal or 0x, try 16384]
   Note: for memory addresses, you can use 'flash' 'launcher' 'bootloader' 'option' 'ram' and say "ram+0x10" for instance
   For filename, you can use - for raw or + for hex.
 -T is a terminal. This MUST be the last argument.
```
 
