What is is
==========

IOoo is an object oriented, univercal framework that allows to write applications that can be easily ported between different platforms. The library's core are basic I/O operations, like GPIO, SPI, I2C, etc. The next layer is built by simple device drivers, implementing basic functions of connected devices. The interface is defined by an abstract class, which describes what functions can be called by the higher layers. Except for instantiation, the application should always refer to the methods defined by the generic interface. Implementations can contain platform-specific methods, however their use is limited to the parts of the application that are tied to the underlying hardware (i.e. setting pin muxing). Hardware-independent modules should always refer to the generic interfaces. 

Currently the interfaces are defined for GPIO and SPI. I2C should follow shortly. Hardware implementation exists for Beaglebone. Because of the way how the platform-specific classes are implemented, porting the framework to Raspberry PI should be trivial.

Implemented hardware interfaces
-------------------
  - __GPIO:__ GPIO operations on Beaglebone are implemented using memory mapped interface. This is the fastest way to access the I/O lines.
  - __SPI:__ SPI implementation relies on kernel drivers and accesses SPI buses through the /dev interface. This implementation should be portable across all Linux versions.

Implemented device drivers
----------------
  - HD44780-compatible LCD displays
  - TLC5946 LED controller

Why yet another I/O library?
============
__TL;DR:__ Because it's better

__Details:__ The framework has been conceived to address my need of having a universal interface that would allow me to prototype, develop and debug applications on general-purpose computer and then quickly move them the target platform based on a small microcontroller. Such approach gives the comfort of using all the development tools that I use every day while working on PC-based projects, while working on a project that will run on a much less capable hardware. 

I started to think about such portable framework long time ago. At that time I was using 8-bit microcontrollers, with small amount of RAM. It was possible to write even complex programs using *C++*, however more advanced language features (like virtual methods) were adding too much overhead to be useful. When I moved my projects to 32bit processors with more memory, such a universal framework started to look like a viable idea.
The idea behind the components of the framework is simple: 
- For hardware interfaces define an abstract class which will define all methods used by the application to "talk" to the HW module.
- The interface contains most commonly used operations, like writing and reading GPIO lines or serial interfaces. The GPIO interface allows to define a group of arbitrary lines and access them as an ordered set of wires. Mapping bits in a word to I/O lines is one of most commonly implemented operations and I wanted to stop implementing it over and over again.
- More complex devices are handled by device drivers. Writing them may look like unnecessary job for Linux systems, as many of the devices will have drivers in the kernel, however they will come in handy when the application will be running on a much simpler platform, without the support of the underlying OS.
- Device drivers are split into two parts:
    - Low-level interface, mapping control signals to hardware lines (like "set MODE line to 1", "write 0x3D to DATA lines").
    - Higher level module, implementing device's functionality (like "putChar", "clear", "drawPoint").

  Splitting the driver into two parts  completely isolates the high-level module from the wire-level interface. The PHY (low level) modules may use the abstract I/O interfaces described above, but they are not required to. As long as the PHY supports all the operations used by the upper layer, the implementation can be as close to the actual hardware as necessary. A good example of this approach is TLC5946 PHY module for Beaglebone. The module is based on generic PHY using GPIOs to control the chip, but uses PRU to generate waveforms required by GSCLK and BLANK lines.

Known problems and limitations
===============
- 4-bit interface for HD44780 has not been tested.
- Linux implementation is not MT-safe

Future work
========
- Add I2C interface (shouldn't take long)
- Add more device drivers
- Split the memory-mapped GPIO interface into two parts: 
    - Generic functionality common for all memory-mapped implementations,
    - Platform specific part
- Port library to Raspberry Pi. SPI should work without any changes, platform specific part of memory mapped GPIO should be very similar to Beaglebone version.
- Port library to Stellaris/Tiva MCUs. Will require almost complete rewrite to eliminate dynamic memory allocations.
    
