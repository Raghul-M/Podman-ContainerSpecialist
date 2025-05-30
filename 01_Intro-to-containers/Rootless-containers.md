### Rootless Containers 

![container image ](https://www.atatus.com/blog/content/images/size/w960/2024/04/Docker-Clean-up-1.png)

Container technology allows developers to break down application tiers (like the backend) into smaller microservices. Since containers share the host OS kernel and use fewer resources than VMs, a single host can run many containers efficiently. However, this density increases risk — compromising one container could affect the entire host. To reduce this risk, developers follow the principle of least privilege to limit access and minimize attack surfaces.


### Analyzing Rootless Containers
Rootless containers, or unprivileged containers, are containers that do not require administrator privileges.

**A container is rootless only when it meets the following conditions:**

- The containerized process does not use the root user, which is a special privileged user in Linux and UNIX systems. Such a user is for administrative purposes and has the ID 0.

- The root user inside of the container is not the root user outside of the container.

- The container runtime does not use the root user. For example, if your container runtime runs as the root user, then containers managed by such runtime are not rootless containers.




####  **Rootful vs Rootless Containers**

| Feature      | **Rootful**                                 | **Rootless**                            |
| ------------ | ------------------------------------------- | --------------------------------------- |
| Runs As      | **Root user**                               | **Non-root user**                       |
| Security     | Higher risk (has full system access)        | Safer (no root privileges on host)      |
| Setup        | Default setup (needs sudo)                  | Requires user namespaces & config       |
| Port Binding | Can bind to **<1024** ports (e.g., 80, 443) | Can only bind to **high ports** (>1024) |
| Storage      | `/var/lib/containers`                       | `$HOME/.local/share/containers`         |
| Use Cases    | System-wide services, legacy tools          | Dev/test environments, CI/CD pipelines  |



### 🔐 Key Differences

* Rootful = More powerful, more dangerous
* Rootless = More secure, but limited capabilities



### ✅ Commands 

* Use `podman` as non-root for **rootless containers**:

  ```bash
  podman run -it nginx
  ```

* For **rootful**:

  ```bash
  sudo podman run -it nginx
  ```

* Use `podman info` to verify rootless vs rootful:

  ```bash
  podman info | grep -i rootless
  ```

---

### Example 


#### 🧪 **Hands-On: Run Rootless Podman NGINX with SELinux Volume**

#####  1. **Enable and Start Podman Service**


```bash
systemctl --user enable --now podman.socket
systemctl --user status podman.socket
```

> 📌 For **rootless**, use `--user` with `systemctl`.



#####  2. **Create a New User**

```bash
sudo useradd strangerthings
sudo passwd strangerthings
```

> 🔑 Set any password you prefer when prompted.



##### 3. **Login as the New User**

```bash
su - strangerthings
```


#####  4. **Create a Directory for Website Content**

```bash
mkdir ~/nginx-data
```



#####  5. **Fix Permissions (chown inside user namespace)**

```bash
podman unshare chown -R $(id -u):$(id -g) ~/nginx-data
```

> This ensures the container can access the directory in rootless mode.



#####  6. **Create a Sample HTML Page**

```bash
echo "Welcome to the upside-down!" > ~/nginx-data/index.html
```



#####  7. **Check SELinux Status**

```bash
sestatus
```

> Ensure SELinux is `enforcing`. If it's disabled, skip the `:z` step.



#####  8. **Run NGINX Container with Volume Mount**

```bash
podman run -d \
  -p 9000:80 \
  -v ~/nginx-data:/usr/share/nginx/html:Z \
  nginx
```

> 🔁 `:Z` applies a **private SELinux label** so NGINX can access the volume.



#####  9. **Verify It’s Working**

```bash
curl http://localhost:9000
```

> You should see: `Welcome to the upside-down!`

---

#### 🛠️ Bonus: SELinux Label Check (Optional)

```bash
ls -Z ~/nginx-data
```

If SELinux blocks access, manually add context:

```bash
sudo semanage fcontext -a -t container_file_t '/home/strangerthings/nginx-data(/.*)?'
sudo restorecon -Rv ~/nginx-data
```

---

### Running Podman containers as systemd services – rootlessly and automatically on boot.


####  Why Use systemd with Podman?

In traditional Linux, services like `nginx`, `postgresql`, or `httpd` are run using `systemctl`.

Similarly, you can **run your containers as services** using `systemd` — even **without root**.


####  How to Set It Up (Rootless systemd Service)

####   Generate systemd unit file

```bash
$ podman generate systemd --name <container-name> --files
```

📌 Example:

```bash
$ podman generate systemd --name web --files
```

This creates:

```
./container-web.service
```

You should **move this file to** your systemd user service directory:

```bash
$ mkdir -p ~/.config/systemd/user
$ mv container-web.service ~/.config/systemd/user/
```


####  Reload systemd to recognize new service

```bash
$ systemctl --user daemon-reload
```



####  Start/Stop/Enable Your Container as a Service

```bash
$ systemctl --user start container-web.service
$ systemctl --user enable container-web.service
$ systemctl --user status container-web.service
```

##### ✅ Notes:

* `--user` makes it a **rootless** systemd service (for your user only).
* If you log out, by default the container will stop too.

---

### 🧷 Make Container Start Even After Logout or Reboot

By default, user services run **only when you're logged in**.

To run them on boot **regardless of login**, enable **linger**:

```bash
$ loginctl enable-linger <your-username>
```

Now the system will:

* Start your container at **system boot**
* Keep it running even after you log out

To disable:

```bash
$ loginctl disable-linger <your-username>
```

---












