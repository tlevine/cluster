BeagleBone Beowolf Cluster



## Installing

### Operating system
Follow the directions from
[ArchLinuxArm](http://archlinuxarm.org/platforms/armv7/beaglebone).
This involves downloading two files.

    wget http://archlinuxarm.org/os/omap/BeagleBone-bootloader.tar.gz
    wget http://archlinuxarm.org/os/ArchLinuxARM-am33x-latest.tar.gz


## Using

### Repository layout
Control your code versions in a git repository with a layout like this

    ./
      .gitmodules
      LICENSE
      README
      dev-run
      schema.sql
      worker/
        run
        foo.r
        bar.py
        baz.jl

The main point is that the directory should contain a submodule to be checked
out on the worker that contains a file called "run". Secondarily, you might
want to write a "dev-run" script that runs the worker in a non-parallelized
manner so that you can develop without pushing .

### Running
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

By default, commands across multiple workers and see their output. If you
specify a worker with `-b`, the command runs only on that worker.

`bone checkout` checks out a git repository in each worker's home directory.
For example,  `bone checkout tlevine@git.thomaslevine.com:middle-names.git`
works. Any arguments get passed to git, so you can check out a particular branch
like this: `bone checkout git://github.com/tlevine/middle-names.git dev`.
First, `bone checkout` sets the `BONE_REPO` environment variable in `~/.bonerc`
on the worker so that the other commands know to work on a particular repository.
Then, it checks out the branch to `~/$BONE_REPO/git/`.

`bone start` tries to run the file `~/$BONE_REPO/git/run`; it raises an error if
the file doesn't exist or if the "bone" user on the worker can't execute it.
`bone stop` stops the running of `~/$BONE_REPO/git/run`

`bone ssh [shell command]` runs the shell command on the worker or workers. If
you don't specify a command but do specify a worker, you open a terminal on
that worker. If you specify neither a worker nor a command, you get an error.
Or maybe you open up all of the workers in a tmux....

One reason you might use this is to copy credentials to the workers. For
example, you might want them to track a git repository on the local network
rather than on the internet, so you might want to give them all an ssh keys.
That's easy enough with `bone ssh "echo $(cat id_rsa) > .ssh/id_rsa"`.
