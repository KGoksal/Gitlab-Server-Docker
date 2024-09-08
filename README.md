# Gitlab-Server-Docker
Running Gitlab Server via Docker

```
docker run -d --hostname gitlab.dev-ops.expert \
-p 443:443 -p 80:80 -p 2222:22 \
--name gitlab \
--restart unless-stopped \
-v /root/gitlab/config:/etc/gitlab \
-v /root/gitlab/logs:/var/log/gitlab \
-v /root/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
