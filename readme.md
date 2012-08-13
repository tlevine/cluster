Bone
======================
Bone makes distributed computations easy for Tom. Here are some properties.

* It feels unixy.
* It is general and low-level.
* It was designed for Tom's BeagleBone Beowolf cluster.

More specifically,

* It is for distributed computation, not for distributed storage; you have to
    figure out where to store non-temporary data.
* Bone lends itself to the
    [divide and conquer](http://zguide.zeromq.org/page:all#Divide-and-Conquer)
    pattern by making it easy to deploy workers. It does not help you deploy
    ventilators or sinks.
* Bone jobs can be run without the cluster and without bone; this facilitates
    quick iteration loops (on the order of seconds rather than half a minute)
    and portability.

## Installing

### Operating system
Follow the directions from
[ArchLinuxArm](http://archlinuxarm.org/platforms/armv7/beaglebone).
This involves downloading two files.

    wget http://archlinuxarm.org/os/omap/BeagleBone-bootloader.tar.gz
    wget http://archlinuxarm.org/os/ArchLinuxARM-am33x-latest.tar.gz

Then you need to partition the microSD card, format it and extract the
archives.

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

### Configure workers
A Bone worker corresponds to a user account on a POSIX system. To add workers
to Bone, add the username and hostname to the `WORKERS` array in `~/.bonerc`
on the box from which you are controlling the workers. (I call mine "desk".)
If you have several cores on one computer, you might want to create two bone
accounts on one computer in order to use its multiple cores.

    $ cat ~/.bonerc
    WORKERS=( bone@192.168.1.103 bone@triquetrum bone@foo.thomaslevine.com bone2@triquetrum )

All of Bone's communication with nodes happens over SSH, so the other part of
the configuration is making sure that SSH is running on the workers and that
one of your desk's SSH keys is authorized on each worker. If you're using
Beaglebones, you can do this by copying the public key when you image the
microSD cards.

### Running
The cluster is controlled from one terminal with the program `bone` installed.
You run it like this.

    bone [-b worker-name] [command] [command arguments]

The commands are

    checkout
    install
    start
    stop
    
    ssh
    status
    log

All of these commands ssh to the bone user on the node in order to run
something on the node.

By default, commands across multiple workers and see their output. If you
specify a worker with `-b`, the command runs only on that worker.

`bone checkout` checks out a git repository in each worker's home directory.
For example,  `bone checkout tlevine@git.thomaslevine.com:middle-names.git`
works. Any arguments get passed to git, so you can check out a particular branch
like this: `bone checkout git://github.com/tlevine/middle-names.git dev`.
First, `bone checkout` sets the `BONE_REPO` environment variable in
`~/.bone-state` on the worker so that the other commands know to work on a
particular repository. Then, it checks out the branch to `~/$BONE_REPO/git/`.

`bone start` tries to run the file `~/$BONE_REPO/git/install`; it raises an error
if the file doesn't exist or if the bone user on the worker can't execute it.
You might want to put an installation script in this file.

`bone start` tries to run the file `~/$BONE_REPO/git/run`; it raises an error if
the file doesn't exist or if the bone user on the worker can't execute it.
`bone stop` stops the running of `~/$BONE_REPO/git/run`. You may want to call
the computation of interest from this file.

`bone ssh [shell command]` runs the shell command on the worker or workers. If
you don't specify a command but do specify a worker, you open a terminal on
that worker. If you specify neither a worker nor a command, you get an error.

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

 1.  8:58pm: scaphoid
 2.  8:58pm: lunate
 3.  8:58pm: triquetrum
 4.  8:58pm: pisiform
 5.  8:58pm: trapezium
 6.  8:58pm: trapezoid
 7.  8:58pm: capitate
 8.  8:58pm: hamate
 9.  9:03pm: scaphoid
11.  9:03pm: lunate
12.  9:03pm: triquetrum
13.  9:03pm: pisiform
14.  9:03pm: trapezium
15.  9:03pm: trapezoid
16.  9:03pm: capitate
17.  9:03pm: hamate
18.  9:27pm: scaphoid
19.  9:27pm: lunate
20.  9:27pm: triquetrum
21.  9:27pm: pisiform
22.  9:27pm: trapezium
23.  9:27pm: trapezoid
24.  9:27pm: capitate
25.  9:27pm: hamate

Section headings inside the output indicate the datetime and node to which each
log section applies.

If you specify a particular worker, the same ordering applies, except that
only that worker's logs will be displayed.
