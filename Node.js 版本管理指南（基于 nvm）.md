# **Node.js 版本管理指南（基于 nvm）**



## **目录**

- [1. 安装 nvm](#1-安装-nvm)
- [2. 常用 nvm 命令](#2-常用-nvm-命令)
- [3. 查看已安装及远程版本](#3-查看已安装及远程版本)
- [4. 安装与卸载 Node.js 版本](#4-安装与卸载-nodejs-版本)
- [5. 切换与设置默认版本](#5-切换与设置默认版本)
- [6. 解决 nvm 中的 N/A 问题](#6-解决-nvm-中的-na-问题)
- [7. 推荐 LTS 版本列表](#7-推荐-lts-版本列表)



## **1. 安装 nvm**



```sh
# 安装脚本（macOS / Linux）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash

# 安装完成后，确保以下配置已添加到 ~/.zshrc 或 ~/.bashrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```



## **2. 常用 nvm 命令**



| **命令**                    | **说明**               |
| --------------------------- | ---------------------- |
| nvm install <version>       | 安装指定版本 Node.js   |
| nvm install --lts           | 安装最新 LTS 版本      |
| nvm uninstall <version>     | 卸载指定版本           |
| nvm use <version>           | 切换到指定版本         |
| nvm alias default <version> | 设置默认版本           |
| nvm ls                      | 查看本地安装的版本     |
| nvm ls-remote               | 查看远程所有可用版本   |
| nvm ls-remote --lts         | 查看远程所有 LTS 版本  |
| nvm current                 | 显示当前正在使用的版本 |



## **3. 查看已安装及远程版本**



```sh
nvm ls                # 查看本地安装的版本
nvm ls-remote         # 查看远程可用版本
nvm ls-remote --lts   # 查看远程 LTS 版本
```



## **4. 安装与卸载 Node.js 版本**



```sh
nvm install 18.20.8    # 安装指定版本
nvm install --lts=hydrogen  # 安装指定代号的 LTS 版本
nvm uninstall 16.20.2  # 卸载版本
```



## **5. 切换与设置默认版本**



```sh
nvm use 20.19.3              # 临时切换版本
nvm alias default 20.19.3    # 设置默认版本（终端启动时默认）
nvm current                  # 查看当前使用的版本
```



## **6. 解决 nvm 中的 N/A 问题**



- N/A 表示该版本未安装，本地缺少对应版本。
- 通过安装对应版本解决：



```sh
nvm install <version>
# 或
nvm install --lts=<codename>
```



## **7. 推荐 LTS 版本列表**



| **代号（Codename）** | **版本号** | **生命周期状态**   |
| -------------------- | ---------- | ------------------ |
| argon                | v4.9.1     | 已结束             |
| boron                | v6.17.1    | 已结束             |
| carbon               | v8.17.0    | 已结束             |
| dubnium              | v10.24.1   | 已结束             |
| erbium               | v12.22.12  | 已结束             |
| fermium              | v14.21.3   | 已结束             |
| gallium              | v16.20.2   | 已结束             |
| hydrogen             | v18.20.8   | 维护中             |
| iron                 | v20.19.3   | 维护中             |
| jod                  | v22.17.0   | 维护中（即将结束） |

