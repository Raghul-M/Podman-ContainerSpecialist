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
* [ ]  **Lab**: Podman Basics

#### Chapter 3: Container Images

* [ ]  Work with container registries (Red Hat, DockerHub)
* [ ]  Pull, inspect, tag, and remove images
* [ ]  Image cleanup and storage optimization
* [ ]  **Lab**: Container Images

####  Chapter 4: Custom Container Images

* [ ]  Write basic and advanced `Containerfile`s
* [ ]  Use commands like `RUN`, `COPY`, `CMD`, `ENTRYPOINT`
* [ ]  Build images using `podman build`
* [ ]  Explore rootless Podman usage
* [ ]  **Lab**: Custom Container Images

####  Chapter 5: Persisting Data

* [ ]  Mount volumes (`-v`, `--mount`)
* [ ]  Use bind mounts vs named volumes
* [ ]  Deploy stateful services like databases
* [ ]  **Lab**: Persisting Data

####  Chapter 6: Troubleshooting Containers

* [ ]  View container logs
* [ ]  Use `podman inspect`, `podman exec`, and `diff`
* [ ]  Debug live containers and identify failures
* [ ]  Remote debugging with port mapping and environment inspection
* [ ]  **Lab**: Troubleshooting Containers

####  Chapter 7: Multi-container Apps with Podman Compose

* [ ]  Understand `podman-compose`
* [ ]  Build developer environments with Compose YAML
* [ ]  Run and manage multi-service apps
* [ ]  **Lab**: Multi-container Applications

####  Chapter 8: OpenShift and Kubernetes Orchestration

* [ ]  Deploy containers on OpenShift using CLI and web console
* [ ]  Manage pods, services, and routes
* [ ]  Multi-pod application deployment
* [ ]  **Lab**: Container Orchestration on OpenShift

####  Chapter 9: Comprehensive Review

* [ ] Revise all core tasks from previous chapters
* [ ] Practice lab scenarios without step-by-step guidance
* [ ] Perform container-to-OpenShift migration mentally and practically

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


