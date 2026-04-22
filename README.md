# Mini Container Runtime (Jackfruit Project)

## Overview

This project implements a minimal container runtime in user space using a supervisor-based architecture. It demonstrates core concepts behind containerization, including process isolation, inter-process communication (IPC), and resource monitoring.

---

## Architecture

```
CLI (engine commands)
        ↓
UNIX Domain Socket (IPC)
        ↓
Supervisor Process
        ↓
Container Processes (chrooted)
        ↓
Logs + Kernel Monitor
```

---

## Features

* Container creation using `fork()` and `chroot()`
* Supervisor-based container management
* UNIX domain socket IPC for control-plane communication
* Per-container logging (`logs/<id>.log`)
* Container lifecycle management:

  * start
  * run
  * ps
  * stop
* Kernel monitor integration via `/dev/container_monitor`

---

## Commands

### Start Supervisor

```bash
sudo ./engine supervisor rootfs-base
```

### Start Container

```bash
sudo ./engine start <id> <rootfs> <command>
```

Example:

```bash
sudo ./engine start alpha rootfs-alpha /cpu_hog
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

### IPC

* Implemented using UNIX domain sockets (`/tmp/mini_runtime.sock`)
* Client sends `control_request_t`
* Supervisor processes and responds

### Supervisor

* Long-running process
* Maintains container metadata (linked list)
* Handles all lifecycle operations

### Container Execution

* Uses `fork()` + `chroot()` for isolation
* Executes command inside container filesystem

### Logging

* Output redirected using `dup2()`
* Stored in per-container log files

### Monitoring

* Uses ioctl interface to communicate with kernel module
* Registers container PID and memory limits

---

## Limitations

* Uses `chroot()` instead of full namespace isolation
* No persistent storage of container state across supervisor restarts
* Simplified logging (no bounded buffer implementation)

---

## Future Improvements

* Use `clone()` with namespaces (PID, mount, UTS)
* Implement bounded buffer logging
* Add persistent metadata storage
* Improve signal handling and graceful shutdown

---

## Author

Nyssa Bansal
