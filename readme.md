BeagleBone Beowolf Cluster



## Installing

### Operating system
Follow the directions from
[ArchLinuxArm](http://archlinuxarm.org/platforms/armv7/beaglebone).
This involves downloading two files.

    wget http://archlinuxarm.org/os/omap/BeagleBone-bootloader.tar.gz
    wget http://archlinuxarm.org/os/ArchLinuxARM-am33x-latest.tar.gz


## Using

The cluster is controlled from one terminal with the program `bone` installed.
You run it like this.

    bone [-b worker-name] [command] [command arguments]

By default, commands across multiple workers and see their output. If you
specify a worker with `-b`, the command runs only on that worker.
