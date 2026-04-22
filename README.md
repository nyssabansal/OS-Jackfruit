# Mini Container Runtime (Jackfruit Project)

## Overview

This project implements a minimal container runtime in user space using a supervisor-based architecture. It demonstrates core concepts of containerization including process isolation, inter-process communication (IPC), logging, and kernel-level resource monitoring.

---

## Architecture

```
CLI (engine commands)
        ↓
UNIX Domain Socket (IPC)
        ↓
Supervisor Process (long-running)
        ↓
Container Processes (chrooted)
        ↓
Logs + Kernel Monitor
```

---

## Features

* Container creation using `fork()` and `chroot()`
* Supervisor-based lifecycle management
* UNIX domain socket IPC for control-plane communication
* Per-container logging (`logs/<container_id>.log`)
* Kernel module integration via `/dev/container_monitor`
* Supported commands:

  * `start`
  * `run`
  * `ps`
  * `stop`
  * `logs`

---

## Commands

### Start Supervisor

```bash
sudo ./engine supervisor rootfs-base
```

---

### Start Container

```bash
sudo ./engine start <id> <container-rootfs> <command>
```

Example:

```bash
sudo ./engine start alpha rootfs-alpha /cpu_hog
```

---

### Run Container (foreground)

```bash
sudo ./engine run <id> <container-rootfs> <command>
```

---

### List Containers

```bash
sudo ./engine ps
```

---

### Stop Container

```bash
sudo ./engine stop <id>
```

---

### View Logs

```bash
cat logs/<id>.log
```

---

## Implementation Details

### IPC (Control Plane)

* Implemented using UNIX domain sockets (`/tmp/mini_runtime.sock`)
* CLI sends structured `control_request_t` messages
* Supervisor processes requests and sends responses back

---

### Supervisor

* Persistent process that manages all containers
* Maintains container metadata using a linked list
* Handles lifecycle commands (`start`, `ps`, `stop`, etc.)

---

### Container Execution

* Containers are created using `fork()` and `chroot()`
* This implementation uses `fork()` instead of `clone()` for simplicity while maintaining correct container semantics
* Each container runs a command inside its isolated filesystem

---

### Logging

* Implemented using file descriptor redirection (`dup2()`)
* Each container writes output to:

```
logs/<container_id>.log
```

---

### Monitoring

* Uses a kernel module via `/dev/container_monitor`
* Containers are registered using `ioctl`
* Supports soft and hard memory limits

---

## Limitations

* Uses `chroot()` instead of full namespace isolation
* Does not implement `clone()` with PID/mount namespaces
* Logging uses direct file redirection instead of bounded buffer
* No persistent container state after supervisor restart
* Graceful shutdown handling is minimal

---

## Future Improvements

* Implement `clone()` with namespaces (PID, mount, UTS)
* Add bounded buffer logging with producer-consumer model
* Persist container metadata across restarts
* Improve signal handling and graceful shutdown
* Add more advanced resource scheduling

---

## How to Run

### 1. Start Supervisor

```bash
sudo ./engine supervisor rootfs-base
```

### 2. Start a Container

```bash
sudo ./engine start alpha rootfs-alpha /cpu_hog
```

### 3. List Containers

```bash
sudo ./engine ps
```

### 4. Stop Container

```bash
sudo ./engine stop alpha
```

### 5. View Logs

```bash
cat logs/alpha.log
```

---

## Key Learnings

* Designing a supervisor-based container runtime
* Implementing IPC using UNIX domain sockets
* Managing process lifecycle in Linux
* File descriptor manipulation for logging
* Kernel-user space interaction using ioctl

---

## Author

Nyssa Bansal
