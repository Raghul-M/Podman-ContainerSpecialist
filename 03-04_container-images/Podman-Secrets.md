

## 🔐 Podman Secrets 

#### What are Podman Secrets?

**Secrets** in Podman are used to **securely store sensitive information** such as passwords, API tokens, SSH keys, TLS certificates, etc.
They are **mounted as read-only files** into the container at runtime and **not baked into the image**, making them more secure.


#### 📌 Why Use Secrets?

* Prevent hardcoding credentials in images or environment variables.
* Manage sensitive data securely using **Podman’s native secrets management** (introduced in Podman 4.4+).
* Secrets can be reused across multiple containers.
* Secrets are stored and isolated on the **host** and only **mounted when needed**.



####  How to Use Secrets in Podman

#### 1. **Create a Secret**

You can create a secret using a string or a file.

```bash
# From a literal string
podman secret create my-password - <<< 'supersecret123'

# From a file
podman secret create my-ssh-key ./id_rsa
```

#### 2. **List Secrets**

```bash
podman secret ls
```

#### 3. **Inspect a Secret**

```bash
podman secret inspect my-password
```

#### 4. **Use Secrets in a Container**

Mount the secret into the container using `--secret`.

```bash
podman run --rm \
  --secret my-password \
  alpine cat /run/secrets/my-password
```

* Inside the container, the secret will be mounted at `/run/secrets/<secret-name>`
* It’s **read-only** and **only available during the container’s lifetime**

#### 5. **Delete a Secret**

```bash
podman secret rm my-password
```

---

####  Extra Options (Podman 5+)

```bash
# Change the target file name inside the container
podman run --rm \
  --secret source=my-password,target=passwd.txt \
  alpine cat /run/secrets/passwd.txt
```

The podman secret create command supports different drivers to store the secrets. You can use the --driver or -d option to specify one of the supported secret drivers:

> **file (default)** Stores the secret in a read-protected file

> **pass** Stores the secret in a GPG-encrypted file.

> **shell** Manages the secret storage by using a custom script.

**eg :**
```
podman secret create -d pass my-secret-pass - <<< 'sensitive-data'
```
---

### 🧠 Key Points 

| Concept       | Description                                                                           |
| ------------- | ------------------------------------------------------------------------------------- |
| Secrets Scope | Stored locally on the host. Not shared across systems unless manually synced.         |
| Security      | Not exposed in image, logs, or `env`. Mounted as secure files inside `/run/secrets/`. |
| Cleanup       | Secrets are not auto-deleted. Use `podman secret rm` to remove them.                  |
| Usage         | Preferred over `ENV` or `--env` for sensitive values.                                 |
| Limitations   | Available in Podman 4.4+ (ensure you’re using a supported version during exam).       |

---


