

## What is a Custom Image?

A **custom container image** is a user-built image tailored to your application or environment needs. Instead of pulling a public image, you create your own with specific software, configurations, and dependencies baked in.



####  Containerfile (Dockerfile Equivalent)

* `Containerfile` is the default filename for container image build instructions in Podman/Buildah.
* It’s functionally equivalent to a `Dockerfile`.
* Defines **instructions** for building a custom container image layer by layer.

#### Basic Structure of a Containerfile

```Dockerfile
# Base image
FROM registry.redhat.io/ubi8/ubi:8.6

# Maintainer information (optional)
LABEL maintainer="you@example.com"

# Arguments (build-time variables)
ARG APP_VERSION=1.0

# Environment variables (runtime)
ENV APP_HOME=/app

# Copy files from local to image
COPY myapp/ $APP_HOME/

# Set working directory
WORKDIR $APP_HOME

# Run commands to install packages or dependencies
RUN dnf install -y python3 && dnf clean all

# Expose ports
EXPOSE 8080

# Specify the command to run when container starts
CMD ["python3", "app.py"]
```

#### Choosing a Base Image
When you build an image, podman executes the instructions in the Containerfile and applies the changes on top of a container base image. A base image is the image from which your Containerfile and its resulting image is built.

**The base image you choose determines the Linux distribution and its components, such as:**

- Package manager
- Init system
- Filesystem layout
- Preinstalled dependencies and runtimes

The base image can also influence other factors, such as **image size**, **vendor support**, and **processor compatibility**.

Red Hat provides container images intended as a common starting point for containers known as **universal base images (UBI)**. These UBIs come in **four variants: standard, init, minimal, and micro**. Additionally, Red Hat also provides UBI-based images that include popular runtimes, such as Python and Node.js.

These UBIs use Red Hat Enterprise Linux (RHEL) at their core and are available from the Red Hat Container Catalog. The main differences are as follows:

**1. Standard** : This is the primary UBI, which includes DNF, systemd, and utilities such as gzip and tar.

**2. Init** : Simplifies running multiple applications within a single container by managing them with systemd.

**3. Minimal** : This image is smaller than the init image and still provides nice-to-have features. This image uses the microdnf minimal package manager instead of the full-sized version of DNF.

**4. Micro** : This is the smallest available UBI because it only includes the bare minimum number of packages. For example, this image does not include a package manager.


---

#### Build Arguments (`ARG`)

* Defined in the `Containerfile` using `ARG`.
* Values can be overridden at build time with `--build-arg`.

 **Example:**

```Dockerfile
ARG APP_VERSION=1.0
RUN echo "Building version $APP_VERSION"
```

**Build command:**

```bash
podman build --build-arg APP_VERSION=2.5 -t myapp:2.5 .
```
Sure! Here's your updated **Key Instructions and Concepts** table with `USER`, `VOLUME`, `ENTRYPOINT`, and `ADD` added, with concise descriptions for each:

---

### Key Instructions and Concepts

| Instruction  | Description                                                                            |
| ------------ | -------------------------------------------------------------------------------------- |
| `FROM`       | Specifies the base image. Required as the first instruction.                           |
| `LABEL`      | Adds metadata (author, version, description).                                          |
| `ARG`        | Defines build-time variables (can be overridden when building).                        |
| `ENV`        | Defines environment variables inside the image, accessible at runtime.                 |
| `COPY`       | Copies files or directories from the build context to the image filesystem.            |
| `ADD`        | Similar to `COPY` but can also unpack local tar archives and download files from URLs. |
| `WORKDIR`    | Sets the working directory for subsequent instructions.                                |
| `RUN`        | Executes commands during image build (e.g., installing software).                      |
| `EXPOSE`     | Declares ports that the container listens on (for documentation).                      |
| `USER`       | Specifies the user (and optionally group) to run the container processes as.           |
| `VOLUME`     | Defines mount points for external storage volumes inside the container.                |
| `ENTRYPOINT` | Configures a container’s executable that will always run (can be overridden).          |
| `CMD`        | Default command and arguments to execute when the container runs (can be overridden).  |

---
**eg:**
```
# This is a comment line 
FROM        registry.redhat.io/ubi8/ubi:8.6 
LABEL       description="This is a custom httpd container image" 
RUN         yum install -y httpd 
EXPOSE      80 
ENV         LogLevel "info" 
ADD         http://someserver.com/filename.pdf /var/www/html 
COPY        ./src/   /var/www/html/ 
USER        apache 
ENTRYPOINT  ["/usr/sbin/httpd"] 
CMD         ["-D", "FOREGROUND"] 
```

#### Building a Custom Image

 **Command to build image from Containerfile:**

```bash
podman build -t myapp:latest .
```

* `-t myapp:latest` tags the image as `myapp` with tag `latest`.
* `.` is the build context (current directory).

 **Additional useful flags:**

* `--file` or `-f`: specify a different Containerfile name or path.
<br>

  ```bash
  podman build -f CustomFile -t myapp:latest .
  ```

* `--no-cache`: build without using cached layers.
<br>

  ```bash
  podman build --no-cache -t myapp:latest .
  ```

* `--pull`: always attempt to pull a newer version of the base image.
<br>
  ```bash
  podman build --pull -t myapp:latest .
  ```

---

#### Best Practices for Custom Images

* Use **minimal base images** (e.g., UBI, Alpine) to reduce size.

* Clean up cache and package manager metadata to reduce layers and image size:

  ```Dockerfile
  RUN dnf install -y python3 && dnf clean all
  ```

* Combine multiple `RUN` commands with `&&` to reduce image layers.

* Use multi-stage builds if you need to compile code but want a smaller final image.

* Always specify versions in `FROM` and packages for reproducibility.

* Avoid including secrets or credentials in images.



#### Example: Complete Containerfile

```Dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.6

LABEL maintainer="raghul@example.com"
ARG APP_VERSION=1.0

ENV APP_HOME=/opt/myapp \
    APP_VERSION=$APP_VERSION

COPY src/ $APP_HOME/

WORKDIR $APP_HOME

RUN dnf install -y python39 && dnf clean all

EXPOSE 5000

CMD ["python3", "app.py"]
```

Build command with build-arg:

```bash
podman build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
```



#### Useful Commands Summary

| Command                                                    | Description                   |
| ---------------------------------------------------------- | ----------------------------- |
| `podman build -t myimage:tag .`                            | Build an image with tag       |
| `podman build --no-cache .`                                | Build image without cache     |
| `podman build --pull .`                                    | Always pull latest base image |
| `podman images`                                            | List local images             |
| `podman run myimage:tag`                                   | Run container from image      |
| `podman push myimage:tag registry.example.com/myimage:tag` | Push image to remote registry |

---


