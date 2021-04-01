---
category: program
title: Inter-process communication - Part 1
---

In one of my recent projects, it uses lots of IPC (Inter-process communication) to exchange messages between different processes.
This reminds me of a good topic to talk through.
Due to its complexity and my own experience limitation, in this article I will just summarize different ways of IPC methods.

## 1. Why do we need IPC?
Each process has its own address space. One process cannot directly access another one's space.
Therefore, if two processes want to exchange data, they must do it through kernel:
They will need a buffer located in the kernel, one process writes its data into that buffer and
the other process reads out the data from the buffer.
This mechanism provided by a kernel is called IPC (Inter-Process Communication).

## 2. Different ways for IPC
There are several ways for IPC, including pipe, FIFO, message queue, signal, and socket etc.

### 2.1. Pipe
Pipe is a buffer that locates in the kernel. It follows the first-in-first-out rule to read and write data:
One writes data into the pipe at one end, and the other reads data from the pipe at the other end.
If the buffer is full, the process which writes data will be put into sleep until the other one reads data.
Similarly, if the buffer is empty, the process which reads data will be put into sleep until there are new data to read.

Pipe is half duplex. That means, data flows in only one direction. If you need two-way communication, then you will have
to create two separate pipes. Another limitation of pipe is that it only works between processes that have a common ancestor,
e.g., two processes that have parent-child relationship or sibling relationship.

<strong>Reference:</strong>

- Advanced Programming in the UNIX Environment:
<a href="http://poincare.matf.bg.ac.rs/~ivana/courses/ps/sistemi_knjige/pomocno/apue/APUE/0201433079/ch15lev1sec2.html">15.2. Pipes</a>


### 2.2. FIFO
Because pipe can be applied only between related processes, FIFO (aka, named pipe) is used to get around this limitation.
A FIFO is associated with a pathname, and it exists in file systems. Therefore, even two unrelated processes can exchange data
by accessing a FIFO using its pathname.
Just like its name indicated, FIFO also follows first-in-first-out rule to read and write data.

Once a FIFO is created, it can be opened like a regular file with `O_RDONLY` or `O_WRONLY` flag.
Moreover, depending on whether the flag `O_NONBLOCK` is specified during opening, a FIFO can be either synchronized or asynchronized.
In the normal case (i.e., where `O_NONBLOCK` is not specified), an open for read-only blocks until some other process opens the FIFO for writing.
Similarly, an open for write-only blocks until some other process opens the FIFO for reading.
If the `O_NONBLOCK` flag is specified, an open for read-only would return immediately.
For an open for write-only, it returns -1 and set `errno` to `ENXIO` if no process has the FIFO open for reading
(`ENXIO` in this case stands for no process has the FIFO open for reading).

<strong>Reference:</strong>

- Advanced Programming in the UNIX Environment:
<a href="http://poincare.matf.bg.ac.rs/~ivana/courses/ps/sistemi_knjige/pomocno/apue/APUE/0201433079/ch15lev1sec5.html#ch15lev1sec5">15.5. FIFOs</a>


