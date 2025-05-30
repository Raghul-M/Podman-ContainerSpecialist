## Podman 

Podman is an open source tool that you can use to manage your containers locally. With Podman, you can find, run, build or deploy OCI (Open Container Initiative) containers and container images.

By default, Podman is daemonless. A daemon is a process that is always running and ready for receiving incoming requests. Some other container tools use a daemon to proxy the requests, which brings a single point of failure. In addition, a daemon might require elevated privileges, which is a security concern. Podman interacts directly with containers, images, and registries without a daemon.

Podman comes in the form of a command-line interface (CLI), which is supported for several operating systems. Along with the CLI, Podman provides two additional ways to interact with your containers and automate processes, the RESTful API and a desktop application called Podman Desktop.


---

###  **Install and Configure Podman**

#### On RHEL/CentOS/Fedora:

```bash
$ sudo dnf install -y podman
```

#### Check version:

```bash
$ podman --version
```

> 📌 **Podman is daemonless** , no service needed for rootless.



###   Create and Run Containers with Podman 

####  Basic command:

```bash
$ podman run hello-world
 [ or ]
$ podman run registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

#### Run interactive shell:

```bash
$ podman run -it ubuntu bash
```

> 📦 Pulls image if not present, runs a container.



#### **Manage Container Lifecycle**

| Task               | Command Example               |
| ------------------ | ----------------------------- |
| **Stop**           | `podman stop <container_id or container_name>`  |
| **Start**          | `podman start <container_id or container_name>` |
| **Remove**         | `podman rm <container_id or container_name>`    |
| **List All Running Containers**    | `podman ps`                   |
| **List All Containers** | `podman ps -a`                |


###  **Expose Ports and Run in Background**

####  Run NGINX in background and expose port:

```bash
$ podman run -d -p 8080:80 nginx
```

> 📌 `-d` = detach, `-p` = port forwarding (host\:container)



### **Use Environment Variables with `-e`**

####  Example with MySQL:

```bash
$ podman run -d \
  -e MYSQL_ROOT_PASSWORD=my-secret \
  -e MYSQL_DATABASE=testdb \
  mysql
```

> 🎯 `-e VAR=value` passes environment variables into the container.

---

###  **Podman Desktop (Optional GUI)**

####  GUI tool to manage images, containers, volumes, and networks.

* Download: [https://podman.io/podman-desktop](https://podman.io/podman-desktop)
* Install on Linux (Flatpak or RPM)
* Useful for: Visualizing containers , Creating pods/volumes , Seeing logs and ports 

----

### Other Commands 
You can also automatically remove a container when it exits by adding the --rm option to the podman run command.
```
$ podman run --rm registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

You can assign a name to your containers by adding the --name flag to the podman run command.
```
$ podman run --rm --name my_image registry.redhat.io/rhel7/rhel:7.9 echo 'Red Hat'
```

To remove all the unused containers 

```
$ podman rm $(podman ps -aq)
```
----


####  `podman exec`: Run commands inside a **running** container

```bash
$ podman exec [options] <container> <command>
```

##### ✅ Most useful options:

| Option | Meaning                                           |
| ------ | ------------------------------------------------- |
| `-i`   | Keep STDIN open (useful for interactive commands) |
| `-t`   | Allocate a pseudo-TTY (for shell/interactive use) |
| `-u`   | Run command as a specific user (default is root)  |

##### 📌 Examples:

```bash
# Run bash inside a container
$ podman exec -it mycontainer bash

# Run a single command
$ podman exec mycontainer ls /var/www

# Run as a different user
$ podman exec -u nginx mycontainer whoami
```



####  `podman cp`: Copy files to/from containers

```bash
podman cp <SRC> <CONTAINER>:<DEST>
podman cp <CONTAINER>:<SRC> <DEST>
```

##### 📌 Examples:

```bash
# Copy local file into container
podman cp index.html mynginx:/usr/share/nginx/html/

# Copy file from container to host
podman cp mynginx:/etc/nginx/nginx.conf ./nginx.conf
```


####  Other Must-Know `podman` Commands

| Command                            | Purpose                                 |
| ---------------------------------- | --------------------------------------- |
| `podman ps -a`                     | List all containers (running + stopped) |
| `podman start/stop <container>`    | Start/stop a container                  |
| `podman rm <container>`            | Remove a container                      |
| `podman images`                    | List all images                         |
| `podman rmi <image>`               | Remove an image                         |
| `podman logs <container>`          | Show logs from a container              |
| `podman inspect <container/image>` | Detailed JSON info                      |
| `podman diff <container>`          | Show file changes in container          |
| `podman port <container>`          | Show container port mappings            |
| `podman stats`                     | Live CPU/mem usage                      |
| `podman top <container>`           | Show processes inside container         |



####  Common Flags Recap:

| Flag     | Meaning                                          |
| -------- | ------------------------------------------------ |
| `-d`     | Detached mode (run in background)                |
| `--rm`   | Remove container after it exits                  |
| `--name` | Give container a name                            |
| `-p`     | Map ports (e.g., `-p 8080:80`)                   |
| `-v`     | Mount volume (e.g., `-v ./data:/data`)           |
| `-e`     | Set environment variables (e.g., `-e VAR=value`) |



#### 🧪 Example : Combine Everything

```bash
podman run -dit --name web -p 8080:80 -v ./html:/usr/share/nginx/html:Z nginx
podman exec -it web bash
podman cp ./newindex.html web:/usr/share/nginx/html/index.html
```

---

####  `podman pause` / `unpause`

##### Purpose : Temporarily **suspend** and **resume** container processes.

```bash
$ podman pause <container>
$ podman unpause <container>
```

> 📌 Useful when you want to "freeze" a container without stopping it.


####  `podman stop` with `--time`

```bash
$ podman stop --time=100 <container>
```

#####  What it does:

* Sends **SIGTERM**, waits up to 100 seconds, then **SIGKILL**.
* Useful when giving apps more time to gracefully shut down.

Example:

```bash
$ podman stop --time=10 myapp
```



#### 🔁 `podman restart`

```bash
$ podman restart <container>
```

* Stops (gracefully) and starts the container again.
* Useful for applying changes to config, env vars, mounted files, etc.

Example:

```bash
$ podman restart mynginx
```

---




