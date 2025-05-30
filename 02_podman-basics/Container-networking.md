## Container Networking Basics

Podman comes with a network called podman. By default, containers are attached to this network and can use it to communicate with one another.

###  Podman Networking Basics


####  **Default Networking**

* Podman creates a **user-defined bridge network** called `podman` by default.
```
$ podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge
```

* Containers get IPs on this bridge and can access the internet through NAT.
* Containers can reach each other via IP but **not by container name** unless configured.


#### **Enabling Domain Name Resolution (DNS)**

By default, Podman **does NOT** allow container names as hostnames inside the default network.

##### How to enable container DNS resolution:

* Use the **`--network`** option with a user-defined network that supports DNS.



#### Create a user-defined network:

```bash
podman network create mynet
```

* This network supports DNS by default.


##### Run containers attached to this network:

```bash
podman run -d --name web --network mynet nginx
podman run -it --network mynet alpine sh
```

* Inside the alpine container, you can ping `web` by container name:

```bash
ping web
```


####  **Connecting Containers**

* Containers on the **same network** can communicate by **container name**.
* Use `podman network connect` to attach running containers to networks.

---

### Example: Connect an existing container to a network

```bash
$ podman network create mynet
$ podman run -d --name app1 nginx
$ podman network connect mynet app1
$ podman run -it --name app2 --network mynet alpine sh
```

Inside `app2` container:

```bash
ping app1
```


#### **Expose Ports**

To access containers from outside host:

```bash
podman run -d -p 8080:80 nginx
```

* Maps host port `8080` to container port `80`


####  **View Networks**

```bash
podman network ls
podman network inspect mynet
```


