---
layout: post
title: Cortex M development quickstart for advanced users
---


I’m a professional linux hacker which comes with a certain bias on how i approach things. Vendor specific tools are a no-go for me, and i have a low tolerance for corporate bullshit. So let me show you the fat-free rundown on how to approach cortex m as a proud engineer.


![](/images/compare-Cortex-M-diagramLG.png)

###ARM makes IP cores, not chips

Arms are in anything these days, and you can buy them from anyone. All the tools and processes are the same for any vendor, be it TI, NXP, ST, whatevs. All they do is add peripherals on the chip, which may all look different, but in the end there aren’t so many ways to approach things. This gives you a plethora of options when designing embedded systems, while not having to worry about learning a new toolchain. The variety you can buy ranges from 30 cents for TSSOP 20 cortex m0 to high performance high pin thingies with SATA and all the shit no one ever needs. There's ones with [bluetooth on the same chip](https://www.nordicsemi.com/) and some wifi thing from ST i think. All your code will work fine on all of them, which is marvelous. If all you want is replace your crappy AVR's, this one is 30 cents and outperforms whatever else you can buy for that:

![](/images/stm32f030.jpg)


###ARM loves you

I don't know where AVR would be if it wasnt for arduino. Anyone remembers the Atmel Butterfly? ohmygawd that thing sucked. Microchip didn't even try. The thing is, ARM actually cares about open soure and contributes *alot*. Sure, Torwalds hates alot on arm, but he hates on everyone, that's no measure.

Arm GCC is maintained by...arm, upstream. They don't brag about it, they just do it. Arm linux? You guessed it. Anyway, i don't want to fanboy out too much and instead show you how to actually use the goodies without pain.


###A cheapstarter

If you're new, start with an stm32 discovery (shortname disc as used on ##stm32). They're cheap as dirt and come with a bazzilion pins exposed. Get a cortex M3. M0 from stm is a bit less out-of-the-box-omg-wow. The disc boards come with the two only parts you will need  on the same board. The MCU with all the shizzles, which comes down to a crystal. The other side of the board is an ST-LINK, which is a fancy vendor specific way to do SWD-to-USB. You can actually litterally break the ST-Link of the board and use it on non-st cpus, because everything is interchangeable in arm world. ST was nice enough to include pin headers for that.

SWD means Serial Wire Debugger or something, which is a two-wire protocol for flashing as well as debugging. Yes with breakpoints and exception handling and [semi-hosting](http://bgamari.github.io/posts/2014-10-31-semihosting.html) and other fancyness on a fucking 30 cent mcu on *two* wires. It's magic.

![](/images/800px-STM32_LV_Discovery_board.jpg)

###Flashing/debugging

[openocd](http://openocd.org/) is the meta right now for all your flashing and debugging needs. It supports st-link, and other more obscure things. I personally prefer using the [blackmagicprobe](http://www.blacksphere.co.nz/main/blackmagic), because it skips an entire step (openocd) and interfaces with gdb directly, but starting with the disc board is just fine. There are other absurdly complicated methods, like DFU and JTAG and stmflash and you can even use the arduino IDE. Really, just ignore all that shit and use SWD with openocd. Every cortex M in the universe has those two pins and they work exactly the same way. Praise SWD, holy SWD, SWD for president.


```bash
openocd -f interface/stlink-v2.cfg \
        -f target/stm32f3x_stlink.cfg \
        -c "init" -c "halt"

```

Now connect gdb and feel magic

```bash
arm-none-eabi-gdb
(gdb) target remote localhost:3333
(gdb) bt
[blabla]
main.c in line 9829798123
(gdb) continue
```

Ok that was kinda backwards. You'll see a backtrace of whatever was on the board by default and not your own code, since i havent told you how to get your code on the thing. eh



###Bulding things


The unfortunate part is that software isn't that interchangeable by default. The compiler is all the same really (arm-none-eabi-gcc), but everything else depends on your taste and abit on the vendor, since obviously vendors dont maintain drivers for other vendors platforms. I personally use the NRF51 SDK alot because it happens to have an entire BLE stack for free. Can't be bothered to write my own. Here are some random alternatives.

- [riot](http://www.riot-os.org/) A Posix layer for a billion mcus with threds and IP and all that
- [contiki](http://www.contiki-os.org/) More popular, less posixy
- [libopencm3](http://libopencm3.org/)  A shit layer layer ontop of the more shitty STM chips
- [freertos](http://www.freertos.org/) Some people use this, i dont know why.

Essentially all of these come down to doing this in a Makefile

1. include some driver for the vendor specific peripherals
2. arm-none-eabi-gcc -c mycode.c driver.c -o mything.o
3. arm-none-eabi-gcc mything.o -o thingy.out  -mcpu=cortex-m3 -mthumb -Tvendor.ld
4. arm-none-eabi-objcopy -O ihex thingy.out thingy.hex

A hex file is some sort of elf. I never understood the difference, neither does it matter. ELF is the format linux and other non-shitty operating systems use to separate binaries into executable code sections, resources and bla. In our case they just separate into flash regions. Openocd can read that stuff and translate it to whatever SWD wants. 

The ld script is the only complicated thing. If you don't want to use an "OS" or "SDK", of which there are a million, you should still just copy their chip specific ld script. It's not that hard to understand, but it's kinda boring. Don't worry about fuses or anything outside your hex file. There is nothing outside your hex file. Unless your chip vendor is a sick sadist, in which case you can just recompile with -Tothervendor.ld

So once you have a hex, wire up the SWD with the two magic wires and unicorns appear:


```bash
openocd -f interface/stlink-v2.cfg \
        -f target/stm32f3x_stlink.cfg \
        -c "init" -c "halt" \
        -c "flash write_image erase thingy.hex"
```


for comparison, here's a totally different board from a totally different vendor with a totally different chip vendor with a totally different sales name for the flasher (j-link, haha) that's supposed to cost a hundret billion bucks. Guess what, it's just SWD:

```bash
openocd -f interface/stlink-v2.cfg \
        -f target/target/nrf51.cfg \
        -c "init" -c "halt" \
        -c "flash write_image erase thingy.hex"
```

![](/images/hy5-froistburn.jpg)

Notice any difference? I don't in practice. Wellcome to arm.
