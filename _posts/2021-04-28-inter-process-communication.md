---
category: program
title: Inter-process communication - Part 3
---

In my previous blogs, we have already discussed four IPC methods.
<!-- For more details, please refer to --> 
<!-- <a href="https://miaxu-src.github.io/program/2021/03/31/inter-process-communication.html">IPC - Part 1</a> -->
<!-- and <a href="https://miaxu-src.github.io/program/2021/04/19/inter-process-communication.html">IPC - Part 2</a>. -->
<!-- - Pipe -->
<!-- - FIFO -->
<!-- - Signal -->
<!-- - Semaphore -->
Today I will continue this topic by talking about the other three IPC methods:
message queue, shared memory, and sockets.


## 2. Different ways for IPC (Cont'd)
There are several ways for IPC, including pipe, FIFO, message queue, signal, and socket etc.

### 2.4. Message Queue
A message queue is a linked list of messages. It is stored in the kernel and identified by a queue ID.
To create a new message queue or open an existing queue, we can use `msgget()`.
To add a new message to the end of a queue or to fetch a message from the queue,
we can use `msgsnd()` and `msgrcv()`.

Unlike pipe and FIFO, in which messages are always read in sequence, a message can be fetched from a queue 
based on a message type, which is specified when calling `msgsnd()`. As such, they don't have to follow
the first-in first-out order, which makes it more flexible.

A message queue stays within the kernel. Once created, it will stay there until the kernel is restarted
or the queue is explicitly removed by calling `msgctl()` with its queue ID and a command parameter `IPC_RMID`.

<strong>Reference</strong>

- Advanced Programming in the UNIX Environment:
<a href="http://poincare.matf.bg.ac.rs/~ivana/courses/ps/sistemi_knjige/pomocno/apue/APUE/0201433079/ch15lev1sec7.html">15.7. Message Queues</a>


### 2.5. Shared Memory
Shared memory allows multiple processes to share a given region of memory. One process writes data into the region and others read data from the region. One of the advantages of using shared memory is that it is very efficient, because the data does not need to be copied between the client and the server. They can read or write the data directly just like reading or writing a block of memory. For other IPC forms such as pipe and message queue, the data must be copied between user spaces and kernel at both the server side and client side.

When using shared memory, all processes must be synchronized to access the region. This can be achieved by using semaphores or a lock.

There are two different ways to use shared memory:
- Using `mmap()` to map a portion of a <strong>file</strong> into the address space of a process. Once mapped into the address space, the file can be accessed just like a block of memory without using file operation functions such as `read()` and `write()`.
- Using `shmat()` to attach a <strong>memory</strong> segment under `/dev/shm/` to its address space. That memory segment can be obtained by calling `ftok()` and `shmget()`. The
  first one converts a pathname and a project identifier to an IPC key, while the second one allocates a shared memory segment using the IPC key.

<strong>Reference</strong>

- Advanced Programming in the UNIX Environment:
<a href="http://poincare.matf.bg.ac.rs/~ivana/courses/ps/sistemi_knjige/pomocno/apue/APUE/0201433079/ch15lev1sec9.html">15.9. Shared Memory</a>


### 2.6. Sockets
Unlike other IPC methods we discussed earlier, which only support communication between processes running on the same host, socket is a type of network IPC that allows processes running on
different hosts to
communicate with each other. This is also the most widely used IPC method across different platforms.

I bet you can find lots of online resources that talk about sockets, so here I will save my space and not jump into it.

<strong>Reference:</strong>

- Advanced Programming in the UNIX Environment:
<a href="http://poincare.matf.bg.ac.rs/~ivana/courses/ps/sistemi_knjige/pomocno/apue/APUE/0201433079/ch16.html">Chapter 16.  Network IPC: Sockets</a>

## 3. Summary
This concludes the series of the IPC topic. We have seen different IPC forms, including pipes, FIFO, signals, semaphores, message queues, shared memory, and sockets. There are
still lots of details to talk about for each method. However, I hope that this series can be a good start for you.
