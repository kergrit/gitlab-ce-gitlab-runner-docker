# Setup Gitlab CE and Gitlab Runner with docker

## Gitalb Server CE
We have 2 solutions to install gitlab-ce
- AWS Marketplace Gitlab CE https://aws.amazon.com/marketplace/pp/prodview-w6ykryurkesjq
- Docker Container
```yaml
  web:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: '54.169.192.203'
    container_name: gitlab-ce
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://54.169.192.203'
    ports:
      - 80:80
      - 443:443
    volumes:
      - '/mnt/efs/fs1/gitlab/config:/etc/gitlab'
      - '/mnt/efs/fs1/gitlab/logs:/var/log/gitlab'
      - '/mnt/efs/fs1/gitlab/data:/var/opt/gitlab'
    networks:
      - gitlab
networks:
  gitlab:
    name: gitlab-network
```

After we install or deploy gitlab ce we need to configuration `external_url`.

```sh
# [Optional for docker container]
docker exec -it --user=root gitlab-ce sh

# Update gitlab ce config for external_url
sudo vim /etc/gitlab/gitlab.rb

# Reload gitlab ce configuration
sudo gitlab-ctl reconfigure

# Get default root password
sudo grep 'Password:' /etc/gitlab/initial_root_password

# Reset root password
sudo gitlab-rake 'gitlab:password:reset[root]'

```


## Gitlab Runner
We have 2 solutions to install gitlab runner 
- Native install on Linux 
```sh
# Based on AMI 2023
# Add the official GitLab repository
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash

# Install the latest version of GitLab Runner
sudo yum install gitlab-runner
```

- Docker container
  
```yaml
gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    container_name: gitlab-runner
    restart: always
    depends_on:
      - web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - '/mnt/efs/fs1/gitlab/gitlab-runner:/etc/gitlab-runner'
    networks:
      - gitlab
networks:
  gitlab:
    name: gitlab-network

```

Next step gitlab runner register


```sh
# Login gitlab ce portal with root /admin
# Goto CI/CD > Runners > New instance runner
# Choose Linux and create

# [Optional for docker container]
docker exec -it --user=root gitlab-runner sh

sudo gitlab-runner register  --url http://54.169.192.203  --token glrt-Xzynx3ozBzMUQdvxhebv

#OR

sudo gitlab-runner register
  --non-interactive \
  --url "http://54.169.192.203"
  --token "glrt-Xzynx3ozBzMUQdvxhebv"
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --maintenance-note "Free-form maintainer notes about this runner" \
  --tag-list "docker,aws" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected" \
  --docker-privileged
```

# References
- [AWS Market Place Gitlab Community Edition](https://aws.amazon.com/marketplace/server/fulfillment?deliveryMethod=4fd521c2-9d5d-4e1a-a34b-66ef9ca0c8e4&productId=4fd521c2-9d5d-4e1a-a34b-66ef9ca0c8e4&region=ap-southeast-1&version=86e24600-544e-4792-8eb5-7b56ca8a96e8&ami=ami-0ac77c9ad3d21e0c3&ref_=cfg_full_continue)
- [AWS implementation patterns](https://docs.gitlab.com/ee/install/aws/)
- [Install GitLab Runner using the official GitLab repositories](https://docs.gitlab.com/runner/install/linux-repository.html)
- [Migrating to the new runner registration workflow](https://docs.gitlab.com/ee/ci/runners/new_creation_workflow)
- [CI/CD YAML syntax reference](https://docs.gitlab.com/ee/ci/yaml/)