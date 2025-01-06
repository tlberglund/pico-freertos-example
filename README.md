# Raspberry Pi Pico FreeRTOS Example

This is my [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) and [FreeRTOS](https://www.freertos.org/) example repo. There are many like it, but this one is mine.

If you know something about FreeRTOS's board support packages, then it might interest you to know that this repo uses the `portable/ThirdParty/` options. These turn out to be reasonably straightforward to get running, if by straightforward you mean hours and hours of sifting through dependencies, wondering why the system tick isn't running, finding out heap allocations were causing the system to hang, calling down oaths on FreeRTOS, CMake, and microcontrollers generally, and crafting an unusually baroque list of include directories in the build file. I chased so many dead ends to get here, and now maybe you don't have to.

## Required Tooling

I've only run this on MacOS, and if I recall correctly, you'll need these tools installed for a good command-line experience:

* `brew install picotool`
* `brew install cmake`
* `brew install ninja`
* `brew install minicom` (optional)

Raspberry Pi also provides a nice VSCode extension that provides support for building and loading code and monitoring serial port output. It looks like this, but I won't document it in any more detail in this README:

<img alt="The official Raspberry Pi Pico VSCode Plugin" width="486" alt="image" src="https://github.com/user-attachments/assets/5e1d32d6-6783-4f9c-b365-b10e69216636" />

## Supported Platforms

* Pico W ([datasheet](https://datasheets.raspberrypi.com/picow/pico-w-datasheet.pdf))
* Pico 2 ([datasheet](https://datasheets.raspberrypi.com/pico/pico-2-datasheet.pdf))

The build is configurable via the `PICO_BOARD` variable in the CMake build (see Building below). Towards the top of `CMakeLists.txt`, you'll see these lines:

```CMake
#set(PICO_BOARD pico_w CACHE STRING "Board type")
set(PICO_BOARD pico2 CACHE STRING "Board type")
```

Be sure to uncomment one and only one of those lines, depending on the kind of board you want to build for.

## Cloning

This repo uses submodules for the [Pico SDK](https://github.com/raspberrypi/pico-sdk) and [FreeRTOS](https://github.com/FreeRTOS/FreeRTOS-Kernel). Don't blame me; this is really the best way to manage C-language dependencies. You want Maven Central? Go try all this in Kotlin.

The submodules are set to use Pico SDK 2.1.0 and an unreleased version of FreeRTOS, which perfidy is necessary to support RP2350-based boards.

To clone the repo, bring this to the working directory you use for super meaningful projects like this:

`git clone --recurse-submodules https://github.com/tlberglund/pico-freertos-example.git`

If it's too late and you already cloned the normal way and you're worrying what do to about the dang submodules, get into the clone directory and drop this:

`git submodule update --init --recursive`

Everything will be fine.

## Building

The build uses [CMake](https://cmake.org) and [Ninja](https://ninja-build.org/), which is standard for Pico C/C++ projects. If you've found this repo because you're looking to get started on the Pico, you might also be new to CMake, which is evidence of a longstanding pattern of positive life choices. To run the build, get into the clone directory and type these things:

1. `mkdir build`
2. `CMake -G Ninja -S . -B build`
3. `ninja -C build`

CMake is actually a system for describing a build, which it typically generates as a makefile or a Ninja build file. Ninja is the tool that is actually invoking the compiler and linker. If you're iterating on an example and changing only your C or C++ code, you can just re-run the `ninja` command without regenerating the build with CMake. This actually is as fast as it claims to be, and isn't a terrible workflow once you get into it.

## That Debugging Life

1. Plug the Pico board into a USB port on your computer using a USB Mini cable you found in the box of cables that you keep around in case you need them. _See, there's a reason you have those!_
2. Wait, make sure you held down the BOOTSEL button on the Pico before you plugged it in. You'll need to do this every time before you load code into it. I don't like it any more than you do, but the sense of vindication you had when you found that cable makes it all worth it.
3. Run the build (see above).
4. From the project directory, run: `picotool load -x build/freeRTOS_hello_world.uf2`
5. Watch that light blink! This is exactly what it felt like for the first human who made a fire.
6. The output of the `printf` function goes to a serial device you can monitor using `minicom`. On MacOS (and presumably Linux, but I haven't tried it), type `minicom -D /dev/tty.usb` and hit tab. If there's only one device with that name, that's your Pico's serial port. If there are multiple devices, well, you're writing firmware. Trial and error is your life now.

Note that the USB serial device name will change randomly throughout your debug session, so don't expect the exact same command to work every time. You'll have to repeat the tab complete step occasionally. There's a one-second sleep on startup in the code to give you a tiny bit of time to do this after `picotool` finishes, so you can still see startup debug output. Feel free to increase the delay if it makes life easier.
