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

The commands are

    checkout
    start
    stop
    
    status
    log
    ssh

All of these commands ssh to the "bone" user on the node in order to run
something on the node.

`bone checkout` checks out a git repository in each worker's home directory.
For example,  `bone checkout tlevine@git.thomaslevine.com:middle-names.git`
works. Any arguments get passed to git, so you can check out a particular branch
like this: `bone checkout git://github.com/tlevine/middle-names.git dev`.
In addition to checking out the branch, `bone checkout` sets the `BONE_REPO`
environment variable in `~/.bonerc` on the worker so that the other commands
know to work on a particular repository.

`bone start` tries to run the file `~/$BONE_REPO/run`; it raises an error if
the file doesn't exist or isn't executable by .
`bone stop` know which repository to start and stop.



By default, commands across multiple workers and see their output. If you
specify a worker with `-b`, the command runs only on that worker.
