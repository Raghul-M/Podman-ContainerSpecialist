#  EX188 Exam  Additional Topics



> **⚠️ Exam Tip:** Red Hat exams focus on the "Red Hat way." While `podman-compose` is popular, the exam usually tests **Kubernetes YAML** (`podman play kube`) for multi-container orchestration.

---

## 1. Kubernetes Compatibility (`podman play kube`)


Podman can manage pods using Kubernetes-compatible YAML files. This allows you to define infrastructure locally that can be deployed to OpenShift later.

### Key Commands
| Command | Description |
| :--- | :--- |
| `podman generate kube <pod/container> > file.yaml` | Exports a running pod/container configuration to K8s YAML. |
| `podman play kube file.yaml` | Creates pods/containers from a K8s YAML file. |
| `podman play kube --down file.yaml` | Tears down the objects defined in the YAML file. |

### 🧪 Exam Scenario: Migration
**Task:** Create a pod with a web server, export it to YAML, delete the pod, and recreate it from the YAML.

```bash
# 1. Create a pod with a container
podman run -dt --pod new:mypod --name myweb -p 8080:80 nginx

# 2. Verify it's running
podman pod ps
# Output: mypod ...

# 3. Generate Kubernetes YAML
podman generate kube mypod > mypod.yaml

# 4. Inspect the YAML (It looks like standard K8s definitions)
cat mypod.yaml

# 5. Remove the original pod
podman pod rm -f mypod

# 6. Re-create the pod using the YAML
podman play kube mypod.yaml
```

### 📝 Persistent Storage in Kube YAML
If your container uses volumes, `generate kube` generally handles them, but verify the `volumes` section in the generated YAML.

```bash
# Run with volume
podman run -dt --pod new:dbpod -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres

# Generate YAML
podman generate kube dbpod > db.yaml
```

---

## 2. Quadlets (Modern Systemd Integration)


While `podman generate systemd` is deprecated (though likely still works in the exam), **Quadlets** are the new, preferred way to manage Podman containers with systemd. They are easier to write than standard unit files.

### How it works
Instead of generating a complex `.service` file, you write a simple `.container` file.

### 🧪 Quick Quadlet Example

1. **Create the Directory:**
   ```bash
   mkdir -p ~/.config/containers/systemd/
   cd ~/.config/containers/systemd/
   ```

2. **Create a `.container` file (e.g., `myserver.container`):**
   ```ini
   [Unit]
   Description=My Nginx Server
   After=network-online.target

   [Container]
   Image=docker.io/library/nginx:latest
   PublishPort=8080:80
   Volume=html-volume:/usr/share/nginx/html:Z

   [Service]
   Restart=always

   [Install]
   WantedBy=default.target
   ```

3. **Reload and Start:**
   ```bash
   systemctl --user daemon-reload
   systemctl --user start myserver
   systemctl --user status myserver
   ```

> **Note:** If the exam asks you to "configure a container to start automatically on boot," check if they specifically ask for the **deprecated** `podman generate systemd` command or if they allow any method. Quadlets are the future proof answer.

---

## 3. Image Management "Gotchas"

### Searching with specific registries
The exam often asks you to pull from specific registries.
```bash
# Search only in Red Hat registry
podman search registry.redhat.io/rhel8
```

### Skopeo Inspection
You need to inspect remote images *without* pulling them.
```bash
# Inspect specific tag
skopeo inspect docker://quay.io/bitnami/nginx:latest

# Check available tags (crucial for finding versions)
skopeo inspect docker://quay.io/bitnami/nginx:latest | grep -i "Tag"
```

---

