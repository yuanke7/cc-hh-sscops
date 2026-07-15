# 100 Server 部署

该目录用于在 `10.241.133.100` 的 `/home/yuankq1/cc-hh-sscops` 部署无界面的 SSC Ops Agent：

- `server` 提供本地 API/WebSocket 服务并调用 Qwen 模型。
- `feishu` 通过飞书长连接接收消息，不需要开放回调端口。
- 两个容器共享 `state/` 中的会话、授权和配对状态。
- API 默认只监听服务器的 `127.0.0.1:3456`。
- 项目代码以只读方式挂载到容器的 `/workspace`。

## 私密配置

部署时在本目录创建两个不入库的文件：

`model.env`：

```dotenv
ANTHROPIC_BASE_URL=http://led-gateway.lenovo.com:30089/intranet/qwen35-122b-a10b/v1
ANTHROPIC_API_KEY=<从安全存储读取>
ANTHROPIC_MODEL=Qwen3.6-27B
ANTHROPIC_DEFAULT_HAIKU_MODEL=Qwen3.6-27B
ANTHROPIC_DEFAULT_SONNET_MODEL=Qwen3.6-27B
ANTHROPIC_DEFAULT_OPUS_MODEL=Qwen3.6-27B
```

`feishu.env`：

```dotenv
FEISHU_APP_ID=<复用原 SSC Ops Agent 配置>
FEISHU_APP_SECRET=<从安全存储读取>
```

两个文件只保存在服务器本地，并限制为 root 可读：

```bash
chmod 600 model.env feishu.env
```

## 启动与检查

```bash
cd /home/yuankq1/cc-hh-sscops/deploy/100server
mkdir -p state
docker compose up -d --build
docker compose ps
docker compose logs --tail=100 server feishu
curl --fail http://127.0.0.1:3456/health
```

若服务器无法访问 Docker Hub，可上传本地构建产物到 `offline/`，复用服务器已有基础镜像：

```bash
docker build -f Dockerfile.offline -t cc-hh-sscops:local ../..
docker compose up -d --no-build
```

在本机建立 SSH 隧道后访问 Web：

```bash
ssh -L 3456:127.0.0.1:3456 root@10.241.133.100
```

浏览器打开 `http://127.0.0.1:3456`。

更新密钥后无需重新构建镜像，按配置类型重建对应容器：

```bash
docker compose up -d --force-recreate server
docker compose up -d --force-recreate feishu
```

首次使用时需要生成一次飞书配对码，并在与机器人的私聊中发送该配对码。配对成功后，授权状态会保存在 `state/adapters.json`。
