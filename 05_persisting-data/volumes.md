

#  Persisting Data & Volumes in Podman

##  Why Persist Data?

By default, **containers are ephemeral**:

* Once a container is removed, **its filesystem is gone**.
* Logs, databases, uploads — all **lost** unless stored externally.

 To **persist data**, we use **volumes** or **bind mounts** to map data outside the container’s lifecycle.



#### 📦 Two Main Storage Types

| Type           | Description                                                                                  | Example Use Case               |
| -------------- | -------------------------------------------------------------------------------------------- | ------------------------------ |
| **Volume**     | Managed by Podman. Stored under `/var/lib/containers/storage/volumes`. Best for portability. | Databases                      |
| **Bind Mount** | Mounts a specific directory from the host into the container. Great for development.         | Mounting configs, static sites |

---

| Option     | Full Form        | Best For            | Flexibility    | Verbose/Clear?    | Example Use  |
| ---------- | ---------------- | ------------------- | -------------- | ----------------- | ------------ |
| `-v`       | `--volume`       | Quick volume mounts | Basic          | ❌ Short & cryptic | Dev/test     |
| `--volume` | Full long form   | Same as `-v`        | Basic          | ✅ Clearer         | Same as `-v` |
| `--mount`  | Structured Mount | Complex scenarios   | ✅ More control | ✅ Verbose         | Prod/scripts |
 
###   Bind Mounts (Host ↔ Container)

```bash
podman run -v /host/path:/container/path <image>
```

### Example:

```bash
mkdir ~/html
echo "Hello Raghul" > ~/html/index.html

podman run --rm -d -p 8080:80 -v ~/html:/usr/share/nginx/html:Z nginx
```

* `~/html` = Host dir
<br>
* `/usr/share/nginx/html` = NGINX web root
 <br>
* `:Z` = SELinux relabel (more below)

 Now visit `http://localhost:8080` → you'll see `"Hello Raghul"`



#### 🔐 SELinux & `:Z` / `:z`

On SELinux-enabled systems (e.g., RHEL, CentOS, Fedora):

| Modifier | Meaning                                                      |
| -------- | ------------------------------------------------------------ |
| `:Z`     | Relabel the volume for **exclusive** container use           |
| `:z`     | Relabel the volume for **shared** use by multiple containers |

 Without this, you may get `403 Forbidden` or `Permission Denied`.

---

###   Volumes (Managed by Podman)

```bash
podman volume create mydata
```

Mount into a container:

```bash
podman run -d -v mydata:/data busybox sh -c "while true; do date >> /data/time.txt; sleep 5; done"
```

Then inspect:

```bash
podman volume inspect mydata
podman run --rm -v mydata:/data busybox cat /data/time.txt
```



#### 🧹 Volume Cleanup

```bash
podman volume ls             # list volumes
podman volume rm mydata      # delete volume
podman volume prune          # remove all unused volumes
```

---

###  Comparison Summary

| Feature         | Bind Mount          | Named Volume                  |
| --------------- | ------------------- | ----------------------------- |
| Managed by      | User                | Podman                        |
| Stored at       | User-specified path | `/var/lib/containers/...`     |
| Best for        | Dev, config files   | Persistent data like DBs      |
| SELinux needed? | ✅ Use `:Z` / `:z`   | Usually handled automatically |

<br>

#### 🔍 Use Case: Persisting a Database

```bash
podman volume create pgdata

podman run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

> Even if container is removed, data in `pgdata` volume stays.



#### 📂 Inspect Volume Contents (Optional)

```bash
sudo ls /var/lib/containers/storage/volumes/pgdata/_data
```

---

###  Troubleshooting

| Issue                   | Cause                          | Fix                                  |
| ----------------------- | ------------------------------ | ------------------------------------ |
| Files not accessible    | SELinux block                  | Use `:Z`                             |
| Data lost after restart | You used a container directory | Always mount to host or named volume |
| Volume mount fails      | Wrong syntax                   | Check path & quotes                  |

<br>

#### 📁 Bonus Tip: Copy Files In/Out of Container

```bash
podman cp mycontainer:/path/in/container ./local
podman cp ./local mycontainer:/path/in/container
```

---
###  `podman run` vs `podman exec -it`

| Command             | Purpose                                                 |
| ------------------- | ------------------------------------------------------- |
| `podman run`        | **Starts a new container** from an image.               |
| `podman exec`       | **Runs a command inside an already running container.** |
| `-it` (with either) | Makes the terminal interactive (TTY + STDIN open).      |

#### ❗Quick Tip
```
podman run = Creates + Starts

podman exec = Attaches to or runs a process in an existing container
```

####  `-it` Flag in Podman
```
Meaning:

-i = Keep STDIN open (interactive input).

-t = Allocate a pseudo-TTY (terminal).
```

#### ✅ Example:

```
podman run -dit --name myos1 -v ./raghul.txt:/raghul.txt:Z busybox
```

This works fine and keeps the container running because:

> **-d** = Detached mode.

> **-it** = Allocates a terminal session.

BusyBox defaults to sh when no command is passed, and -it keeps it interactive. 

----
#### ✅  `--mount`  :More structured, less error-prone

```
podman run --mount type=bind,source=./html,target=/usr/share/nginx/html,bind-propagation=rslave,ro nginx

```
| Field               | Description                        |
| ------------------- | ---------------------------------- |
| `type=`             | `bind`, `volume`, or `tmpfs`       |
| `source=`           | Host path (or volume name)         |
| `target=`           | Container path                     |
| `readonly`          | Optional — mount read-only         |
| `bind-propagation=` | Optional (advanced mount behavior) |

----


##  `podman unshare`

####  What is it?

When using **rootless Podman**, container filesystems are stored with **UID/GID mappings** that are **invisible to your regular user**. `podman unshare` drops you into a shell with the **same UID/GID mapping** as the container — giving you full access to otherwise “restricted” files.



####  Why it Matters

* Rootless containers use **user namespaces**, and files in `/home/<user>/.local/share/containers/...` may look like:

```bash
ls -l ~/.local/share/containers/storage/volumes/myvol/_data
# Output: ?????? permission denied
```

### 🔧 Fix: Use `podman unshare`

```bash
podman unshare ls ~/.local/share/containers/storage/volumes/myvol/_data
```

Now you can:

* View files
* Copy files
* Debug volume data



#### ✅ Real-World Example

```bash
# Create and use a volume
podman volume create myvol
podman run -it -v myvol:/data alpine sh -c "echo 'Hello Raghul' > /data/hello.txt"

# Check the file from host (won't work)
cat ~/.local/share/containers/storage/volumes/myvol/_data/hello.txt  # Permission denied

# Use unshare to access it
podman unshare cat ~/.local/share/containers/storage/volumes/myvol/_data/hello.txt
```

---

### 🗃️ Part 2: Exporting and Importing Data with Volumes

This is useful for **backups**, **migrations**, and **debugging**.


#### ✅ Export Volume Data (Copy to Host)

```bash
# Use podman unshare if needed
podman unshare cp ~/.local/share/containers/storage/volumes/myvol/_data/hello.txt ./hello-backup.txt
```

Or use `podman cp` directly:

```bash
podman run --rm -v myvol:/data alpine ls /data
```

---

#### ✅ Import Data into a Volume

You can import content **into a volume** from your host using `podman cp` or `--mount`/`-v`.

```bash
# Create a volume
podman volume create myvol

# Copy a local folder into the volume via a container
podman run --rm -v myvol:/import busybox cp /imported/file /import/
```

**Or use `podman unshare`**:

```bash
podman unshare cp ./myfile.txt ~/.local/share/containers/storage/volumes/myvol/_data/
```



#### ✅ Volume Backup/Restore (Tar Approach)

```bash
# Backup volume to a tar file
podman run --rm -v myvol:/data alpine tar -czvf /data-backup.tar.gz -C /data .

# Restore volume from tar
podman run --rm -v myvol:/data -v $(pwd):/backup alpine sh -c "cd /data && tar -xzvf /backup/data-backup.tar.gz"
```



##  Summary

| Task                          | Command                            |
| ----------------------------- | ---------------------------------- |
| View rootless volume contents | `podman unshare`                   |
| Copy data from volume         | `podman cp` or `podman unshare cp` |
| Backup a volume               | `tar` inside container             |
| Restore volume from tar       | `tar` into mounted volume          |

---

