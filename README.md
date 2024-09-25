GoEdge API节点源码

## 源码编译

## 整体源码结构
从Github上对应仓库下载各个组件的源码后，建议的整体源码结构为：

EdgeProject/
   EdgeAdmin     # 管理平台
   EdgeAPI       # API节点
   EdgeNode      # 边缘节点
   EdgeCommon    # 公共依赖
   ....


源码下载地址：

* GitHub: https://github.com/orglen ，一直保持最新

## 运行环境

* 操作系统：目前只支持macOS和Linux开发环境；
* MySQL：支持 v5.7.x 以上 / TiDB 3.0以上；
* Golang：支持 v1.21及以上；
* macOS X上需要安装musl交叉编译工具链，安装方法当前文档的下一段会给出。
* 安装前需要准备wget、patch、bzip2、gcc-c++ gcc等。



### 编译EdgeAPI API节点

API节点是唯一可以操作数据库的节点，所以需要在步骤中配置数据库，也是其他节点依赖运行的节点。

从 https://github.com/orglen/EdgeAPI 下载EdgeAPI源码；

从 https://github.com/orglen/EdgeCommon 下载EdgeCommon源码，如果已经下载则不需要重复下载；

将EdgeAdmin和EdgeCommon放在同一目录下；

转到 EdgeAPI 目录下；

执行 `go mod download` 下载项目依赖的源码；

复制 `build/configs/api.template.yaml` 到 `build/configs/api.yaml`，如果 `api.yaml` 已经存在则无需重复复制；然后修改其中的配置 nodeId 为API节点的ID，secret 为API节点的密钥；如果还没有创建过API节点，则可以在修改数据库配置（第7步）后，通过执行 `go run -tags community cmd/edge-api/main.go setup -api-node-protocol=http -api-node-host=127.0.0.1 -api-node-port=8003` 初始化数据库，然后在执行后的控制台提示或者数据库 edgeAPINodes中获取节点ID（字段uniqueId）和密钥（字段secret）；

商业版源码请将 `-tags community` 换成 `-tags plus`

复制 `build/configs/db.template.yaml 到 build/configs/db.yaml`，将其中的 `prod` 修改为 `dev`，并修改其中的数据库配置，通常是用户名、密码、MySQL数据库地址和端口；

运行 `go run -tags community cmd/edge-api/main.go`

商业版源码请将 `-tags community` 换成 `-tags plus`

如果想编译整个项目，请参考 `build/build.sh`，比如可以运行：

`./build.sh linux amd64`

来编译linux/amd64版本的项目压缩包，编译后生成的压缩包可以在dist目录下找到。
如果出现amd64之外的节点编译报错时，可以修改build.sh脚本，修改其中的：


`NODE_ARCHITECTS=("amd64" "386" "arm64" "mips64" "mips64le")`
为
`NODE_ARCHITECTS=("amd64")`

这样只编译amd64，大部分Linux系统应该是支持的。


自动集成数据库结构变更


如果你修改了数据库结构，希望用户在安装时自动升级老的数据库，你需要运行 build/sql.sh 脚本，自动从你的数据库中生成新的结构代码（internal/setup/sql.go文件），然后再运行build.sh来重新编译API节点。

对于开源版本，如果你想每次运行编译脚本的时候都自动运行sql.sh，可以修改build.sh中的：


```sh
if [ $TAG = "plus" ]; then
	echo "building sql ..."
	${ROOT}/sql.sh
fi
```

修改为：

```sh
if [ $TAG = "community" ]; then
	echo "building sql ..."
	${ROOT}/sql.sh
fi
```

即可