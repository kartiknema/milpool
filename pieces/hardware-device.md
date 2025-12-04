
# What exactly is a hardware device?
A hardware device is any physical component which interacts with the system, it could be the keyboard, or the webcam or internal components like RAM, Harddrive.

Examples: CPU, Harddisk, RAM, Keyboard, Mice, Webcam, Serial Ports, USB Serial Adapters, Network Card.

Some of these devices are built in as part of the computer chassis, however some like Webcam, Mice are external connected through USB ports. USB is just a connector, the real device is what is connected through the USB interface. For example, if we connect a mobile through USB then that mobile will be considered a hardware device with respect to the kernel.

- USB (Universal Serial Bus) is just the transport, it provides a physical connector interface, for connecting the external device and communication protocols for structuring and exchanging data. Various different kinds of USB devices can be connected, each device class has its own driver and its own device file in /dev. For example:
- Webcam has the device file /dev/video0
- USB Serial Adapter has device files of the type /dev/ttyUSB* (for example: /dev/ttyUSB0, /dev/ttyUSB1 etc.)
- Keyboard, mice (/dev/input/eventX)

Note: tty refers to terminal interfaces (the actual abbreviation is teletypewriter).

Some other device files and the devices they represent:
- /dev/mmcblk0: SD card (block device)
- /dev/ttyS0: Serial Port (more general, /dev/ttyS*) (char device)
- /dev/tty: Special file, representing the terminal for the current process.
- /dev/tty1 … n: Virtual Consoles

Virtual Consoles vs Terminal: https://askubuntu.com/questions/33078/what-is-a-virtual-terminal-for

### What about the sda* entries in /dev?
The /dev/sda, /dev/sdb/ /dev/sda1 represent block devices which host filesystems.

=> sda can be broken down into 2 parts: sd (SCSI disk / SATA disk) + “a” (meaning the first disk detected by the kernel).
- /dev/sda: Entire first disk detected by the kernel
- /dev/sdb: Second disk and so on.

=> Each disk can have multiple partitions, /dev/sda1 represents the first partition on disk sda, /dev/sda2 represents the second partition on disk sda.

The storage device in question could be HDD / SSD, USB storage device, virtual disk etc.
