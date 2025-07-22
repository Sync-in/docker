<a href="https://sync-in.com" target="_blank" rel="noopener">
<picture>
  <source srcset="https://raw.githubusercontent.com/Sync-in/assets/main/logo-dark.svg" media="(prefers-color-scheme: dark)" />
  <img src="https://raw.githubusercontent.com/Sync-in/assets/main/logo.svg" alt="Sync-in" width="250" height="auto" />
</picture>
</a>

_Welcome to the official Sync-in Docker repository!_

- 🌍 [Website](https://sync-in.com)
- 📄 [Documentation](https://sync-in.com/docs)
- 🧪 [Try the demo](https://sync-in.com/docs/demo)

<a href="#-license"><img src="https://img.shields.io/badge/Licence-AGPL%20v3.0-green.svg" alt="License"/></a>
<a href="https://www.paypal.com/donate/?business=HU3F3CMDDH7YJ&no_recurring=0&item_name=I+rely+on+your+donations+to+grow+the+Sync-in+open+source+project.+Thank+you+for+your+support+%E2%80%94+it+truly+makes+a+difference%21&currency_code=EUR" target="_blank"><img src="https://img.shields.io/badge/Donate-PayPal-blue.svg" alt="Paypal"/></a>
<a href="https://liberapay.com/sync-in/donate" target="_blank"><img src="https://img.shields.io/badge/Donate-LiberaPay-yellow.svg" alt="LiberaPay"/></a>
<a href="https://discord.gg/qhJyzwaymT" target="_blank"><img src="https://img.shields.io/badge/Discord-Online-brightgreen.svg" alt="Discord"/></a>


**Sync-in** can be installed using the official Sync-in Docker image.  
This [official image](https://hub.docker.com/r/syncin/server) is designed to be used in **docker compose setup**.

---

## Table of Contents

- [Requirements](#-requirements)
    - [Docker](#-docker)
    - [Setup Files](#-setup-files)
- [QuickStart](#-quickstart)
    - [1. Default secrets](#1-default-secrets)
    - [2. Launch the Sync-in server](#2-launch-the-sync-in-server)
    - [3. Access the web interface](#3-access-the-web-interface)
- [Configuration](#-configuration)
    - [Nginx](#nginx)
    - [OnlyOffice (optional)](#onlyoffice-optional)
    - [Client repository (optional)](#clients-repository-optional)
- [Support](#-support)
- [Contributing](#-contributing)
- [License](#-license)

---

## 📋 Requirements

### 🐳 Docker

This setup relies on Docker Compose files using the `x-include` extension.  
You need:

- **Docker Engine** version **24 or higher**
- **Docker Compose** version **2.20 or higher**

✅ You can check your installed versions with:

```bash
docker --version
docker compose version
```

🔗 If Docker is not installed, follow
the [official Docker installation guide](https://docs.docker.com/get-started/get-docker/).

### 📦 Setup Files

To get the files from this repository, use either:

- `curl` and `unzip`:
  ```bash
  curl -L -o sync-in-docker.zip \
  https://github.com/Sync-in/docker/archive/refs/heads/main.zip && \
  unzip sync-in-docker.zip
  ```
- or `git`:
  ```bash
  git clone https://github.com/Sync-in/docker.git sync-in-docker
  ```

You should have the following file structure in the `sync-in-docker` directory:

```
├── config
│   ├── nginx
│   │   ├── docker-compose.nginx.yaml
│   │   └── nginx.conf
│   ├── onlyoffice
│   │   └── docker-compose.onlyoffice.yaml
│   └── sync-in-desktop-releases
│       └── docker-compose.sync-in-desktop-releases.yaml
│       └── update.sh
├── docker-compose.yaml
└── environment.yaml
```

---

## 🚀 QuickStart

### 1. Default secrets

Move into the Docker Compose directory:

```bash
cd sync-in-docker
```

Before starting the server, make sure to replace the default secrets for security purposes.

Edit the `environment.yaml` file and update the values as follows:

```yaml
auth:
  token:
    access:
      secret: "changeAccessWithStrongSecret"
    refresh:
      secret: "changeRefreshWithStrongSecret"
mysql:
  # Use the MySQL root password as defined in docker-compose.yaml
  url: mysql://root:MySQLRootPassword@mariadb:3306/sync_in
```

Then, edit the `docker-compose.yaml` file and make sure the MySQL root password matches:

```yaml
services:
  mariadb:
    environment:
      MYSQL_ROOT_PASSWORD: "MySQLRootPassword"
```

> 🔐 Tip: Use long, randomly generated strings for secrets to ensure proper security.

> ⚠️ Warning: Make sure to wrap your passwords in quotes if they contain special characters.

### 2. Launch the Sync-in server

To define your own administrator credentials on first launch, use:

```bash
INIT_ADMIN_LOGIN='user' INIT_ADMIN_PASSWORD='password' docker compose up -d
```

Replace `user` and `password` with your preferred login and password for the initial admin account.

If you omit these variables and run:

```bash
docker compose up -d
```

The server will use the **default administrator credentials**:

- **Login:** `sync-in`
- **Password:** `sync-in`

> ⚠️ **Important:** For security reasons, it’s strongly recommended to **change both the default login and password**
> immediately after your first login.

> 💡 To check the container status, use: `docker compose ps`

### 3. Access the web interface

Once the server is running, open your browser and navigate to:

```
http://localhost:8080
```

Log in using the administrator credentials you configured in [step 2](#2-launch-the-sync-in-server).

---

## 🛠️ Configuration

### Nginx

In a production setup, a **reverse proxy** helps ensure secure and efficient access to your Sync-in instance.

The official **Nginx Docker image** is used, and a **Sync-in-specific Nginx configuration** is included in this
repository.

To enable it, edit the `docker-compose.yaml` file and **uncomment** the following lines at the top of the file:

```yaml
include:
  - ./config/nginx/docker-compose.nginx.yaml
```

To configure **HTTPS**, edit the file `./config/nginx/nginx.conf` and add your SSL certificate settings.

> 💡`docker compose up -d` to apply

Nginx is now enabled on **port 80**, making Sync-in accessible via `http://localhost`.

### OnlyOffice (optional)

⚠️ To enable **OnlyOffice integration** with Sync-in, make sure the **[Nginx configuration](#nginx) is set up** first.

To enable the container, edit the `docker-compose.yaml` file and **uncomment** the following lines at the top of the
file:

```yaml
include:
  - ./config/nginx/docker-compose.nginx.yaml
  - ./config/onlyoffice/docker-compose.onlyoffice.yaml
```

Update the `environment.yaml` configuration file to activate the integration:

```yaml
applications:
  files:
    onlyoffice:
      # enable integration
      enabled: true
      # use the same secret as in docker-compose.yaml
      secret: onlyOfficeSecret
```

Replace the default secret above with a strong value, and ensure it matches the one in
`./config/onlyoffice/docker-compose.onlyoffice.yaml`:

```yaml
services:
  onlyoffice:
    environment:
      - JWT_SECRET=onlyOfficeSecret
```

> 💡`docker compose up -d && docker compose restart sync_in` to apply

Sync-in and OnlyOffice run in Docker containers and communicate through **Nginx**.  
It is essential to access the interface using either the **server’s IP address** or a
**properly configured domain name**.

⚠️ Accessing Sync-in via `http://localhost` or `http://127.0.0.1` **will not allow OnlyOffice to function properly** —
documents won't be viewable or editable.

### Clients repository (optional)

If desktop applications do not have internet access, the Sync-in server can act as a **local repository** for desktop
client releases.  
This allows users to **download the applications directly from their account** and benefit from **automatic updates**.

To enable it, edit the `docker-compose.yaml` file and **uncomment** the following lines at the top of the file:

```yaml
include:
  - ./config/sync-in-desktop-releases/docker-compose.sync-in-desktop-releases.yaml
```

Then, in the `environment.yaml` file, add the `appStore` section under `applications/files` path:

```yaml
applications:
  files:
    appStore:
      repository: local
```

To enable this feature or update the releases, you need to run the following commands:

```bash
chmod +x ./config/sync-in-desktop-releases/update.sh
./config/sync-in-desktop-releases/update.sh
```
---

## 💛 Support

Sync-in is an independent open source project.  
If you find it useful, you can:

- ⭐ Star the repositories
- 🐛 Report issues and suggest improvements
- 🤝 Contribute code, translations, or documentation
- 💬 Join the community on :
  - [Discord](https://discord.gg/qhJyzwaymT)
  - [Stack Overflow](https://stackoverflow.com/questions/tagged/sync-in)
- 💖 Support the project via :
  - [Paypal](https://www.paypal.com/donate/?business=HU3F3CMDDH7YJ&no_recurring=0&item_name=I+rely+on+your+donations+to+grow+the+Sync-in+open+source+project.+Thank+you+for+your+support+%E2%80%94+it+truly+makes+a+difference%21&currency_code=EUR)
  - [Liberapay](https://liberapay.com/sync-in)
  

---

## 🤝 Contributing
Before submitting your pull request, please confirm the following:

- ✅ I have read and followed the [contribution guide](readme/CONTRIBUTING.md).
- ✅ I am submitting this pull request in good faith and to help improve Sync-in.

---

## 📜 License

This project is licensed under the **GNU Affero General Public License (AGPL-3.0-or-later)**.  
See [LICENSE](LICENSE) for the full text.

Sync-in® is a registered trademark, see our [Trademark Policy](https://sync-in.com/docs/about/trademark).

---

_Thank you for using **Sync-in** ! 🚀_