<p align="center"><img src="https://dydx.exchange/icon.svg?" width="256" /></p>

<h1 align="center">dYdX 链协议</h1>

<div align="center">
  <a href="https://github.com/dydxprotocol/v4-chain/actions/workflows/protocol-test.yml?query=branch%3Amain" style="text-decoration:none;">
    <img src="https://github.com/dydxprotocol/v4-chain/actions/workflows/protocol-test.yml/badge.svg?branch=main" />
  </a>
</div>

使用 Cosmos SDK 和 CometBFT 构建的 dYdX 主权区块链。

TODO(CORE-512): 添加关于 dYdX 链的信息/资源。[文档](https://www.notion.so/dydx/V4-36a9f30eee1d478cb88e0c50860fdbee)

## 开始使用

### 安装

1. 安装 Go `1.22` [点击这里](https://go.dev/dl/)。
  - 或者使用 Homebrew (`brew install go@1.22`)
2. 安装 Docker Desktop [点击这里](https://www.docker.com/products/docker-desktop/)
3. 运行 `make install` 安装项目依赖。

### 有用的开发命令

* `make test` 运行单元测试。
* `make lint` 检查源代码（`make lint-fix` 同时修复问题）。
* `make build` 构建源代码。
* `make mock-gen` 为 [mocks/Makefile](https://github.com/dydxprotocol/v4/tree/main/mocks/Makefile) 中列出的文件生成模拟对象。更多关于模拟的信息请查看[这里](https://github.com/dydxprotocol/v4/tree/main/mocks/README.md)。

### 本地运行链

要求：确保您运行的是 docker-compose 版本 `1.30.x` 或更新版本，以及 Docker 引擎版本 `20.10.x` 或更新版本。

您可以通过几个简单的命令快速测试对 dYdX 链的更改：

1. 对您想要测试的 dYdX 链代码进行任何更改

2. 准备好测试时，运行 `make localnet-start`（或 `make localnet-startd` 以无头模式运行网络）
    - 这首先会将您的所有更改编译到名为 `dydxprotocol-base` 的 docker 镜像中（约90秒）
    - 然后您将运行一个包含您更改的本地网络！
    - 注意，这些命令将**重置链**到创世状态

3. 要删除所有区块历史并从头开始，重新运行 `make localnet-start` 或 `make localnet-startd`。

4. 要停止链但保持状态，运行 `make localnet-stop`。要使用之前的状态重启协议，运行 `make localnet-continue`。

#### 部署到 AWS 测试网

合并到 `main` 分支会自动触发构建新的 Docker 容器镜像并推送到 ECR。镜像推送到 ECR 后，目前需要运行 Terraform Cloud 来将新容器部署到 ECS。

以下命令可用于本地构建并推送容器到 ECR。

* `make aws-push-dev` 本地构建并推送容器镜像到 "dev" 环境。
* `make aws-push-dev2` 本地构建并推送容器镜像到 "dev2" 环境。
* `make aws-push-dev3` 本地构建并推送容器镜像到 "dev3" 环境。
* `make aws-push-dev4` 本地构建并推送容器镜像到 "dev4" 环境。
* `make aws-push-dev5` 本地构建并推送容器镜像到 "dev5" 环境。
* `make aws-push-staging` 本地构建并推送容器镜像到 "staging" 环境。

#### 代码检查

我们使用 [`yamllint`](https://github.com/adrienverge/yamllint) 来检查 YAML 文件。使用特定 `yamllint` 操作的说明链接如下：
- [安装 `yamllint`](https://yamllint.readthedocs.io/en/latest/quickstart.html#installing-yamllint)。
- [运行 `yamllint`](https://yamllint.readthedocs.io/en/latest/quickstart.html#running-yamllint)。
- [配置 `yamllint`](https://yamllint.readthedocs.io/en/latest/configuration.html)。

我们目前在 [`Lint` CI 作业](https://github.com/dydxprotocol/v4/blob/c5ec83f074b4ff997d71a6f5dc486579ea112600/.github/workflows/lint.yml) 中检查以下 YAML 文件：
- `.golangci.yml`。
- `.github/workflows/*.yml`。
  - 注意这包括 `.github/workflows` 目录中所有以 `yml` 文件扩展名结尾的文件。
- `buf.work.yaml`。

#### 协议缓冲区

协议缓冲区可以在 `../proto/` [这里](https://github.com/dydxprotocol/v4-chain/tree/main/proto) 找到。

#### 创世

您可以在 `testing/genesis.sh` 中找到本地链的创世数据。这决定了运行 `make localnet-start` 时链的起始应用状态。我们目前使用 BTC 和 ETH 永续合约和价格启动链，但可以轻松添加另一个永续合约和市场，如下所示：

```
...
	dasel put string -f "$GENESIS" '.app_state.perpetuals.perpetuals.[2].ticker' 'LINK-USD'
	dasel put int -f "$GENESIS" '.app_state.perpetuals.perpetuals.[2].market_id' '1'
	dasel put int -f "$GENESIS" '.app_state.perpetuals.perpetuals.[2].atomic_resolution' -v '-9'
	dasel put int -f "$GENESIS" '.app_state.perpetuals.perpetuals.[2].default_funding_ppm' '0'
	dasel put int -f "$GENESIS" '.app_state.perpetuals.perpetuals.[2].initial_margin_ppm' '50000'     # 5 %
	dasel put int -f "$GENESIS" '.app_state.perpetuals.perpetuals.[2].maintenance_fraction_ppm' '600000' # 3 % (60% of IM)
...
	dasel put string -f "$GENESIS" '.app_state.prices.markets.[2].pair' 'LINK-USD'
	dasel put int -f "$GENESIS" '.app_state.prices.markets.[2].exponent' -v '-6'
	dasel put int -f "$GENESIS" '.app_state.prices.markets.[2].min_exchanges' '1'
	dasel put int -f "$GENESIS" '.app_state.prices.markets.[2].min_price_change_ppm' '50'
	dasel put int -f "$GENESIS" '.app_state.prices.markets.[2].price' '3000000000' # $3,000 = 1 ETH.
	dasel put int -f "$GENESIS" '.app_state.prices.markets.[2].exchanges.[0]' '0'
```

另一个可以类似修改的模块是子账户。我们可以通过更新 `./testing/local.sh` 中的 `TEST_ACCOUNTS` 数组来添加另一个子账户。

## 调试技巧

### 设置密钥链
要运行以下命令，您需要导入在 [testnet-local/local.sh](https://github.com/dydxprotocol/v4/blob/main/testing/testnet-local/local.sh) 中指定的测试账户的私钥。运行以下命令并输入来自 `MNEMONICS` 的相应12个单词字符串。生成的地址应与 `TEST_ACCOUNTS` 中的地址匹配。

```sh
./build/dydxprotocold keys add alice --recover

./build/dydxprotocold keys add bob --recover
```

### 本地发送测试交易
有时发送交易到本地链来观察通过 API 的 Cosmos 行为（如事件）是有帮助的。在 `clob` `v0.1` 完成之前，您可以使用默认的 Cosmos `bank` 模块在 `genesis.sh` 文件中定义的两个账户之间转移资产。

```sh
./build/dydxprotocold tx bank send dydx199tqg4wdlnu4qjlxchpd7seg454937hjrknju4 dydx10fx7sy6ywd5senxae9dwytf8jxek3t2gcen2vs 100usdc
```

### 本地下测试订单

有时发送交易到本地链来测试订单下单和匹配是有帮助的。连续运行以下两个命令以在两个账户之间匹配订单。

```sh
./build/dydxprotocold tx clob place-order dydx199tqg4wdlnu4qjlxchpd7seg454937hjrknju4 0 0 0 1 10 10000 20 --from alice --chain-id localdydxprotocol
./build/dydxprotocold tx clob place-order dydx10fx7sy6ywd5senxae9dwytf8jxek3t2gcen2vs 0 0 0 2 10 10000 20 --from bob --chain-id localdydxprotocol
```

运行以下命令取消订单。

```sh
./build/dydxprotocold tx clob cancel-order dydx199tqg4wdlnu4qjlxchpd7seg454937hjrknju4 10 0 20 --from alice
```

### 本地查询链

在通过 `make localnet-start` 运行开发服务器时，您可以使用 Tendermint API 在本地进行查询。支持[这里](https://docs.tendermint.com/v0.37/rpc/#/Info/block)列出的所有端点。例如，要获取高度为2的区块：`curl -X GET "localhost:26657/block?height=2"`。

### 更新本地标志
在本地调试或检查链的行为时，您可能希望修改传递给 `dydxprotocold` 的标志。您可以通过在本地修改 `docker-compose.yml` 文件的 `entrypoint` 部分来更改这些传入的标志。

### 本地启用更详细的日志记录
参考上面的部分并将 `log_level` 更改为 `trace`。注意 `trace` 可能会很嘈杂，因为它会将每个区块提案、消息和提交的区块记录到标准输出。

### 调试 Cosmos SDK 中的行为
有时能够在 `cosmos-sdk` 本身中输出日志或修改行为是有用的。要做到这一点，请在代表此存储库中 [`go.mod` 文件](https://github.com/dydxprotocol/v4/blob/main/go.mod) 中指定版本的分支上本地检出 [cosmos-sdk](https://github.com/cosmos/cosmos-sdk)。

```sh
git clone git@github.com:cosmos/cosmos-sdk.git
git checkout v0.47.0-alpha2
```

克隆存储库后，您可以修改 `v4-chain` 中的 `go.mod` 文件以包含一个 [_replace 指令_](https://go.dev/ref/mod#go-mod-file-replace)，该指令将 `cosmos-sdk` 本地指向您的本地版本的 `cosmos-sdk`。示例：

```diff
replace (
+	github.com/cosmos/cosmos-sdk v0.47.0-alpha2 => /Users/bryce/projects/cosmos-sdk
	github.com/keybase/go-keychain => github.com/99designs/go-keychain v0.0.0-20191008050251-8e49817e8af4
	google.golang.org/grpc => google.golang.org/grpc v1.50.1
)
```

现在运行 `make localnet-start` 将包含您对本地 `cosmos-sdk` 存储库所做的任何更改。

### 减慢区块速度
出于调试目的，更改区块提交的速度可能很有用。这对于更容易在同一区块内发送多个交易，或者只是减少 `--verbose` 输出的噪音（包括每次提交新区块时的输出）很有用。

例如，对于 `1分钟` 的区块时间，在此存储库中本地将以下行添加到您的 `local.sh` 文件：

```yaml
# 在预投票 nil 之前我们等待提案区块的时间
dasel put string -f "$CONFIG_FOLDER"/config.toml '.consensus.timeout_propose' '60s'

# 提交区块后，在开始新高度之前我们等待的时间
# （这给我们机会接收更多预提交，即使我们已经有 +2/3）。
dasel put string -f "$CONFIG_FOLDER"/config.toml '.consensus.timeout_commit' '60s'
```

## CometBFT 分叉

我们当前的实现包含 CometBFT 的轻量级分叉。分叉可以在[这里](https://github.com/dydxprotocol/cometbft)找到。更新分叉的说明包含在那里。

## CosmosSDK 分叉

我们当前的实现包含 CosmosSDK 的轻量级分叉。分叉可以在[这里](https://github.com/dydxprotocol/cosmos-sdk)找到。更新分叉的说明包含在那里。

## 守护进程

守护进程是在 go-routines 中运行的后台进程，用于执行异步工作。守护进程可以使用各自的标志进行配置，例如 `price-daemon-enabled` 或 `price-daemon-delay-loop-ms`。

TODO(CORE-512): 更新守护进程标志

### 桥接守护进程

TODO(CORE-512): 添加详细信息

### 清算守护进程

TODO(CORE-512): 添加详细信息

### 价格馈送守护进程

价格馈送守护进程负责从第三方交易所（如币安）获取价格，并将这些价格发送到应用程序，然后由价格模块使用。价格馈送守护进程在应用程序启动时默认启动。

TODO(CORE-469): 使用新方法更新文档以覆盖参数

## 了解更多

- [Cosmos SDK 文档](https://docs.cosmos.network)
- [开发者聊天](https://discord.gg/H6wGTY8sxw)