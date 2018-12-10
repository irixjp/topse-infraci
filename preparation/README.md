
## 環境構築

以下の操作を全て `root` で実施します。

Prepare the latest docker & docker-compose for CentOS7
```
yum update -y
yum install -y yum-utils device-mapper-persistent-data lvm2 git
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce

vi /usr/lib/systemd/system/docker.service
---
ExecStart=/usr/bin/dockerd -H unix://
  ↓
ExecStart=/usr/bin/dockerd --insecure-registry 192.168.33.10:4567 -H unix://
---

systemctl enable docker


# Check the latest version by https://github.com/docker/compose/releases/
VERSION=1.23.2
curl -L https://github.com/docker/compose/releases/download/${VERSION}/docker-compose-`uname -s`-`uname -m` -o /bin/docker-compose
chmod +x /bin/docker-compose


reboot
```


CI環境の起動（3-5分）
```
git clone https://github.com/irixjp/topse-infraci.git
cd topse-infraci/preparation/docker/compose/setup/docker-compose
docker-compose up -d
```


GitLabへログイン可能になったら以下を実行
```
RUNNER_HOST=gitlab-runner-1
RUNNER_TOKEN=token-AABBCCDD
RUNNER_TAG=docker
BASE_IMAGE=centos:latest

docker exec ${RUNNER_HOST:?} \
       gitlab-runner register \
       --non-interactive \
       --url http://192.168.33.10 \
       --registration-token ${RUNNER_TOKEN:?} \
       --tag-list ${RUNNER_TAG:?} \
       --executor docker \
       --locked=false \
       --docker-image ${BASE_IMAGE:?} \
       --clone-url http://192.168.33.10/ \
       --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
       --docker-privileged=true \
       --docker-network-mode docker-compose_infraci_nw
```
