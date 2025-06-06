
##  **Container Image**

A **container image** is a **lightweight, standalone, executable package** that includes everything needed to run a piece of software:

* Code or app
* Runtime (e.g., Python, Node)
* Libraries
* Environment variables
* Filesystem

> 📌 Images are immutable and layered. They become containers **when you run them**.



###  OCI Images (Open Container Initiative)

**OCI - Open Container Initiative**, a standardization initiative led by the Linux Foundation.

OCI defines two things:

* **Image format** (how container images are structured)
* **Runtime spec** (how to run them)

##### ✅ Why OCI matters?

* Podman, Docker, CRI-O, Buildah, and Kubernetes **all follow OCI standards**.
* This means images built by Docker or Podman are compatible across platforms.



###  Containerfile

A `Containerfile` is a **set of instructions** to build a container image — exactly like a `Dockerfile`.

```Dockerfile
# Containerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

> ✅ `Containerfile` is the preferred name in Podman/Buildah world (but `Dockerfile` works the same).

#### Base Image:


- Dockerfiles start from a parent image or **"base image"**
- It is a Docker image that your image is based on, the starting point of creating a containerized application.
- Contains minimal OS and runtime components needed.
- On top of that, we add your application code and dependencies to create a customized Docker image.

###  Build Image from Containerfile

Create the file:

```bash
mkdir ~/mynginx && cd ~/mynginx
echo "Hello EX188!" > index.html

# Create Containerfile
cat <<EOF > Containerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

Then build the image:

```bash
podman build -t mynginx .
```



---
###  Container Registries

#### 🔐 Logging in to Registries using Podman

To access private container registries like `quay.io` or `registry.redhat.io`, you must first authenticate using `podman login`:

```bash
# Login to Quay.io
podman login quay.io

# Login to Red Hat Registry
podman login registry.redhat.io
```

You'll be prompted to enter your **username** and **password** (for Red Hat, you can generate a service account or use credentials tied to your Red Hat subscription).



###  Unqualified Image Names & Registry Search Order

When you pull a container image **without specifying the registry**, Podman will search registries listed in `/etc/containers/registries.conf`, in the order specified by `unqualified-search-registries`.

#### Example:

```bash
podman pull ubi8/python-39
```

Here, `ubi8/python-39` is an **unqualified image name** it lacks a full registry prefix like `registry.redhat.io/ubi8/python-39`.

Podman will search the list of registries defined in
>  `/etc/containers/registries.conf`, in the specified order, to find this image.

#### Sample registries.conf search configuration:

```toml
unqualified-search-registries = ["registry.redhat.io", "docker.io"]
```

#### 🔒 Blocking a registry:

You can explicitly block Podman from pulling from certain registries, like `docker.io`, by configuring:

```toml
[[registry]]
location = "docker.io"
blocked = true
```



#### ✅ Fully Qualified:

```bash
podman pull registry.redhat.io/rhel7/rhel:7.9
```

---

### 🔍 Managing Registries and Images with `skopeo`

[`skopeo`](https://github.com/containers/skopeo) is a command-line tool to **inspect** and **copy container images** between registries without needing to pull them locally.

####  Key Features of `skopeo`

* Inspect remote container images.
* Copy images between registries.
* Convert image formats (Docker ↔ OCI).
* Sign container images using OpenPGP keys.
* Does not require a local container runtime or daemon.



###  Common Skopeo Usage Examples

#### 1. Inspect an image remotely:

```bash
skopeo inspect docker://quay.io/centos/centos:stream9
```

#### 2.  Copy image between registries:

```bash
skopeo copy docker://quay.io/centos/centos:stream9 docker://myregistry.local/centos/centos:stream9
```

#### 3. Copy image to a local directory:

```bash
skopeo copy docker://quay.io/centos/centos:stream9 dir:/tmp/image-dir
```

> `skopeo` uses a `transport:image` format such as:
>
> * `docker://image`
> * `dir:/local/path`
> * `oci:/oci/path:tag`



#### 📚 Note for Windows Users

On Windows, Podman runs in a Linux-based VM. You can SSH into it to access files like `registries.conf`.

```bash
podman machine ssh
ls /etc/containers/registries.conf
```



### `podman rmi` vs `podman prune`
```
# Remove a specific image by name
podman image rmi nginx:latest

# Remove all dangling (untagged) images
podman image prune

# Remove all unused images (dangling and tagged)
podman image prune -a
```




