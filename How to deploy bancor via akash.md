# Bancor Web App

**编译**

下载 [Bancor Web App](https://github.com/bancorprotocol/webapp)

```
git clone https://github.com/bancorprotocol/webapp.git
```

然后进到 `webapp` 根目录依次执行命令

```
yarn install
yarn start
```

如果这里正确浏览器访问 `http://localhost:8081/` 就可以访问 `Bancor` 

然后执行命令打包应用

```
yarn build
```

成功打包之后的文件会放在 `./dist` 目录下

**创建 `Docker` 镜像**

创建 ``Dockerfile`` 文件在里面添加一下内容

```
FROM nginx
COPY dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

之后打包镜像

```
docker build . -t bancor:latest
```

查看刚刚创建的镜像文件，确认刚刚打包的 `bancor` 镜像是否存在

```
docker images
```

创建并运行后台容器， 指定端口号 3257

```
docker run -d --name bancor -p 3257:80 --restart=unless-stopped bancor:latest
```

运行之后通过 `http://localhost:3257` 访问 `Bancor` 



确认无误之后我们给刚刚的镜像打赏 `TAG`

```
docker tag bancor:latest sueinz/bancor:latest
```

之后我们上传前面生成的镜像到 `Docker Hub`

```
docker push sueinz/bancor:latest
```



**通过 `AKASH` 部署 `Bancor`**

这里参考部署教程，由于我们这个应用是个标准的 `web` 应用，我们直接参考[官方教程部署](https://docs.akash.network/v/master/guides/deploy)

修改官方文件[ deploy.yml](https://github.com/ovrclk/docs/blob/488f808b804b646771baf28e64dcfae2c5b09cac/guides/deploy/deploy.yml)文件，修改

```
services:
  web:
    image: lanakul/balancer:latest
    expose:
      - port: 80
        as: 80
        to:
          - global: true
```

为

```
services:
  web:
    image: sueinz/bancor:latest
    expose:
      - port: 80
        as: 80
        to:
          - global: true
```

然后执行命令部署

```
akash tx deployment create deploy.yml --from $KEY_NAME --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID -y
```

得到如下输出

```
{
	"height": "149760",
	"txhash": "AC22B0BA0B466F759CEC6088205329811EF7C0494B916000C85376F81784AFB8",
	"codespace": "",
	"code": 0,
	"data": "0A130A116372656174652D6465706C6F796D656E74",
	"raw_log": "[{\"events\":[{\"type\":\"akash.v1\",\"attributes\":[{\"key\":\"module\",\"value\":\"deployment\"},{\"key\":\"action\",\"value\":\"deployment-created\"},{\"key\":\"version\",\"value\":\"1e519ae40d324465b34c2239f8da96200b50bcb01f97722833da2c83f934fe2a\"},{\"key\":\"owner\",\"value\":\"akash19uwxt0ln3p9fenpdwtsm5rlmqsekc3g9ym6xqz\"},{\"key\":\"dseq\",\"value\":\"149758\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"create-deployment\"}]}]}]",
	"logs": [{
		"msg_index": 0,
		"log": "",
		"events": [{
			"type": "akash.v1",
			"attributes": [{
				"key": "module",
				"value": "deployment"
			}, {
				"key": "action",
				"value": "deployment-created"
			}, {
				"key": "version",
				"value": "1e519ae40d324465b34c2239f8da96200b50bcb01f97722833da2c83f934fe2a"
			}, {
				"key": "owner",
				"value": "akash19uwxt0ln3p9fenpdwtsm5rlmqsekc3g9ym6xqz"
			}, {
				"key": "dseq",
				"value": "149758"
			}]
		}, {
			"type": "message",
			"attributes": [{
				"key": "action",
				"value": "create-deployment"
			}]
		}]
	}],
	"info": "",
	"gas_wanted": "200000",
	"gas_used": "56891",
	"tx": null,
	"timestamp": ""
}
```

之后运行命令查看刚刚部署的状态

```
akash query market lease list --owner $ACCOUNT_ADDRESS --node $AKASH_NODE --state active
```

得到输出

```
leases:
- lease_id:
    dseq: "149758"
    gseq: 1
    oseq: 1
    owner: akash19uwxt0ln3p9fenpdwtsm5rlmqsekc3g9ym6xqz
    provider: akash1ds82uk3pzawavlasc2qd88luewzye2qrcwky7t
  price:
    amount: "235"
    denom: uakt
  state: active
pagination:
  next_key: null
  total: "0"
```

这里记录下前面的输出的几个变量备用

```
export PROVIDER=akash1ds82uk3pzawavlasc2qd88luewzye2qrcwky7t
export DSEQ=149758
export OSEQ=1
export GSEQ=1
```

然后上传 `Manifest`

```
akash provider send-manifest deploy.yml --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --owner $ACCOUNT_ADDRESS --provider $PROVIDER
```

最后查看我们刚刚部署的具体信息

```
akash provider lease-status --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --provider $PROVIDER --owner $ACCOUNT_ADDRESS
```

输出

```
{
  "services": {
    "web": {
      "name": "web",
      "available": 1,
      "total": 1,
      "uris": [
        "4ge79u689dd3d2qie2670bbjhg.provider5.akashdev.net"
      ],
      "observed-generation": 0,
      "replicas": 0,
      "updated-replicas": 0,
      "ready-replicas": 0,
      "available-replicas": 0
    }
  },
  "forwarded-ports": {}
}
```

然后访问 `4ge79u689dd3d2qie2670bbjhg.provider5.akashdev.net` 查看我们刚刚部署的应用程序，不出意外的话部署成功。

