

#  Orchestrating Containers with `podman-compose`

##  What is `podman-compose`?

* It's the Podman-compatible version of Docker Compose.
* It lets you define **multi-container applications** in a single `docker-compose.yml` file.
* Uses **Podman pods** under the hood to group containers.



####  Why Use Podman Compose?

| Feature                      | Benefit                               |
| ---------------------------- | ------------------------------------- |
| Declarative setup            | All services defined in one file      |
| Automatically creates pods   | Groups containers with shared network |
| Supports volumes, ports, env | Just like Docker Compose              |
| Podman-native                | Works without Docker daemon           |



#### 📦 Install `podman-compose`

#### On RHEL, Fedora, CentOS:

```bash
sudo dnf install -y python3-pip
pip3 install --user podman-compose
```

#### On Ubuntu:

```bash
sudo apt install podman-compose
```

---

###  Example: Web + Redis

###  `docker-compose.yml`

```yaml
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "8080:80"

  redis:
    image: redis
```

#### The Compose File

The Compose file is a YAML file that contains the following sections:

- **version (deprecated) :** Specifies the Compose version used.

- **services:** Defines the containers used.

- **networks:** Defines the networks used by the containers.

- **volumes:** Specifies the volumes used by the containers.

- **configs:** Specifies the configurations used by the containers.

- **secrets:** Defines the secrets used by the containers.

> The **secrets** and **configs** objects are mounted as a file in the containers.


####  Run It:

```bash
podman-compose up -d
```

✅ This:

* Creates a **pod**
* Adds `web` and `redis` containers
* Shares the pod’s network (can use container names to connect)



####  Inspect What Was Created

```bash
podman pod ls              # See the new pod
podman ps                  # See containers running
podman pod inspect <id>    # See container list in the pod
```



####  Stop & Clean Up

```bash
podman-compose down
```



#### 🧪 Add Volume & Env Example

```yaml
version: "3.9"
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: mypass
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Now run:

```bash
podman-compose up -d
```

Data will persist inside the `pgdata` volume even if the container is removed.


### Pod Example 

```
# Create a pod
podman pod create --name mypod -p 8080:80

# Add nginx to the pod
podman run -d --name web --pod mypod nginx

# Add busybox to the same pod
podman run -dit --name debug --pod mypod busybox

# Now inside `debug`, access nginx via localhost
podman exec -it debug sh
wget -qO- http://localhost

```

---

##  Container vs Pod — Simplified

### 📦 What is a **Container**?

A **container** is:

* A single, isolated unit that runs **one application or service**.
* Has its **own filesystem**, **network interface**, and **process space**.
* Created from an **image** (like `nginx`, `python`, etc.).

> 🧠 Think of it like **one room** with one worker inside it — clean, isolated, focused.



### 🧩 What is a **Pod**?

A **pod** is:

* A group of **one or more containers** that **share**:

  * The same **network namespace** (IP address, ports)
  * The same **storage volumes** (if configured)
  * The same **hostname**
* All containers inside a pod can talk to each other over `localhost`.

> 🧠 Think of a pod like a **shared apartment** where several roommates (containers) live together and share the **same Wi-Fi and kitchen** (network and storage).

---

### 🔁 Key Differences

| Feature      | Container                | Pod (Podman or Kubernetes)             |
| ------------ | ------------------------ | -------------------------------------- |
| Runs         | A single application     | One or more tightly-coupled containers |
| Network      | Own isolated IP & ports  | Shared IP and ports between containers |
| Volumes      | Configured per container | Can be shared across containers        |
| Use Case     | Simple services          | Complex services that need co-location |
| Created with | `podman run`             | `podman pod create + podman run --pod` |


## 🤔 When to Use a Pod?

Use a **pod** when:

* You need **multiple containers to tightly collaborate**.
* E.g., an app container and a logging sidecar or a helper like Redis exporter.

---

