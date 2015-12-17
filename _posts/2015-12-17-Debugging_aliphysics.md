---
layout: post
title:  "Debugging Aliphysics with gdb"
categories: alice aliroot aliphysics
---

There was a nice [presentation](https://indico.cern.ch/event/463952/) (only for ALICE members) about debugging during the ALICE junior day today. While giving a nice overview of the zoo of nice tools available in software development, I found the application to the (admittedly screwed) ROOT workflow missing. Which is a bummer, because it is actually not difficult at all to use GDB with ROOT if you know how! But on the other hand, GDB can be overwhelming and starting with a tiny example program has its advantages as well. Anyhow, I would like to try to give the missing link between the talk today and how to use it with (Ali)root.

## Attach GDB to your running ROOT process
If you followed the presentation, you remember that the presented way to start gdb debugging was something along the line of:

{% highlight bash %}
$ c++ -g -o tinyprogram tinyprogram.cc
$ gdb tinyprogram
{% endhighlight bash %}

which is not applicable to ALICE analysis code where your task is within the aliphysics codebase and needs to be called from your `run.C` macro. Sure, you can compile your macros somehow but you can do it easier. You do not need to run a program with GDB from the start, you can also attach to it after it was launched. This is what we want for aliphysics development. We will need two terminals for this: one to run your macro through ROOT, and one to debug your task with GDB.

Fire up ROOT in the first terminal:

{% highlight bash %}
$ root
{% endhighlight bash %}

now that root is running, we need to tell GDB to attach to the process. For this we need the process id of the process called `root.exe` (never mind the the odd name). In the second terminal, run:

{% highlight bash %}
$ ps -u `whoami` | egrep root.exe
{% endhighlight bash %}

which will output something like

{% highlight bash %}
4755 pts/3    00:00:00 root.exe
{% endhighlight bash %}

in this case 4755 is the sought after process id. Now we can connect to GDB to this process:

{% highlight bash %}
$ gdb -p 4755
{% endhighlight bash %}

At this point, you might see an error message pointing out that you cannot connect to the process because you are lacking rights to do so. Skip to the bottom to see how to fix that - its a one liner to fix.

If everything worked as planned you are now attached to your running root instance. You will notice that you cannot enter anything in the root prompt now. This is because gdb paused the process and awaits your input in the gdb shell. So, lets tell gdb where we want to break (ie. pause and debug) our code. For example we know that something strange is happening in the file `AliObservableEtaNch.cxx` around line 48. So we want to set a breakpoint there:
{% highlight bash %}
(gdb) break AliObservableEtaNch.cxx:48
{% endhighlight bash %}
Since so far we only started root, not aliroot, GDB does not know about this file and asks if it should just add the breakpoint, in the hope that it will load the appropriate library later. Thats what we want indeed.

Now we can allow root to continue its execution, by typing `c` at the GDB prompt. Hence we can now start our analysis by calling `run.C` on the root prompt (or whatever macro you use to start your analysis locally). If everything went right, GDB should break at the first breakpoint it encounters. At this point you can do all the great stuff covered in the tutorial today. For example, type `info locals` to see every variable and its current value in the current scope! 

## Further hints

### Start your analysis with a bash script instead of ROOT shell + `run.C`
Just a few further hints which might make the use of GDB more efficient. For example, often you don't want to run your macro from the root shell but from a bash script. In this case you might want to just add

{% highlight C++ %}
  std::cin.ignore();
{% endhighlight C++ %}

somewhere in your `run.C` macro to make the macro wait for you to hit return. This gives you time to define the breakpoint before your actual analysis code starts.

### Emacs package
Furthermore, if you are an emacs user, you should check out [emacs-dgbr](https://github.com/rocky/emacs-dbgr), which allows you to step through the code in emacs making navigation much easier.

### Give yourself permission to attach to running processes on Ubuntu
If you think about the possibility to attach to a running process you will realize that this has potential security implications. Hence, this is disabled by default in current Ubuntus. To activate this feature temporarily, do:

{% highlight bash %}
$ echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
{% endhighlight bash %}

If you would like to make this change persistent, make to have the following line in in `/etc/sysctl.d/10-ptrace.conf`:
{% highlight bash %}
kernel.yama.ptrace_scope = 0
{% endhighlight bash %}

For a bit more details, check out this [post](http://askubuntu.com/questions/41629/after-upgrade-gdb-wont-attach-to-process).

