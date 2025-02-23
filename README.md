<h2>前言</h2>
<p>本文的目的是记录在我Ubuntu 24 LST上安装RAGFLOW的过程，主要解决RAGFLOW从docker拉取的各种问题：</p>
<ol>
   <li>Ubunut 24.LST docker的升级</li>
   <li>infiniflow/ragflow的安装</li>
   <li>确保用户加入docker群（group）并生效</li>
</ol>
<h2 id="docker">Ubunut 24.LST docker的升级</h2>
在Ubuntu 24 LST上安装Docker的步骤如下：

**步骤1：更新系统包**
```bash
sudo apt update && sudo apt upgrade -y
```

**步骤2：安装依赖项**
```bash
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release -y
```

**步骤3：添加Docker官方GPG密钥**
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

**步骤4：配置Docker稳定版仓库（推荐国内镜像加速）**
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**步骤5：安装Docker Engine**
```bash
sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y
```

**步骤6：验证安装**
```bash
sudo docker run hello-world
```
若输出类似`Hello from Docker!`即表示安装成功。

**优化配置：**
1. **配置Docker镜像加速（可选）**
```bash
sudo mkdir -p /etc/docker
echo "{
  \"registry-mirrors\": [\"https://docker.mirrors.ustc.edu.cn\"]
}" | sudo tee /etc/docker/daemon.json
sudo systemctl restart docker
```

2. **赋予非root用户权限**
```bash
sudo usermod -aG docker $USER
newgrp docker  # 需重新登录生效
```

3. **安装Docker Compose**
- 通过以下命令安装Docker Compose：
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.33.1/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```


<h2 id="ragflow">RAGFLOW的安装</h2>

**步骤1. 下载ragflow**
```bash
git clone https://github.com/infiniflow/ragflow
```

**步骤2. 设置镜像**

<p>修改ragflow/docker目录里.evn, docker-compose-base.yml, docker-compose.yml相关镜像设置，直接从我这个仓库下载题换即可。</p>

```bash
docker
├── docker-compose-base.yml
├── docker-compose-base.yml.bak
├── docker-compose.yml
├── docker-compose.yml.bak
├── .env
└── .env.bak
```

**步骤3. 拉取docker镜像**

```bash
docker compose -f docker/docker-compose.yml up -d
```

<p>如果出现权限问题，比如：</p>
  
```bash
WARN[0000] The "MACOS" variable is not set. Defaulting to a blank string. 
unable to get image 'swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/mysql:8.0.39': permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.47/images/swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/mysql:8.0.39/json": dial unix /var/run/docker.sock: connect: permission denied
exit status 1
```

<p>则请确保用户已经加入docker群组，方法见上一节的“赋予非root用户权限“。</p>

