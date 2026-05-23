# Linux 考试电脑：VS Code 远程连接 GPU 开发机完整流程

本文档用于明天在 Linux 考试电脑上配置 VS Code、Continue、DeepSeek API，并通过 VS Code Remote SSH 连接远程 GPU 开发机。

核心思路：

- 本地 Linux 电脑：负责打开 VS Code、安装插件、发起 SSH 连接。
- 远程开发机：真正有 NVIDIA A100 显卡，负责运行代码、训练、推理、评测、保存结果。
- `/vepfs-readonly`：远程开发机上的题目只读目录。
- `/vepfs`：远程开发机上的个人可写目录，代码和结果都应放这里。

## 1. 需要安装的 VS Code 插件

### 必装：Remote - SSH

插件名称：

```text
Remote - SSH
```

插件 ID：

```text
ms-vscode-remote.remote-ssh
```

用途：

- 让 VS Code 连接远程 Linux 开发机。
- 在 VS Code 中直接打开远程目录。
- 在远程服务器上编辑代码、运行终端命令、使用 GPU。

命令行安装：

```bash
code --install-extension ms-vscode-remote.remote-ssh
```

如果想一次安装完整远程开发扩展包，也可以安装：

```bash
code --install-extension ms-vscode-remote.vscode-remote-extensionpack
```

但本场景只安装 `Remote - SSH` 通常就够了。

### 可选：Continue

插件名称：

```text
Continue
```

插件 ID：

```text
Continue.continue
```

用途：

- 在 VS Code 中使用 DeepSeek API 辅助读代码、改代码、解释报错。

命令行安装：

```bash
code --install-extension Continue.continue
```

## 2. 本地终端先测试 SSH

考试说明中给出的远程连接方式是：

```bash
ssh dev
```

这说明考试电脑大概率已经提前配置好了 SSH Host 别名 `dev`。

在本地 Linux 终端执行：

```bash
ssh dev
```

如果能登录远程开发机，继续执行：

```bash
nvidia-smi
```

如果看到类似 `NVIDIA A100`、显存 `80G` 的信息，说明已经进入 GPU 开发机。

退出远程服务器：

```bash
exit
```

这一阶段是在本地终端里做的，只是为了确认远程服务器能连上。

## 3. 用 VS Code 连接远程开发机

打开 VS Code，按：

```text
Ctrl + Shift + P
```

输入并选择：

```text
Remote-SSH: Connect to Host...
```

选择或输入：

```text
dev
```

第一次连接时，如果 VS Code 询问远程系统类型，选择：

```text
Linux
```

等待 VS Code 自动在远程开发机上安装 VS Code Server。

连接成功后，VS Code 左下角通常会显示：

```text
SSH: dev
```

看到这个标记，说明当前 VS Code 窗口已经在远程开发机环境中工作。

## 4. 打开远程工作目录

连接成功后，在 VS Code 中选择：

```text
File -> Open Folder
```

建议打开：

```text
/vepfs
```

考试说明中的目录含义：

```text
/vepfs-readonly
```

只读目录，里面有题目文件，例如：

```text
problem1
problem2
problem3
problem4
```

```text
/vepfs
```

个人可写目录，用于保存：

- 代码
- 数据处理结果
- 模型 checkpoint
- 推理结果
- 最终提交文件

## 5. 复制题目到可写目录

不要直接在 `/vepfs-readonly` 中改文件。

在 VS Code 的远程终端中执行：

```bash
ls /vepfs-readonly
ls /vepfs
```

复制四道题到自己的可写目录：

```bash
cp -r /vepfs-readonly/problem1 /vepfs/
cp -r /vepfs-readonly/problem2 /vepfs/
cp -r /vepfs-readonly/problem3 /vepfs/
cp -r /vepfs-readonly/problem4 /vepfs/
```

进入某一道题：

```bash
cd /vepfs/problem1
ls
```

之后所有代码修改、运行、输出都在 `/vepfs/problem1`、`/vepfs/problem2` 等目录中进行。

## 6. 本地终端和远程终端怎么区分

### 本地终端

本地终端指的是考试电脑自己的 Linux 终端。

适合做：

- 安装 VS Code 插件。
- 测试 `ssh dev` 能不能连上。
- 排查 SSH 连接问题。
- 打开 VS Code。

常用命令：

```bash
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension Continue.continue
ssh dev
exit
```

一般不要在本地终端里写题目代码或跑训练，因为本地电脑通常没有 A100 显卡，也不一定挂载了题目目录。

### 远程终端

远程终端指的是：

- 通过 `ssh dev` 登录后的终端；或
- VS Code 已连接 `SSH: dev` 后打开的 Terminal。

适合做：

- 复制题目。
- 修改和运行代码。
- 跑训练、推理、评测。
- 检查 GPU。
- 保存模型和提交结果。

常用命令：

```bash
hostname
pwd
ls
ls /vepfs-readonly
ls /vepfs
nvidia-smi
cd /vepfs/problem1
python3 train.py
python3 infer.py
```

判断当前是不是远程环境：

```bash
hostname
nvidia-smi
pwd
```

如果 `nvidia-smi` 能看到 A100，说明在远程 GPU 开发机。

如果 `pwd` 显示 `/vepfs/...`，说明在远程挂载目录里。

## 7. 配置 Continue + DeepSeek API

如果需要在 VS Code 中使用 Continue，先安装插件：

```bash
code --install-extension Continue.continue
```

创建 Continue 配置目录：

```bash
mkdir -p ~/.continue
```

编辑配置文件：

```bash
nano ~/.continue/config.yaml
```

写入以下内容。

注意：把 `YOUR_DEEPSEEK_API_KEY` 替换成自己的 DeepSeek API Key。

```yaml
name: DeepSeek Continue
version: 1.0.0
schema: v1

models:
  - name: DeepSeek V4 Flash
    provider: deepseek
    model: deepseek-v4-flash
    apiKey: YOUR_DEEPSEEK_API_KEY
    roles:
      - chat
      - edit
      - apply
      - summarize
    capabilities:
      - tool_use
    defaultCompletionOptions:
      temperature: 0.2
      maxTokens: 4096

  - name: DeepSeek V4 Pro
    provider: deepseek
    model: deepseek-v4-pro
    apiKey: YOUR_DEEPSEEK_API_KEY
    roles:
      - chat
      - edit
      - apply
      - summarize
    capabilities:
      - tool_use
    defaultCompletionOptions:
      temperature: 0.2
      maxTokens: 4096
```

保存后，在 VS Code 中执行：

```text
Ctrl + Shift + P
Developer: Reload Window
```

打开 Continue 面板，选择：

```text
DeepSeek V4 Flash
```

或：

```text
DeepSeek V4 Pro
```

## 8. 常用 Linux 命令速查

### 查看当前位置

```bash
pwd
```

### 查看目录内容

```bash
ls
ls -la
```

### 进入目录

```bash
cd /vepfs
cd /vepfs/problem1
cd ..
```

### 复制文件或目录

复制文件：

```bash
cp source.txt target.txt
```

复制目录：

```bash
cp -r /vepfs-readonly/problem1 /vepfs/
```

### 创建目录

```bash
mkdir output
mkdir -p checkpoints
```

### 查看文件内容

```bash
cat README.md
less README.md
sed -n '1,120p' README.md
```

### 搜索文件名

```bash
find . -name "*.py"
find /vepfs/problem1 -type f
```

### 搜索文件内容

如果有 `rg`：

```bash
rg "TODO"
rg "train"
rg "checkpoint"
```

如果没有 `rg`：

```bash
grep -R "TODO" .
grep -R "train" .
```

### 查看 Python 版本

```bash
python --version
python3 --version
```

### 查看 Python 包

```bash
python3 -m pip list
python3 -m pip show torch
```

### 检查 GPU

```bash
nvidia-smi
```

### 检查 PyTorch 是否能使用 GPU

```bash
python3 -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'no cuda')"
```

### 运行 Python 脚本

```bash
python3 train.py
python3 infer.py
python3 main.py
```

### 后台运行训练

如果允许后台运行，可以使用：

```bash
nohup python3 train.py > train.log 2>&1 &
```

查看日志：

```bash
tail -f train.log
```

查看后台进程：

```bash
ps aux | grep train.py
```

停止进程：

```bash
kill PID
```

其中 `PID` 是进程号。

### 查看磁盘空间

```bash
df -h
du -sh /vepfs/*
```

### 查看文件大小

```bash
ls -lh
du -sh checkpoints
```

## 9. 推荐明天实际执行顺序

### 第一步：本地 Linux 终端测试远程连接

```bash
ssh dev
nvidia-smi
exit
```

### 第二步：本地安装 VS Code 插件

```bash
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension Continue.continue
```

如果扩展市场无法访问，用 VSIX 离线安装：

```bash
code --install-extension /path/to/continue.vsix
code --install-extension /path/to/remote-ssh.vsix
```

### 第三步：VS Code 连接远程开发机

```text
Ctrl + Shift + P
Remote-SSH: Connect to Host...
dev
```

确认左下角出现：

```text
SSH: dev
```

### 第四步：打开远程目录

```text
File -> Open Folder
/vepfs
```

### 第五步：远程终端复制题目

```bash
ls /vepfs-readonly
cp -r /vepfs-readonly/problem1 /vepfs/
cp -r /vepfs-readonly/problem2 /vepfs/
cp -r /vepfs-readonly/problem3 /vepfs/
cp -r /vepfs-readonly/problem4 /vepfs/
```

### 第六步：开始做题

```bash
cd /vepfs/problem1
ls
nvidia-smi
python3 --version
```

根据题目说明运行对应脚本，例如：

```bash
python3 train.py
python3 infer.py
```

## 10. 最重要的原则

不要在这里改文件：

```text
/vepfs-readonly
```

应该把题目复制到这里再改：

```text
/vepfs
```

需要 GPU 的任务必须在远程开发机上做：

- 训练
- 推理
- 评测
- 生成 checkpoint
- 生成提交文件

本地 Linux 电脑只负责：

- 安装 VS Code 插件
- 连接远程服务器
- 显示 VS Code 界面
- 配置 Continue

判断是否在远程开发机的最简单方法：

```bash
nvidia-smi
```

能看到 A100，就在远程 GPU 环境。
