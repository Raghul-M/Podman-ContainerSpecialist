

## 🛠️  Troubleshooting Containers (EX188)



###  1. View Container Logs

####  Purpose:

Check what's happening inside your container (e.g., app output, errors).

#### Command:

```bash
podman logs <container-name or ID>
```

#### Live follow logs:

```bash
podman logs -f <container>
```

#### 🧪 Example:

```bash
podman run --name web -d nginx
podman logs web
```

---

###  2. Use `podman inspect` (for deep metadata)

####  Purpose:

Get full config, network, volumes, environment, runtime info, etc.

####  Command:

```bash
podman inspect <container or image>
```

#### 🧪 Example:

```bash
podman inspect web | grep -i ipaddress
podman inspect web | jq '.[0].Mounts'
```

###  Useful Fields:

* `Config.Env`
* `HostConfig.PortBindings`
* `Mounts`
* `NetworkSettings.IPAddress`
* `State.Status`

---

### 3. Use `podman exec` (interact with running container)

####  Purpose:

* Check logs, inspect files, install debug tools, run commands inside.

####  Command:

```bash
podman exec -it <container> sh
```

#### 🧪 Example:

```bash
podman exec -it web cat /etc/nginx/nginx.conf
```

> Inside the shell, you can use `ps`, `top`, `netstat`, etc.

---

###  4. Use `podman diff` (detect changes)

####  Purpose:

See **which files were added/modified/deleted** in a container compared to its original image.

####  Command:

```bash
podman diff <container>
```

#### 🧪 Example:

```bash
podman run --name testbox -it alpine sh
# Inside:
echo "hello" > /etc/test.txt
exit

# Back on host:
podman diff testbox
```

####  Output:

```bash
A /etc/test.txt      # File added
```

---

###  5. Remote Debugging with Port Mapping

####  Purpose:

Access internal services from the host (or remotely) by mapping container ports to host ports.

####  Example:

```bash
podman run -d --name app -p 8080:80 nginx
podman port < container name / ID >
```

Now visit `http://localhost:8080` — you're accessing the container's port `80`.

---

###  6. Inspect Environment Variables

####  View with `inspect`:

```bash
podman inspect <container> | jq '.[0].Config.Env'
```

####  Or from inside container:

```bash
podman exec <container> printenv
```

---

## 🧪 Bonus: Troubleshooting a Crashing Container

### Scenario:

You run a container, and it crashes immediately. You want to find out why.

#### ✅ Steps:

```bash
podman run --name myapp myimage
podman ps -a                     # See exit status
podman logs myapp                # Check logs
podman inspect myapp             # Check config/env
podman diff myapp                # Check file changes
```

> If needed, override CMD or ENTRYPOINT:

```bash
podman run -it myimage sh        # Bypass crash point
```


### ✅ Summary Cheat Sheet

| Task                       | Command                          |
| -------------------------- | -------------------------------- |
| View logs                  | `podman logs <container>`        |
| Live logs                  | `podman logs -f <container>`     |
| Inspect config/network/env | `podman inspect <container>`     |
| Enter container shell      | `podman exec -it <container> sh` |
| Detect filesystem changes  | `podman diff <container>`        |
| Map ports for debugging    | `podman run -p <host>:<cont>`    |
| List env vars              | `printenv` or `inspect -> Env`   |


### podman events — Real-Time Event Stream

 **What is it?**
podman events gives you a live event stream from the Podman service — showing you everything happening, such as:

| Event Type | Example                             |
| ---------- | ----------------------------------- |
| `create`   | A container is created              |
| `start`    | A container is started              |
| `stop`     | A container is stopped              |
| `kill`     | A container is forcefully stopped   |
| `remove`   | A container or image is deleted     |
| `exec`     | A command is run inside a container |

```
$ podman events

```
**You’ll see a stream like:**
```
2024-06-06 11:02:43.823854393 +0000 UTC container create f3a2d10c...
2024-06-06 11:02:44.222177981 +0000 UTC container start f3a2d10c...
2024-06-06 11:02:48.090283019 +0000 UTC container stop f3a2d10c...

```
**Filter By Container**
```
podman events --filter container=mycontainer
podman events --since 10m #Time based filters
```