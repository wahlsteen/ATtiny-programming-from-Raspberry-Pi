# ATtiny programming from Raspberry Pi
This repository instructs how to program an ATtiny85 from Raspberry Pi
Hardware and wiring 

GPIO pin|ATtiny pin|Comment|
|-------|----------|-------|
|15|1|GPIO25 to Reset (through 1K, Blue wire)
|17|8|3.3 V (Green wire)
|19|5|MOSI (through 1K, Yellow wire)
|21|6|MISO (through 1K, Orange wire)
|23|7|SCLK (through 1K, Red wire)
|25|4|GND (Brown wire)



Activate SPI on the Raspberry Pi

	sudo raspi-config

Choose 
	Interface Options → SPI → Enable 

Restart

	sudo reboot

Install Tools

	sudo apt update
	sudo apt install avrdude gcc-avr avr-libc make

Test the Contact with ATtiny

	sudo avrdude -c linuxspi -P /dev/spidev0.0:/dev/gpiochip0 -p t85 -B 100 -v

change SPI speed: 
	-B 10
	-B 50
	-B 100

Healthy respond: 
	[...]
	avrdude: AVR device initialized and ready to accept instructions avrdude: device signature = 0x1e930b (probably t85)

Make a new file with code: 

	nano blink.c

The code: 

	#define F_CPU 1000000UL
	#include <avr/io.h>
	#include <util/delay.h>
	
	int main(void)
	{
	    DDRB |= (1 << PB4);
	
	    while (1)
	    {
	        PORTB ^= (1 << PB4);
	        _delay_ms(500);
	    }
	}

save
	Ctrl+X, Y, Enter 

Compile: 

	avr-gcc -mmcu=attiny85 -Os -o blink.elf blink.c
	avr-objcopy -O ihex blink.elf blink.hex

Upload: 

	sudo avrdude -c linuxspi -P /dev/spidev0.0:/dev/gpiochip0 -p t85 -B 10 -U flash:w:blink.hex
	
(change to speed that worked earlier! -B xxx)
you should see:

	avrdude: 82 bytes of flash written
	
	[...]
	
	avrdude: 82 bytes of flash verified

Pull out the reset cable and the LED should blink 

You can check the readback:

	sudo avrdude -c linuxspi -P /dev/spidev0.0:/dev/gpiochip0 -p t85 -B 50 -U flash:r:readback.hex:i
	
Empty file or most FF indicates failure 
