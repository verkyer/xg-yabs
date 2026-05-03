# VPS YABS 中文版

VPS YABS 是一个面向 Linux 服务器的性能基准测试脚本，用于快速查看 VPS 或独立服务器的磁盘、网络、CPU 和内存表现。

脚本会自动运行常见测试工具：使用 [fio](https://github.com/axboe/fio) 测试磁盘性能，使用 [iperf3](https://github.com/esnet/iperf) 测试网络吞吐，使用 [Geekbench](https://www.geekbench.com/) 测试 CPU/内存与整体系统性能。脚本不要求提前安装测试依赖，也不需要管理员权限即可运行。

## 如何运行

```sh
curl -sL https://raw.githubusercontent.com/verkyer/xg-yabs/master/vps-yabs.sh | bash
```

或：

```sh
wget -qO- https://raw.githubusercontent.com/verkyer/xg-yabs/master/vps-yabs.sh | bash
```

**本地 fio/iperf3 软件包**：如果被测试系统已经安装了 fio 和/或 iperf3，默认会优先使用本地软件包，而不是仓库中的预编译二进制文件。

**ARM 兼容性说明**：脚本已加入初步 ARM 兼容支持，但由于不同 ARM 设备测试覆盖有限，仍视为实验性功能。如遇到问题，建议记录系统架构和报错信息后反馈。

**高带宽使用提醒**：默认情况下，脚本会执行多个 iperf 网络测试，每个节点会尝试占满网络端口约 20 秒（上传和下载各约 10 秒）。低带宽服务器（例如 NAT VPS）建议使用 `-r` 参数减少 iperf 节点，或使用 `-i` 参数完全禁用网络测试。

**Windows 用户**：可以通过 [Windows Subsystem for Linux v2 (WSL 2)](https://learn.microsoft.com/en-us/windows/wsl/about) 在 Windows 系统上运行脚本。WSL1 无法正确运行脚本和相关二进制文件。

## 参数说明

```sh
curl -sL https://raw.githubusercontent.com/verkyer/xg-yabs/master/vps-yabs.sh | bash -s -- -flags
```

| 参数 | 说明 |
| ---- | ---- |
| `-b` | 强制使用仓库中的预编译二进制文件，而不是本地安装的软件包 |
| `-f`/`-d` | 禁用 fio 磁盘性能测试 |
| `-i` | 禁用 iperf 网络性能测试 |
| `-g` | 禁用 Geekbench 系统性能测试 |
| `-n` | 跳过网络信息查询和输出 |
| `-h` | 显示帮助信息、已识别参数以及本地 fio/iperf 检测状态 |
| `-r` | 减少 iperf 测试节点数量，以降低带宽消耗 |
| `-4` | 运行 Geekbench 4，并禁用默认 Geekbench 6 |
| `-5` | 运行 Geekbench 5，并禁用默认 Geekbench 6 |
| `-9` | 运行 Geekbench 4 和 5，而不是默认 Geekbench 6 |
| `-6` | 重新启用 Geekbench 6；如果同时使用 `-4`、`-5` 或 `-9`，`-6` 必须放在最后 |
| `-j` | 测试结束后在屏幕输出 JSON 结果 |
| `-w <filename>` | 将 JSON 结果写入指定文件 |
| `-s <url>` | 将 JSON 结果发送到指定 URL |
| `-p <servers>` | 指定自定义 iperf 服务器，格式为 `host:port_range:name:location:network_modes`，多个服务器使用逗号分隔 |

参数可以组合使用。例如 `-fg` 会跳过磁盘和系统性能测试，只执行网络相关测试。

**Geekbench 授权密钥**：如需在 Geekbench 测试中使用授权密钥，可在执行目录下创建 `geekbench.license` 文件，内容为邮箱和密钥：

```sh
echo "email@domain.com ABCDE-12345-FGHIJ-57890" > geekbench.license
```

## 提交 JSON 结果

脚本运行结果可以用 JSON 格式发送到你选择的基准测试结果网站。使用 `-s` 参数并传入提交地址即可：

```sh
curl -sL https://raw.githubusercontent.com/verkyer/xg-yabs/master/vps-yabs.sh | bash -s -- -s "https://example.com/yabs/post"
```

多个提交端点可以使用逗号连接，例如：`https://example.com/yabs/post,http://example.com/yabs2/post`。

支持提交 JSON 结果的网站：

| 网站 | 示例命令 |
| ---- | -------- |
| [YABSdb](https://yabsdb.com/) | `curl -sL https://raw.githubusercontent.com/verkyer/xg-yabs/master/vps-yabs.sh \| bash -s -- -s "https://yabsdb.com/add"` |
| [VPSBenchmarks](https://www.vpsbenchmarks.com/yabs/get_started) | `curl -sL https://raw.githubusercontent.com/verkyer/xg-yabs/master/vps-yabs.sh \| bash -s -- -s https://www.vpsbenchmarks.com/yabs/upload` |

示例 JSON 输出见 [bin/example.json](bin/example.json)。

## 测试项目

* **[fio](https://github.com/axboe/fio)**：用于评估磁盘随机读写性能。脚本会使用 4k、64k、512k 和 1m 块大小执行四组随机读写测试，并采用 50/50 读写比例。
* **[iperf3](https://github.com/esnet/iperf)**：用于测试不同地区的上传和下载速度。脚本使用 8 个并行线程测试双向网络速度。如果某个 iperf 服务器繁忙，脚本会在多次尝试后跳过该节点或方向。
* **[Geekbench](https://www.geekbench.com/)**：用于评估 CPU、内存和整体系统性能。脚本会输出 Geekbench 网页结果链接，方便查看完整测试和单项分数。认领结果用的 URL 会写入执行目录下的 `geekbench_claim.url` 文件。

## 安全提示

脚本会下载或运行外部二进制文件来完成部分性能测试。网络测试和磁盘测试可能使用仓库中的预编译 fio/iperf3 二进制文件；系统性能测试会下载 Geekbench 官方 tarball，解压后运行其中的二进制文件。

和运行任何来自互联网的脚本一样，请自行评估风险。二进制文件的版本、哈希和编译说明可参考 [bin/README.md](bin/README.md)。

## 示例输出

```text
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #
#              Yet-Another-Bench-Script              #
#                     v2026-04-29                    #
#          https://github.com/verkyer/xg-yabs         #
# ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## ## #

基础系统信息：
---------------------------------
运行时间   : 12 days, 3 hours, 20 minutes
处理器     : Intel(R) Xeon(R) CPU
CPU 核心   : 4 @ 2499.998 MHz
AES-NI     : ✔ Enabled
VM-x/AMD-V : ✔ Enabled
内存       : 7.8 GiB
Swap       : 0.0 KiB
磁盘       : 98.0 GiB
发行版     : Ubuntu 22.04.4 LTS
内核       : 5.15.0-generic
虚拟化类型 : KVM
IPv4/IPv6  : ✔ 在线 / ✔ 在线

fio 磁盘速度测试（混合读写 50/50）（分区 /dev/vda1）：
---------------------------------
块大小     | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ----
读取       | 405.41 MB/s (101.3k) | 407.96 MB/s   (6.3k)
写入       | 406.48 MB/s (101.6k) | 410.11 MB/s   (6.4k)
合计       | 811.90 MB/s (202.9k) | 818.08 MB/s  (12.7k)

YABS 已完成，耗时 12 分 49 秒
```

## 许可证

完整许可证文本请查看 [LICENSE](LICENSE)。
