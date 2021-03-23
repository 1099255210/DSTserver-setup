# DSTserver-setup

Record of the setup of Don't Starve Together.

## 文件结构

主/地上服务器 (Master)

- .klei
  - DoNotStarveTogether
    - Cluster_1
      - Master
        - server.ini
      - adminlist.txt
      - cluster_token.txt
      - cluster.ini
- lgsm
  - config-lgsm
    - dstserver
      - dstserver.cfg
- serverfiles
  - mods
    - dedicated_server_mods_setup.lua

副/地下服务器 (Cases)

- .klei
  - DoNotStarveTogether
    - Cluster_1
      - Cases
        - server.ini
      - adminlist.txt
      - cluster_token.txt
      - cluster.ini
- lgsm
  - config-lgsm
    - dstserver
      - dstserver.cfg
- serverfiles
  - mods
    - dedicated_server_mods_setup.lua

## 操作流程

### 对于存档的创建与修改

在自己的计算机上创建一个新的饥荒存档，记得打开所有你需要添加的 mod ，打开洞穴。

创建之后不要选人物，左下角直接离开游戏。找到存档的文件夹（应当在游戏内有打开存档文件夹的按钮），将其中的内容拷贝出来，应该包括：

```
Master/
Caves/
cluster_token.txt
cluster.ini
```

我们所要做的就是在同一个服务器的两个用户内分别运行游戏，一个运行 Master 也就是主世界，另外一个运行 Caves 也就是洞穴。要做到这一点，我们就需要把原本在一个文件夹里的 Master 和 Caves 分开，但为了链接它们又需要共同的令牌与设置。按照这个逻辑我们就可以分开它们：

主世界文件夹

```
Master/
cluster_token.txt
cluster.ini
```

洞穴文件夹

```
Master/
cluster_token.txt
cluster.ini
```

这就要求你保证这两个文件夹的 `cluster_token.txt` 和 `cluster.ini` 相同。这里的 `cluster.ini` 与你刚刚在游戏内创建时的房间设置一致，不必修改。但是 `cluster_token.txt` 中存放的是游戏服务器的令牌，当前是刚刚创建的房间的令牌，如果不修改则会导致服务器无法上线。因此我们需要重新注册一个服务器令牌。

在饥荒游戏主菜单左下角有账号按钮，点开弹出 klei 官方网站，点上方的 `游戏` 找到饥荒联机游戏服务器，创建新的服务器，里面的设置无需修改不用关心，直接复制令牌并替换 `cluster_token.txt` 里的内容即可。

### 对于服务器文件系统的操作

先切换至在服务器内创建两个用户

```shell
adduser master
adduser caves
```

分别在两个用户内安装 lgsm 并安装饥荒，我这里演示在 `master` 里操作的流程，对于 `caves` 是一样的：

```shell
su master
cd
wget -O linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh dstserver
./dstserver install
```

如果在安装中遇到 `raw.githubusercontent.com` 连接不上的问题可以在系统的 `etc/hosts` 里加上一行 `199.232.4.133 raw.githubusercontent.com` 再重试（或者重启）。

安装完全完成以后，需要修改 lgsm 启动项，进入 `lgsm/config-lgsm/dstserver` ，将 `_default.cfg` 里的内容粘贴进 `dstserver.cfg` 里，并修改这几行：

```
## Predefined Parameters | https://docs.linuxgsm.com/configuration/start-parameters
sharding="true"
master="true"
shard="Master"
cluster="Cluster_1"
cave="false"
```

注意，上面这段代码是 `master` 用户里的设置， `caves` 用户则与其不同，应当这样设置：

```
## Predefined Parameters | https://docs.linuxgsm.com/configuration/start-parameters
sharding="false"
master="false"
shard="Caves"
cluster="Cluster_1"
cave="true"
```

改完启动项，我们需要将存档分别放入对应的服务器内。进入 `.klei/DoNotStarveTogether/Cluster_1` 文件夹（注意：这个文件夹默认是隐藏的！），然后删除里面的所有内容，将上边分好的存档分别放入。`master` 用户里放 `Master` 存档， `caves` 用户里放 `Caves` 存档。

最后一步也是比较困难的一步：修改 mod 配置文件。进入 `serverfiles/mods` ，打开 `dedicated_server_mods_setup.lua` 文件，将自己存档里用到的 mod 以这种格式添加进文件：

```
ServerModSetup("1271089343")
ServerModSetup("1898181913")
ServerModSetup("2078243581")
```

每一行里的数字对应一个 mod，这个编号可以从该 mod 创意工坊的链接末尾获得，也可以在你的存档文件里找到。打开 `Master/modoverrides.lua` ，这里会列出所有存档里加载的 mod 的设置。我截取以下片段作为例子：

```
["workshop-1185229307"]={
    configuration_options={ damage=2, noepic=false, phase=true, range=20, timeout=10 },
    enabled=true
  },
```

这里的 `1185229307` 就是该 mod 对应的编号，将这个编号以上述的 `ServerModeSetup("**********")` 添加即可。存档里用到的所有 mod 都要添加；`master` 和 `caves` 用户都需要添加。

最后一项操作可选：配置服务器管理员。服务器管理员可以执行控制服务器的指令，包括 T 键模组 也需要使用者拥有管理员权限。

饥荒里每一位玩家都有独一无二的编号标识。自己的编号可以在饥荒游戏主菜单左下角的 `账户` 里找到，是一个格式为 `KU_********` 的编号。

打开 `Master/adminlist.txt` 将编号添加进去即可，`Caves/adminlist.txt` 同理

完成了这些所有的配置基本就绪。接下来就可以尝试启动服务器了。

### 游戏服务器操作

先切换 `master` 用户进行操作：

```shell
cd
./dstserver start
./dstserver console

```

这里有两个操作，`start` 是启动服务器， `console` 是在 shell 里显示控制台，这样就可以查看服务器是否成功启动。如果最后看到如下内容，说明基本成功：

```
[**:**:**]Sim Paused
[**:**:**]Registering master server in *** lobby
```

此时可以另开一个 shell 窗口，切换 `caves` 用户，同样如此：

```shell
cd
./dstserver start
./dstserver console

```

待 `caves` 完全启动，观察 `master` 的控制台是否有链接到 `caves` 的输出信息，如果有的话，服务器就基本搭建完成了。这时候两个窗口都可以使用 `Ctrl + B` 再按 `D` 的方式安全退出控制台，这时候就可以关闭窗口了。如果要停止服务器，则分别输入 `./dstserver stop` 即可。

在游戏中想要快速连接到服务器，可以按 `~` 键进入游戏内控制台，输入 `c_connect("服务器地址")`。
