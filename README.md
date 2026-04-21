# Multi-Container Runtime in C

## Project Overview

This project implements a lightweight container runtime in C on Linux. The system consists of a long-running supervisor process and a kernel-space memory monitoring module.

Containers are implemented as isolated processes using operating system features such as namespaces, chroot, and system calls. The supervisor manages container lifecycle operations, while the kernel module monitors memory usage and enforces limits.

The project demonstrates key operating system concepts including process management, inter-process communication, synchronization, memory management, and scheduling.

---

## System Architecture

The system has two main components:

### 1. Supervisor (User Space)

* Long-running process
* Creates and manages containers
* Maintains container metadata
* Handles CLI commands
* Implements logging system

### 2. Kernel Module (Kernel Space)

* Monitors memory usage (RSS)
* Enforces soft and hard memory limits
* Communicates with supervisor using ioctl

---

## Key Features

* Multi-container support
* Process isolation using namespaces
* Filesystem isolation using chroot
* CLI-based container management
* Logging using producer-consumer model
* Kernel-level memory monitoring
* Scheduling experiments using nice values
* Proper cleanup with no zombie processes

---

## OS Concepts Demonstrated

* Process creation (clone, fork, exec)
* Process lifecycle and signals
* IPC (pipes and UNIX domain sockets)
* Synchronization (mutex, condition variables)
* Producer-consumer problem
* Memory management (RSS monitoring)
* CPU scheduling (nice values)
* Kernel-user space interaction (ioctl)

---

## Build and Setup Instructions

### Step 1: Install dependencies

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

### Step 2: Build the project

```bash
cd boilerplate
make
```

---

## Load Kernel Module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
```

---

## Prepare Root Filesystem

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base

cp -a rootfs-base rootfs-alpha
cp -a rootfs-base rootfs-beta
```

---

## Running the System

### Start Supervisor (Terminal 1)

```bash
sudo ./engine supervisor ./rootfs-base
```

---

### Start Containers (Terminal 2)

```bash
sudo ./engine start alpha ./rootfs-alpha /bin/sh
sudo ./engine start beta ./rootfs-beta /bin/sh
```

---

### View Running Containers

```bash
sudo ./engine ps
```

---

### View Logs

```bash
sudo ./engine logs alpha
```

---

### Stop Containers

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

---

### Check Kernel Logs

```bash
dmesg | tail
```

---

### Unload Module

```bash
sudo rmmod monitor
```

---

## Task-wise Explanation

### Task 1: Multi-Container Runtime

The supervisor creates containers using clone(). Each container is an isolated process with its own namespace and filesystem.

### Task 2: CLI and Control IPC

A CLI client communicates with the supervisor using UNIX domain sockets. The supervisor processes commands such as start, stop, ps, and logs.

### Task 3: Logging System

Container output is captured using pipes. A producer thread reads output and pushes it into a bounded buffer. A consumer thread writes logs to files using synchronization mechanisms.

### Task 4: Kernel Memory Monitoring

A kernel module tracks memory usage of containers. Soft limit generates warnings, while hard limit kills the container.

### Task 5: Scheduling Experiment

Containers are executed with different nice values to observe Linux scheduling behavior.

### Task 6: Resource Cleanup

All processes, threads, and kernel data structures are properly cleaned to avoid zombies and memory leaks.

---

## Engineering Analysis

### Isolation

Containers use namespaces (PID, UTS, mount) and chroot to isolate processes and filesystem.

### Process Management

Supervisor manages lifecycle using clone, exec, and signal handling.

### IPC and Synchronization

Pipes and sockets are used for communication. Mutex and condition variables ensure safe concurrent logging.

### Memory Management

Kernel module monitors RSS and enforces limits to prevent resource exhaustion.

### Scheduling

Linux CFS scheduler is used. nice values influence CPU allocation.

---

## Design Decisions and Tradeoffs

* Used clone instead of fork for namespace support
* Used chroot for simplicity instead of pivot_root
* Used pipes for logging due to simplicity and efficiency
* Used kernel module for accurate memory enforcement

---

## Demo Instructions (Important for Evaluation)

Use two systems:

### System 1 (Code Explanation)

* Show engine.c and monitor.c
* Explain clone, chroot, logging, and kernel module

### System 2 (Execution Demo)

* Run supervisor
* Start multiple containers
* Show ps output
* Show logs
* Demonstrate memory limit violation
* Show scheduling behavior using nice values
* Show clean shutdown

---

## Expected Output Highlights

* Multiple containers running simultaneously
* Logs generated for each container
* Memory warnings and kills visible in dmesg
* Different CPU behavior based on nice values
* No zombie processes after shutdown

---

## Conclusion

This project successfully demonstrates how container runtimes work internally using fundamental operating system concepts. It provides a clear understanding of process isolation, memory management, IPC, and scheduling in Linux systems.

---
