## Multistage Build in Containerfile

A multistage build uses multiple FROM instructions to create multiple independent container build processes, also called stages. Every stage can use a different base image and you can copy files between stages. The resulting container image consists of the last stage.


####  What is a Multistage Build?

A **Multistage Build** lets you use **multiple `FROM` instructions** in a single Containerfile to create lightweight and secure images by:

* **Building** code in one stage (with all build tools and dependencies).
* **Copying only the required artifacts** (like binaries) into a smaller base image.
* **Reducing final image size** by avoiding unnecessary build tools.


####  Why Use Multistage Builds?

| Benefit                     | Description                                           |
| --------------------------- | ----------------------------------------------------- |
| Smaller image size          | Only copies what’s needed into final image.           |
| Cleaner and more secure     | No compilers, build tools, or secrets in final image. |
| Easier caching              | Speeds up builds using intermediate layers.           |
| Better separation of stages | Logical separation of build and runtime environments. |



####  Example

```Dockerfile
# Stage 1: Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Final minimal image
FROM alpine:latest
COPY --from=builder /app/myapp /usr/local/bin/myapp
ENTRYPOINT ["myapp"]
```


####  Key Points

| Keyword           | Description                                                             |
| ----------------- | ----------------------------------------------------------------------- |
| `AS builder`      | Assigns a name to a stage. You can refer to this name in `COPY --from`. |
| `COPY --from=...` | Copies files from a previous build stage.                               |

---

#### 📁 Example: Python App Multistage Build

```Dockerfile
# Stage 1: Build dependencies
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: Final image
FROM python:3.11-alpine
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```



#### Notes:

* Multistage builds are useful to separate build logic from final image runtime.
* Use `COPY --from=...` carefully to include **only necessary files** in the final image.
* Helps comply with **Red Hat best practices** for minimal and secure container images.


