# GitLab Server Deployment via Docker

This guide will walk you through deploying a GitLab server using Docker. GitLab is a web-based DevOps lifecycle tool that provides a Git repository manager, CI/CD pipelines, and more.

## Prerequisites 

Before starting, make sure you have the following installed on your system:

- **Docker**: [Installation Guide](https://docs.docker.com/get-docker/)
- **Docker Compose** (optional, but recommended for easier multi-service deployment): [Installation Guide](https://docs.docker.com/compose/install/)

## Table of Contents

- [Clone the GitLab Docker Image](#clone-the-gitlab-docker-image)
- [Run GitLab Using Docker](#run-gitlab-using-docker)
- [Run GitLab Using Docker Compose (Optional)](#run-gitlab-using-docker-compose-optional)
- [Configuration Options](#configuration-options)
- [Accessing GitLab](#accessing-gitlab)
- [Persisting Data](#persisting-data)
- [Backup and Restore](#backup-and-restore)
- [Contributing](#contributing)
- [License](#license)

---

## Clone the GitLab Docker Image

GitLab provides a ready-to-use Docker image that can be deployed easily. First, pull the GitLab Docker image from Docker Hub:

```bash
docker pull gitlab/gitlab-ee:latest
```

For the open-source community edition (CE), use:

```bash
docker pull gitlab/gitlab-ce:latest
```

## Run GitLab Using Docker

You can run the GitLab container with the following command. This will start a GitLab instance using default configurations.

```bash
docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ee:latest
```

### Explanation:
- `--detach`: Run the container in the background.
- `--hostname`: Set your desired hostname for GitLab.
- `--publish`: Map the container ports to your local machine. Ports 443 (HTTPS), 80 (HTTP), and 22 (SSH) are exposed.
- `--name`: Name the Docker container.
- `--restart always`: Ensure the container restarts automatically on failure.
- `--volume`: Persist configuration, logs, and data across restarts.
- `gitlab/gitlab-ee:latest`: The official GitLab Docker image (Enterprise Edition).

### Notes:
- If you don't need the Enterprise Edition, replace `gitlab/gitlab-ee:latest` with `gitlab/gitlab-ce:latest` for the Community Edition.

## Run GitLab Using Docker Compose (Optional)

You can also deploy GitLab using Docker Compose, which simplifies managing multiple services. Create a `docker-compose.yml` file with the following content:

```yaml
version: '3'
services:
  gitlab:
    image: gitlab/gitlab-ee:latest
    container_name: gitlab
    restart: always
    hostname: 'gitlab.example.com'
    ports:
      - '443:443'
      - '80:80'
      - '22:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'
```

### Steps to run:

1. **Start GitLab with Docker Compose**:
   ```bash
   docker-compose up -d
   ```

2. **Stop GitLab with Docker Compose**:
   ```bash
   docker-compose down
   ```

This will start GitLab with the same default settings as the Docker run command.

## Configuration Options

### Customize External URL

You can change the external URL of GitLab by setting the `GITLAB_OMNIBUS_CONFIG` environment variable or directly in `/etc/gitlab/gitlab.rb`.

To specify the external URL, you can add:

```bash
docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.example.com/'" \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ee:latest
```

Or with Docker Compose:

```yaml
environment:
  GITLAB_OMNIBUS_CONFIG: |
    external_url 'http://gitlab.example.com'
```

### Additional Configuration

To further customize GitLab settings, modify the `/etc/gitlab/gitlab.rb` file inside the container. After making changes, run the following command to reconfigure the container:

```bash
docker exec -it gitlab gitlab-ctl reconfigure
```

## Accessing GitLab

Once the GitLab container is running, you can access the web interface via:

```
http://<your-server-ip>
```

By default, the root user password is set on the first login. You will be prompted to set it via the web interface.

- **Username**: `root`
- **Password**: Set on first login

If you lose access to the root account, you can reset the password by running:

```bash
docker exec -it gitlab gitlab-rails runner "User.where(id: 1).first.update_attributes(password: 'newpassword')"
```

## Persisting Data

GitLab's data, logs, and configuration are stored in Docker volumes. This ensures data is retained even after container restarts.

To persist data, we have mounted the following directories:
- `/etc/gitlab`: GitLab configuration files.
- `/var/log/gitlab`: Logs generated by GitLab.
- `/var/opt/gitlab`: Application data.

These volumes are mapped to the host directories `/srv/gitlab/config`, `/srv/gitlab/logs`, and `/srv/gitlab/data` respectively.

## Backup and Restore

### Backup

To create a backup of your GitLab instance, run the following command:

```bash
docker exec -t gitlab gitlab-backup create
```

Backups are stored in `/var/opt/gitlab/backups` by default. You can map this to your local machine or any persistent storage.

### Restore

To restore a backup, first, stop the running GitLab instance:

```bash
docker-compose down
```

Then, place your backup in `/var/opt/gitlab/backups` and run the following:

```bash
docker exec -it gitlab gitlab-backup restore BACKUP=<backup filename>
```

Finally, restart the GitLab instance:

```bash
docker-compose up -d
```

## Contributing

Contributions are welcome! Feel free to open a pull request or submit issues.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

---
