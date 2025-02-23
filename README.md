<h2>前言</h2>
<p>本文的目的是记录在我Ubuntu 24 LST上安装RAGFLOW的过程，主要解决RAGFLOW从docker拉取的各种问题：</p>
<ol>
   <li>Ubunut 24.LST docker的升级</li>
   <li>infiniflow/ragflow的安装</li>
   <li>确保用户加入docker群（group）并生效</li>
</ol>
<h2 id="docker">Ubunut 24.LST docker的升级</h2>

<p>第一次在Ubuntu 24 LST/Docker 24.0安装RAGFLOW失败，发现需要更新的版本，因此进行升级，步骤如下：</p>

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

<p>则请确保用户已经加入docker群组，方法见上一节的“赋予非root用户权限“。或者用sudo启动docker</p>

**步骤4. 检查docker运行**

```bash
docker logs -f ragflow-server
```

<p>你会看到类似下面的内容：</p>

```bash
2025-02-23 15:22:41,207 INFO     16 ragflow_server log path: /ragflow/logs/ragflow_server.log, log levels: {'peewee': 'WARNING', 'pdfminer': 'WARNING', 'root': 'INFO'}
2025-02-23 15:22:46,826 INFO     16 init database on cluster mode successfully
2025-02-23 15:22:57,053 INFO     16 
        ____   ___    ______ ______ __               
       / __ \ /   |  / ____// ____// /____  _      __
      / /_/ // /| | / / __ / /_   / // __ \| | /| / /
     / _, _// ___ |/ /_/ // __/  / // /_/ /| |/ |/ / 
    /_/ |_|/_/  |_|\____//_/    /_/ \____/ |__/|__/                             

    
2025-02-23 15:22:57,053 INFO     16 RAGFlow version: v0.16.0-63-g7b5d8312 full
2025-02-23 15:22:57,053 INFO     16 project base: /ragflow
2025-02-23 15:22:57,053 INFO     16 Current configs, from /ragflow/conf/service_conf.yaml:
	ragflow: {'host': '0.0.0.0', 'http_port': 9380}
	mysql: {'name': 'rag_flow', 'user': 'root', 'password': '********', 'host': 'mysql', 'port': 3306, 'max_connections': 100, 'stale_timeout': 30}
	minio: {'user': 'rag_flow', 'password': '********', 'host': 'minio:9000'}
	es: {'hosts': 'http://es01:9200', 'username': 'elastic', 'password': '********'}
	infinity: {'uri': 'infinity:23817', 'db_name': 'default_db'}
	redis: {'db': 1, 'password': '********', 'host': 'redis:6379'}
2025-02-23 15:22:57,054 INFO     16 Use Elasticsearch http://es01:9200 as the doc engine.
2025-02-23 15:22:57,209 INFO     16 GET http://es01:9200/ [status:200 duration:0.154s]
2025-02-23 15:22:57,213 INFO     16 HEAD http://es01:9200/ [status:200 duration:0.004s]
2025-02-23 15:22:57,214 INFO     16 Elasticsearch http://es01:9200 is healthy.
2025-02-23 15:22:57,217 WARNING  16 Load term.freq FAIL!
2025-02-23 15:22:57,220 WARNING  16 Realtime synonym is disabled, since no redis connection.
2025-02-23 15:22:57,224 WARNING  16 Load term.freq FAIL!
2025-02-23 15:22:57,226 WARNING  16 Realtime synonym is disabled, since no redis connection.
2025-02-23 15:22:57,227 INFO     16 MAX_CONTENT_LENGTH: 134217728
2025-02-23 15:22:57,227 INFO     16 SERVER_QUEUE_MAX_LEN: 1024
2025-02-23 15:22:57,227 INFO     16 SERVER_QUEUE_RETENTION: 3600
2025-02-23 15:22:57,227 INFO     16 MAX_FILE_COUNT_PER_USER: 0
2025-02-23 15:23:00,289 INFO     16 init web data success:1.0174591541290283
2025-02-23 15:23:00,292 INFO     16 RAGFlow HTTP server start...
2025-02-23 15:23:00,294 INFO     16 WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:9380
 * Running on http://172.18.0.5:9380
2025-02-23 15:23:00,294 INFO     16 Press CTRL+C to quit
2025-02-23 15:23:02,849 INFO     18 task_executor_0 log path: /ragflow/logs/task_executor_0.log, log levels: {'peewee': 'WARNING', 'pdfminer': 'WARNING', 'root': 'INFO'}
2025-02-23 15:23:03,145 INFO     18 init database on cluster mode successfully
2025-02-23 15:23:09,998 INFO     18 TextRecognizer det uses CPU
2025-02-23 15:23:10,119 INFO     18 TextRecognizer rec uses CPU
2025-02-23 15:23:10,144 INFO     18 

```

**步骤5. 配置模型**

<p>需要注意的是，docker配置模型的URL：</p>

```bash
http://host.docker.internal:11434
```

<h2 id="image">Docker镜像搜索</h2>
<p>https://docker.aityp.com/</p>
<p>比如：</p>
<p>https://docker.aityp.com/image/docker.io/infiniflow/ragflow:latest</p>
