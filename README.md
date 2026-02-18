# Podman-ContainerSpecialist
Podman Notes and EX-188 Container Specialist Exam Preparation Guide and Resources 

![podman image](https://fedoramagazine.org/wp-content/uploads/2024/09/podman-2.jpg)

### ✅ **EX188 Container Specialist (EX188) — To-Do Topics Checklist**

#### Chapter 1: Introduction to Containers and OpenShift

* [X]  Understand what containers are and how they differ from VMs
* [ ]  Learn container orchestration with Kubernetes and OpenShift
* [X]  Document key differences: containers vs VMs, container images vs instances
* [X]  Important Concepts Cgroups, Namespaces, Overlay FS, SE LInux
* [X]  Rootless Containers

####  Chapter 2: Podman Basics

* [x]  Install and configure Podman
* [X]  Create and run containers using `podman run`
* [X]  Manage container lifecycle (`start`, `stop`, `rm`, `ps`, etc.)
* [X]  Expose ports, run background containers (`-p`, `-d`)
* [X]  Use environment variables with `-e`
* [X]  Explore Podman Desktop (optional GUI usage)
* [X]  **Lab**: Podman Basics

#### Chapter 3: Container Images

* [x]  Work with container registries (Red Hat, DockerHub)
* [x]  Pull, inspect, tag, and remove images
* [x]  Image cleanup and storage optimization
* [X]  **Lab**: Container Images

####  Chapter 4: Custom Container Images

* [x]  Write basic and advanced `Containerfile`s
* [x]  Use commands like `RUN`, `COPY`, `CMD`, `ENTRYPOINT`
* [x]  Build images using `podman build`
* [x]  Explore rootless Podman usage
* [X]  **Lab**: Custom Container Images

####  Chapter 5: Persisting Data

* [x]  Mount volumes (`-v`, `--mount`)
* [x]  Use bind mounts vs named volumes
* [x]  Deploy stateful services like databases
* [X]  **Lab**: Persisting Data

####  Chapter 6: Troubleshooting Containers

* [X]  View container logs
* [X]  Use `podman inspect`, `podman exec`, and `diff`
* [X]  Debug live containers and identify failures
* [X]  Remote debugging with port mapping and environment inspection
* [X]  **Lab**: Troubleshooting Containers

####  Chapter 7: Multi-container Apps with Podman Compose

* [X]  Understand `podman-compose`
* [X]  Build developer environments with Compose YAML
* [X]  Run and manage multi-service apps
* [X]  **Lab**: Multi-container Applications

####  Chapter 8: OpenShift and Kubernetes Orchestration

* [X]  Deploy containers on OpenShift using CLI and web console
* [X]  Manage pods, services, and routes
* [X]  Multi-pod application deployment
* [X]  **Lab**: Container Orchestration on OpenShift

####  Chapter 9: Comprehensive Review

* [X] Revise all core tasks from previous chapters
* [X] Practice lab scenarios without step-by-step guidance

---

### 📁 Content Structure

```bash
EX188-Podman-Prep/
├── 01_intro-to-containers/
├── 02_podman-basics/
├── 03_container-images/
├── 04_custom-container-images/
├── 05_persisting-data/
├── 06_troubleshooting/
├── 07_podman-compose/
├── 08_openshift-deployment/
├── 09_review-and-practice/
└── README.md
```

Each directory can have:

* `README.md`: Goal, key commands, explanations
* `Containerfile`: if applicable
* `notes.md`: Personal learnings, gotchas
* `script.sh`: bash/shell script 


