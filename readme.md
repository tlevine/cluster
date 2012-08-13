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
    
    ssh
    status
    log

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

Run `bone status` tells you something like this.

    $ bone status
    scaphoid    Running
    lunate      Idle
    triquetrum  Running
    pisiform    Running
    trapezium   Error
    trapezoid   Running
    capitate    Running
    hamate      Idle

The left column is the hostname of the node, and the right column is one of
"Running", "Idle" and "Error".

Maybe we want to investigate this "Error". This is why we keep logs. A new log
file is started each time `bone start` is run. It is stored in
`~/$BONE_REPO/git/log/$(date -Is)`; that is, it is named according to the
second at which the run starts. It uses the date of the terminal controlling
the nodes, not the date of the node on which it is run.

`bone log` displays this log. By default, it shows the log file for the last
run only. You can pass a glob select other log files. For example,

* `bone log 2012-08-12T20:57:13-0400` selects the logs from one particular run.
* `bone log $(date -Id)*` selects all logs from today.
* `bone log *` selects all logs.

The logs are concatenated and sent to stdout. They are sorted first by filename
(date) and second by node. For example, if each worker has logs from August 12
at 8:58 pm, 9:03 pm and 9:27 pm and you select all logs from all workers, the
logs would be be printed in this order

1   8:58pm: scaphoid
    8:58pm: lunate
    8:58pm: triquetrum
    8:58pm: pisiform
    8:58pm: trapezium
    8:58pm: trapezoid
    8:58pm: capitate
    8:58pm: hamate
    9:03pm: scaphoid
    9:03pm: lunate
    9:03pm: triquetrum
    9:03pm: pisiform
    9:03pm: trapezium
    9:03pm: trapezoid
    9:03pm: capitate
    9:03pm: hamate
    9:27pm: scaphoid
    9:27pm: lunate
    9:27pm: triquetrum
    9:27pm: pisiform
    9:27pm: trapezium
    9:27pm: trapezoid
    9:27pm: capitate
    9:27pm: hamate

If you specify a particular worker,
If you do not specify a particular worker, the logs
of different workers are concatenated in the following order
 with headers indicating the current worker
