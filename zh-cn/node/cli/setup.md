# 安装 NEO-CLI

本文将介绍如何安装 NEO-CLI 的官方发布程序包以及启动 NEO 节点。你也可以选择直接从 GitHub 上下载 NEO-CLI 源代码并发布成可执行文件。如果使用 macOS，则只能通过后一种方式安装运行 NEO-CLI。相关信息请参考 [发布 NEO-CLI 源码](publish.md)。

## 配置要求

建议运行 NEO-CLI 的计算机配置固态硬盘，以免同步区块数据时速度太慢。

## 安装客户端

1. 在 GitHub 上下载系统对应的 [Neo-CLI](https://github.com/neo-project/neo-cli/releases) 程序包并解压。

2. 对于 Linux 系统，需要安装 LevelDB 和 SQLite3 开发包。例如，在 ubuntu 17.10 上输入以下命令：

   ```
   sudo apt-get install libleveldb-dev sqlite3 libsqlite3-dev libunwind8-dev
   ```

   对于 Windows 系统，[Neo-CLI](https://github.com/neo-project/neo-cli/releases) 的安装包中已经包含了 LevelDB，可跳过该步骤。  

## 修改配置文件

在启动 NEO-CLI 前需先配置 config.json 文件，开启自动绑定并打开钱包功能，钱包打开后才可以调用与钱包相关的 API。下面是一个标准设置的例子，其中 Path 是钱包的路径，Password 是钱包的密码，IsActive 设为 true 表示允许自动打开钱包。

```
  {
  "ApplicationConfiguration": {
    "Paths": {
      "Chain": "Chain_{0}",
      "Index": "Index_{0}"
    },
    "P2P": {
      "Port": 10333,
      "WsPort": 10334
    },
    "RPC": {
      "BindAddress": "127.0.0.1",
      "Port": 10332,
      "SslCert": "",
      "SslCertPassword": ""
    },
    "UnlockWallet": {
      "Path": "wallet.json",
      "Password": "11111111",
      "StartConsensus": false,
      "IsActive": true
    }
  }
}
```
> [!Note]
>
> 在 NEO-CLI 2.9.1 中新增了 BindAddress 选项，默认为本地 127.0.0.1。可以绑定指定网卡的 ipv4 地址以允许远程调用 RPC。若没有指定对象，则可以设成 0.0.0.0。
   
## 安装插件

从 NEO-CLI 2.9.0 开始，一些附加功能被独立封装在插件中用以调用，目的是为了提升节点的安全性，稳定性和灵活性。
请参见 [NEO 客户端插件](../plugin.md)选取所需要的插件进行下载。然后在 neo-cli 根目录新建 Plugins 文件夹（注意首字母大写），将解压出来的插件拷贝到其中。

![plugins.png](../../../assets/plugins.png)

对于 NEO-CLI 2.9.4 及之后版本，支持使用内部命令自动下载或卸载插件，操作更为简便。例如：

```
install ImportBlocks
uninstall ApplicationLogs

```
在安装完毕后，请重启 NEO-CLI 以使插件生效。

## 启动 NEO 节点

打开命令行，定位到 NEO-CLI 所在目录，输入以下命令启动 NEO 节点 。

**Windows 10**:

```
dotnet neo-cli.dll
```

或

```
neo-cli.exe
```

**Linux (ubuntu 17.10)**:

```
./neo-cli
```

或

   ```
dotnet neo-cli.dll
   ```

> [!Note]
>
> 如果使用 dotnet，需要先安装 .net core 环境。

如果想在启动节点的同时启动 API 服务，可以输入参数 `--rpc`  或 `/rpc` 或 `-r`

   ```
neo-cli.dll --rpc
   ```
仅连接配置文件中的种子节点，可以输入参数 `--nopeers`

   ```
neo-cli.dll --nopeers
   ```

如果你想让外部程序访问该节点的 API 需要开放防火墙端口：10331-10334, 20331-20334 

> [!Important]
>
> 请注意，如果开通了 API 服务，并且在 NEO-CLI 中打开钱包的话，需要设置防火墙策略，例如设置防火墙的白名单，这些端口仅对白名单的 IP 地址开放。如果完全对外开放，其它人可能会通过 API 导出私钥或者进行转账。

## 快速同步区块数据

客户端运行时会自动同步区块数据，打开钱包时也会自动同步钱包数据，当同步完成后才可以正常使用客户端以及查看钱包内资产。由于区块链数据庞大，初次同步时等待时间通常很久，建议采用离线同步包进行同步，相关信息，请参见 [快速同步区块数据](../../network/syncblocks.md)。
