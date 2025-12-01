## Containerization : 

Containerization is a lightweight form of virtualization that packages an application and all its dependencies (libraries, binaries, config files, etc.) into a single unit called a container. This ensures the application runs consistently across different environments — from development to production.

![Containers vs virtual machines](https://static.ole.redhat.com/rhls/courses/do188-4.14/images/overview/intro/assets/images/overview/container_vs_os.svg)

- It is a lightweight virtualization technology alternative to hypervisor virtualization.
- Any application can be bundled in a container can run without any worries about dependencies, libraries and binaries.
- So, containers are designed to run on physical servers, virtual machines and any cloud instances.
- Also container was designed to solve modern problems and application management issues. so it is not a replacement for virtualization, but it’s complementary to it.

### Container:



A container is a lightweight, isolated environment that runs an application along with everything it needs  like libraries and dependencies  without depending on the host system. It uses the host operating system's kernel but keeps the app's files separate.

#### Key points about containers:

- They run in their own space but share the host OS kernel.
- They are created from container images.
- They start quickly, use fewer resources, and are more portable than virtual machines (VMs).


> 📌 Think of a container as a small, self-contained box that runs your app  like a mini-sandbox  without the heavy setup of a full operating system like a VM.


#### Container Engine vs Container Runtime


| Term           | Description                                                                                                          |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Container Engine**  | CLI / tool users interact with to build, run, and manage containers.  e.g., Docker CLI, Podman CLI.(User-facing tool that orchestrates everything.)                           |
| **Container Runtime** | Low-level tool responsible for creating and running containers using Linux kernel features(namespaces, cgroups, seccomp, etc..)  e.g., runc, cri-o, containerd. |




### Container Engine

What it does:
- Provides a CLI or API interface.
- Pulls container images.
- Builds new images.
- Runs and manages containers.
- Talks to the runtime to actually execute the container.


> 📌 Examples: Podman , Docker, CRI-O (used in OpenShift — container runtime interface only)


### Container Runtime

What it does:
- Actually starts the container process.
- Uses kernel features like namespaces, cgroups, seccomp, and SELinux.
- Enforces isolation and resource control.


> 📌 Examples: runc (used by Podman and Docker) , crun (faster, written in C, default in newer Podman) , containerd (used in Docker and Kubernetes) , CRI-O (used in OpenShift)



> 📌  Podman uses runc or crun under the hood, while OpenShift uses CRI-O, which in turn also uses runc or crun.


----- 

### Important Concepts
  


####  1. Control Groups (cgroups)

Cgroups (Control Groups) is a Linux kernel feature for limiting, accounting, and isolating the resource usage (CPU, memory, disk I/O, etc.) of a group of processes. It helps allocate resources among user-defined groups efficiently and ensures that one group's resource usage doesn't adversely affect others . Control how much resources **(CPU, memory, I/O, etc.)** a process (like a container) can use.

**Example**:
Limit a container to 256MB of memory or 0.5 CPU cores.

**Key Points**:

* Containers use **cgroups v2** via `systemd`.
* Used in **Podman** and **CRI-O** to manage resource limits.
* Relevant options:

  ```bash
  $ podman run --memory=256m --cpus=0.5 ...
  ```

**Internals**:

* cgroups are mounted under `/sys/fs/cgroup/`
* Each container gets its own cgroup to track and restrict usage.

---

#### 2. Namespaces

Namespaces are a Linux kernel feature that provides isolated environments for processes. Each namespace has its own global system resources (process IDs, network interfaces, file system mounts, etc.), ensuring that processes in different namespaces do not interfere with each other. This isolation is essential for containerization, allowing multiple containers to run on the same host without conflicts. Provide **process isolation** by giving each container its own view of system resources.

**Types of Namespaces** :

* `pid`: Process IDs (each container has its own PID 1)
* `mnt`: Filesystem mount points
* `net`: Network interfaces (e.g., `eth0` inside the container)
* `uts`: Hostname
* `ipc`: Interprocess communication
* `user`: User and group IDs

**Used for**:

* Creating isolated environments for containers.
* Behind the scenes of tools like `podman`, `buildah`.

**Check namespaces of a container**:

```bash
$ lsns -p <PID>
```

---

####  3. OverlayFS

OverlayFS (Overlay Filesystem) is a union mount filesystem implementation in Linux that allows multiple filesystems to be stacked on top of one another, creating a single, unified view. It's often used in containerization platforms like Docker, Podman to provide a writable layer on top of a read-only base filesystem. A **union filesystem** that allows read-only base image + writable layer for changes. This enables **image layering**.

**Used in**:

* Podman, Buildah, CRI-O, and Docker.
* Each container image is a set of layers.
* Final writable layer is ephemeral (deleted when container is deleted).

**Structure**:

* `lowerdir`: base image layers (read-only)
* `upperdir`: container changes (read-write)
* `workdir`: internal work directory for OverlayFS

**Example**:

```bash
$ podman run -it fedora /bin/bash # Changes you make go to the upperdir
```

**Check storage drivers**:

```bash
$ podman info | grep -i overlay
```

---

####  4. SELinux (Security-Enhanced Linux)

Adds a **mandatory access control** layer for enhanced security.

**Use in Containers**:

* Ensures containers can't access host files unless allowed.
* Adds security labels to container processes and volumes.

**Key SELinux Modes**:

* `enforcing`: Policy is enforced (default in RHEL)
* `permissive`: Policy is not enforced, but violations are logged
* `disabled`: SELinux is off

**Typical issues**:

* Container can't write to mounted host directory due to SELinux.
* Fix with `:z` or `:Z` label:

  ```bash
  $ podman run -v /data:/app:z ...
  ```

**Check SELinux context**:

```bash
$ ls -Z /path
```

**Tip**:

* Always remember SELinux may block volume mounts without proper labeling.



🔐 **SELinux Volume Mounts in Podman**

##### `:Z` vs `:z` :

| Modifier | Meaning                                 | Use Case                         |
| -------- | --------------------------------------- | -------------------------------- |
| `:Z`     | Relabel for **exclusive** container use | Private volumes                  |
| `:z`     | Relabel for **shared** container use    | Shared volumes across containers |

#####  Without `:Z` or `:z`:

```You may get Permission denied / 403 Forbidden errors due to SELinux blocking access.```

#### ✅ Tip:

```bash
$ podman run -v /host/dir:/container/dir:Z ...
```

Use `:Z` on **SELinux systems** (RHEL, Fedora, etc.) to avoid permission issues.

---




### ✅ Summary Table

| Feature    | Purpose                             | Usage in Containers                            |
| ---------- | ----------------------------------- | ---------------------------------------------- |
| cgroups    | Resource limits (CPU, memory, etc.) | `--cpus`, `--memory` in `podman run`           |
| namespaces | Process isolation                   | Each container has its own PID/net/FS space    |
| OverlayFS  | Union filesystem (image layers)     | Image = base layers + writable container layer |
| SELinux    | Enforces security policies          | Use `:z`/`:Z` for volume mount labels          |

----

