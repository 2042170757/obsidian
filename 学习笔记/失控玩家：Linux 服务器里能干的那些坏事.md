---
title: 失控玩家：Linux 服务器里能干的那些坏事
date: 2026-04-30
tags:
  - Linux
  - 运维
  - 命令行
  - 服务器
---

# 失控玩家：Linux 服务器里能干的那些坏事

> [!abstract] 前言
> 这是一份 Linux 命令行速查 + 深度笔记。
> 基础部分适合初学者入门，进阶技巧和实战组合适合有经验的用户查漏补缺。每个难懂概念都配有比喻帮助理解，过时命令已标注替代方案。
>
> 整合日期：2026-04-30

---

## 第一组：基础目录/文件操作

> [!info] 从这里开始
> 这些是日常使用频率最高的命令，掌握它们就能在 Linux 里自由穿梭。

### cd -- 切换目录

**基础知识：**
- `cd`：回到当前用户 HOME 目录
- `cd ~`：回到**家目录**（home directory，如 `/home/username`），**不是根目录**
- `cd /`：切换到**根目录**（文件系统顶层）
- `cd ..`：回到上级目录
- `cd -`：回到上一次所在的目录（读取 `$OLDPWD` 变量）

> [!tip] 比喻：家目录 vs 根目录
> 根目录 `/` 像一栋大楼的大厅入口，是整个文件系统的起点。
> 家目录 `~` 像你自己租的那间房，每个用户都有自己的"房间"。
> root 用户的家目录是 `/root`，普通用户的家目录通常是 `/home/username`。

**常见误解**：初学者容易混淆"根目录"（`/`）、"root 用户的家目录"（`/root`）和"当前用户的家目录"（`~`），这是三个完全不同的概念。

**进阶技巧：**

1. **OLDPWD 环境变量**：`cd -` 的底层原理就是 `cd "$OLDPWD"`。可以用 `echo $OLDPWD` 查看上一个目录路径。

2. **CDPATH 自定义搜索路径**（类似 PATH 变量）：
   ```bash
   # 在 ~/.bashrc 中设置
   CDPATH=".:~:~/projects:/var/www"
   # 设置后，可以直接 cd myapp 跳转到 ~/projects/myapp
   ```
   【注意】CDPATH 可能导致 cd 行为不符合预期，建议在脚本中谨慎使用。

3. **pushd/popd 目录栈管理**：
   ```bash
   pushd /var/log    # 将目录压栈并切换
   pushd /etc        # 再压一个
   dirs -v           # 查看目录栈
   popd              # 弹出栈顶目录并切换
   ```

4. **cd -L vs cd -P**：

   | 选项 | 行为 | 说明 |
   |------|------|------|
   | `cd -L` | 遵循符号链接 | 默认行为，逻辑路径 |
   | `cd -P` | 解析符号链接到物理实际路径 | 物理路径 |

5. **实用别名**：
   ```bash
   alias ..='cd ..'
   alias ...='cd ../..'
   alias mkcd='f(){ mkdir -p "$1" && cd "$1" }; f'
   # cd 时自动 ls（慎用，可能影响脚本）
   cdls() { builtin cd "$@" && ls; }
   ```

6. **冷知识**：`cd` 是 shell 内建命令（builtin），不是外部可执行文件。用 `type cd` 可以验证。这意味着 `which cd` 通常找不到它，因为 `which` 只搜索外部命令。

---

### ls -- 查看目录内容

**基础知识：**
- `ls -l`：长格式列出（权限、所有者、大小、时间）
- `ls -a`：显示所有文件（含隐藏文件，以 `.` 开头，包括 `.` 和 `..`）
- `ls -A`：显示所有文件但排除 `.` 和 `..`（脚本中更实用）
- `ls -lh`：人性化显示文件大小（K/M/G）
- `ls -lt`：按修改时间排序（最新在前）
- `ls -lS`：按文件大小排序（最大在前）

**进阶技巧：**

1. **实用组合**：
   ```bash
   ls -lrt           # 按时间倒序排列，最新文件在底部（常用于看最近修改的文件）
   ls -lhS           # 按大小排序，最大文件在前
   ls -la --color=auto  # 带颜色区分文件类型
   ls -d */          # 只列出目录（不展开内容）
   ls -R             # 递归列出所有子目录
   ```

2. **ls -l 第一列文件类型标识符**：
   | 标识 | 类型 | 说明 |
   |------|------|------|
   | `-` | 普通文件 | 二进制文件、文本文件等 |
   | `d` | 目录 | |
   | `l` | 符号链接 | 软链接 |
   | `c` | 字符设备 | 终端、键盘等 |
   | `b` | 块设备 | 硬盘、U盘等 |
   | `s` | 套接字 socket | 进程间通信 |
   | `p` | 管道 FIFO | 进程间通信 |

3. **按时间精确查找**：
   ```bash
   ls -l --time=atime    # 显示访问时间而非修改时间
   ls -l --time=ctime    # 显示状态变更时间
   ```

4. **脚本中绕过 alias**：很多发行版将 `ls` 设置了 alias（如 `alias ls='ls --color=auto'`），在脚本中使用时可能需要用 `command ls` 或 `\ls` 来绕过 alias。

5. **冷知识**：`ll` 通常是 `ls -l` 的别名（alias），不是独立命令。在不同发行版中定义不同，用 `type ll` 确认。

---

### mv -- 移动/重命名

**基础知识：**
- `mv old.txt new.txt`：重命名
- `mv file.txt /target/path/`：移动文件到目录
- `mv file.txt /target/path/new_name.txt`：移动并重命名

**进阶技巧：**

1. **安全选项**：
   ```bash
   mv -i old.txt new.txt    # 覆盖前交互确认（推荐默认加上）
   mv -n old.txt new.txt    # 不覆盖已有文件（静默跳过）
   mv -u source target      # 只在源文件比目标新时才移动
   mv -b old.txt new.txt    # 覆盖前创建备份（~结尾）
   mv -v old.txt new.txt    # 显示操作详情
   ```

2. **批量移动**：
   ```bash
   mv *.txt /backup/                    # 移动所有txt文件
   mv -t /dest/ file1 file2 file3       # -t 先指定目标目录
   ```

**安全性**：`mv` 默认会静默覆盖目标文件，数据不可恢复。建议养成使用 `mv -i` 的习惯，或设置 alias `mv='mv -i'`。

**常见坑：**
- 移动目录时如果目标已存在，会把源目录移动到目标目录**内**，而非覆盖。建议先检查目标路径。
- `mv` 跨文件系统移动大文件时实际上会先复制再删除，速度较慢。此时考虑用 `rsync`。
- `mv` 在同一文件系统上是原子操作（rename），跨文件系统时等同于复制+删除。

---

### cp -- 复制文件/目录

**基础知识：**
- `cp file1 file2`：复制文件
- `cp -r dir1 dir2`：递归复制目录
- `cp -i`：覆盖前确认

**进阶技巧：**

1. **属性保留复制**：
   ```bash
   cp -a source/ dest/      # 保留所有属性（权限、时间、属主等），相当于 cp -dR --preserve=all
   cp -p file1 file2        # 保留权限和时间戳（不递归）
   ```

2. **增量复制**：
   ```bash
   cp -u *.txt backup/      # 只复制比目标更新的文件
   ```

3. **创建链接而非真正复制**：
   ```bash
   cp -l file1 file2        # 创建硬链接（节省空间）
   cp -s file1 file2        # 创建符号链接（快捷方式）
   ```

4. **大量小文件性能优化**：直接 `cp` 复制大量小文件效率极低。实测 10 万个 1KB 文件直接 cp 耗时约 3 分钟，而用 tar 管道方式仅需约 28 秒：
   ```bash
   tar -cf - source/ | tar -xf - -C dest/
   ```

5. **安全实践**：始终使用 `cp -i`（交互确认），避免误覆盖。建议 alias `cp='cp -i'`。

**常见误解**：`cp -r dir1 dir2` 如果 `dir2` 已存在，结果是 `dir2/dir1/...` 而非 `dir2/...`。这个行为经常让人困惑。

---

### mkdir -- 创建目录

**基础知识：**
- `mkdir dir1`：创建单层目录
- `mkdir -p a/b/c`：递归创建多级目录

**进阶技巧：**

1. **批量创建**：
   ```bash
   mkdir -p /tmp/test/{a1,b1}/{c1,d1}    # 使用花括号扩展，创建所有组合
   mkdir -p project/{src,lib,bin,doc}     # 创建项目目录结构
   ```

2. **创建时指定权限**：
   ```bash
   mkdir -m 700 secure_dir    # 创建目录并直接设置权限
   mkdir -m 755 test          # 创建时指定权限
   ```

3. **冷知识**：`mkdir -p` 在目录已存在时不会报错，这使它非常适合用在脚本中。

---

### touch -- 创建空文件/更新时间

**基础知识：**
- `touch file.txt`：若文件不存在则创建空文件，若已存在则更新时间戳

**重要说明**：`touch` 的主要设计意图是**更新文件的访问和修改时间戳**。创建空文件只是副产品。在脚本中，`> newfile` 或 `: >> newfile` 也可以创建空文件。

**进阶技巧：**

1. **批量创建**：
   ```bash
   touch file{1..100}.txt            # 创建 file1.txt 到 file100.txt
   touch {a,b,c}.log                 # 创建 a.log b.log c.log
   ```

2. **修改时间戳但不创建文件**：
   ```bash
   touch -c file.txt         # 文件不存在时不创建
   touch -t 202501011200 file.txt   # 设置指定时间
   touch -a file.txt         # 只修改访问时间（access time）
   touch -m file.txt         # 只修改修改时间（modify time）
   ```

3. **实用场景**：用 touch 更新文件时间后，结合 `make` 命令可以触发重新编译。

---

### cat -- 查看文件内容

**基础知识：**
- `cat file.txt`：查看文件内容
- `cat -n file.txt`：显示行号

**进阶技巧：**

1. **实用选项**：
   ```bash
   cat -A file.txt      # 显示所有字符（包括控制字符、$结尾标记）
   cat -s file.txt      # 压缩连续空行为一行
   cat -b file.txt      # 只对非空行编号
   ```

2. **合并文件**：
   ```bash
   cat file1.txt file2.txt > combined.txt    # 合并两个文件
   cat *.log > all_logs.txt                   # 合并所有日志
   ```

3. **追加内容**：
   ```bash
   cat >> file.txt <<EOF
   这是追加的内容
   EOF
   ```

4. **相关命令对比**：
   | 命令 | 功能 | 适用场景 |
   |------|------|----------|
   | `cat` | 输出全部内容 | 小文件 |
   | `tac` | 反向逐行输出（cat 反写） | 查看最新日志行 |
   | `more` | 分页查看，只能向下翻 | 较大文件（已过时，推荐 less） |
   | `less` | 分页查看，可上下翻、搜索 | 大文件（推荐） |
   | `head` | 显示前 N 行 | 快速预览 |
   | `tail` | 显示后 N 行，支持 `-f` 实时跟踪 | 日志监控 |

5. **冷知识**：`cat` 是 "concatenate"（连接）的缩写，不只是"查看文件"。

**常见误解**：初学者常用 `cat` 查看大文件然后终端被淹没，应该养成用 `less` 的习惯。

---

## 第二组：进程管理、服务控制、文件查找与文本搜索

> [!info] 这组命令是服务器运维的核心
> 管进程、管服务、找文件、搜内容。

### more / less -- 分页查看

**more 基础知识：**
- `more file.txt`：空格翻页，Q 退出，只能向下

**重要局限**：`more` 只能**向前**翻页，不能向后翻。

> [!warning] 过时命令
> `more` 已过时，`less` 是大多数发行版的默认分页器（包括 `man` 命令的默认查看器）。`less` 的名字是个文字游戏："less is more"。

**less 进阶技巧（推荐使用）：**
```bash
less -S file.txt         # 长行不自动换行（左右滚动）
less +F file.txt         # 类似 tail -f，实时跟踪文件末尾（按 Ctrl+C 退出跟踪模式，进入less交互）
less -N file.txt         # 显示行号
/pattern                 # 在 less 中搜索（n 下一个，N 上一个）
?pattern                 # 向上搜索
b                        # 向后翻页
Space / f                # 向前翻页
```

**less vs more 对比**：
| 特性 | more | less |
|------|------|------|
| 向前翻页 | 支持 | 支持 |
| 向后翻页 | 不支持 | 支持 |
| 搜索 | 不支持 | 支持 |
| 内存效率 | 一次性读入 | 按需读取 |
| 现代推荐 | 已过时 | 推荐 |

---

### ps -- 查看进程

**基础知识：**
- `ps aux`：查看所有进程（BSD 风格，不带 `-`）
- `ps -ef`：查看所有进程（UNIX/POSIX 风格，带 `-`）
- `ps aux | grep nginx`：查找特定进程

**ps aux 和 ps -ef 的关键区别**：

| 特性 | ps -ef | ps aux |
|------|--------|--------|
| 语法风格 | UNIX/POSIX | BSD |
| PPID（父进程ID） | 有 | 无 |
| %CPU, %MEM | 无 | 有 |
| VSZ, RSS（内存） | 无 | 有 |
| C（CPU利用率） | 有 | 无 |

**常见误解**：`ps aux` 中的 `-` 是**不能加**的，`ps -aux` 和 `ps aux` 是不同的。`ps -aux` 意味着按用户 x 查询，可能产生混淆或警告。

**进阶技巧：**

1. **进程树**：
   ```bash
   ps -ef --forest        # 以树状结构显示进程关系
   pstree                 # 更直观的进程树
   ```

2. **按资源排序**：
   ```bash
   ps aux --sort=-%mem | head -10    # 按内存占用排序 TOP10
   ps aux --sort=-%cpu | head -10    # 按 CPU 占用排序 TOP10
   ```

3. **精确查找进程（避免 grep 自身）**：
   ```bash
   ps aux | grep '[n]ginx'    # 用正则技巧排除 grep 自身
   pgrep -a nginx             # 直接查找，更简洁
   ```

4. **查看特定进程的线程**：
   ```bash
   ps -T -p <pid>         # 查看某进程的所有线程
   ps -Lf <pid>           # 更详细的线程信息
   ```

---

### kill -- 终止进程

**基础知识：**
- `kill <pid>`：发送 **SIGTERM（信号 15）**，请求进程优雅终止
- `kill -9 <pid>`：发送 **SIGKILL（信号 9）**，强制终止

> [!tip] 比喻：SIGTERM vs SIGKILL
> SIGTERM（`kill PID`）= 礼貌地敲门说"请离开"，对方可以收拾东西、保存文件后再走，也可以选择不理你。
> SIGKILL（`kill -9`）= 保安直接把人抬走，不给任何收拾的时间，东西可能丢。

**关键区别**：

| 特性 | kill (SIGTERM/15) | kill -9 (SIGKILL/9) |
|------|-------------------|---------------------|
| 进程能否捕获 | 能，可执行清理操作 | 不能，立即杀死 |
| 进程能否忽略 | 能 | 不能 |
| 允许清理 | 是 | 否 |
| 是否安全 | 安全，允许资源释放 | 危险，可能丢失数据 |
| 推荐用途 | 首选尝试 | 最后手段 |

**生产环境红线**：优先用 `kill -15`（SIGTERM）优雅停机，严禁上来就 `kill -9`，仅当服务完全卡死无法响应时才使用。

**最佳实践**：先 `kill PID`，等待几秒，若进程仍存在再 `kill -9 PID`。

**相关信号**：
```bash
kill -1 <pid>    # SIGHUP，重新加载配置
kill -2 <pid>    # SIGINT，等同 Ctrl+C
kill -15 <pid>   # SIGTERM，优雅终止（默认）
kill -9 <pid>    # SIGKILL，强制终止
kill -USR1 <pid> # 用户自定义信号1（如让 nginx 重新打开日志）
kill -STOP <pid> # 暂停进程
kill -CONT <pid> # 恢复暂停的进程
```

**批量终止**：
```bash
killall nginx                    # 按进程名终止
pkill -f "python app.py"         # 按完整命令行匹配
pkill -9 -t pts/0                # 强制踢出某个终端用户
```

**冷知识**：`kill -9` 的 9 是 POSIX 标准定义的信号编号。SIGKILL 是 31 个标准信号中唯一一个进程无法捕获或忽略的信号，是操作系统级别的"最终手段"。

---

### systemctl / service -- 服务管理

**基础知识：**
- `service` 是旧式 **SysVinit** 的服务管理命令
- `systemctl` 是 **systemd** 的服务管理命令（现代 Linux 主流）

**时代划分**：systemd 于 2010 年引入，2014 年 RHEL/CentOS 7 采用，2015 年 Debian 8 和 Ubuntu 15.04 采用。

**关键补充**：在 systemd 系统上，`service` 命令仍然存在，是一个**兼容性包装器**，会将命令转发给 `systemctl`。

**发行版差异**：
- Debian 8+、Ubuntu 15.04+、RHEL/CentOS 7+、Fedora 15+、Arch Linux 使用 systemd
- Devuan、AntiX 等少数发行版刻意不使用 systemd
- Alpine Linux 使用 OpenRC（不是 systemd 也不是 SysVinit）

**常用命令**：
```bash
systemctl start nginx        # 启动
systemctl stop nginx         # 停止
systemctl restart nginx      # 重启
systemctl reload nginx       # 重新加载配置（不中断服务）
systemctl status nginx       # 查看状态（显示日志片段、进程树、最近状态变化）
systemctl enable nginx       # 设置开机启动
systemctl disable nginx      # 取消开机启动
systemctl is-active nginx    # 检查是否运行中
systemctl list-units --type=service --state=failed  # 查看失败的服务
journalctl -u nginx -f       # 实时跟踪服务日志
journalctl -u nginx --since "1 hour ago"  # 查看最近1小时日志
```

---

### find -- 文件查找

**基础知识：**
- `find /path -name "*.log"`：按名称查找
- `find /path -type f -mtime +7`：查找 7 天前修改的文件

**重要提醒**：引号很重要！`find . -name *.txt`（不加引号）会在 shell 层面先展开通配符，导致行为不符预期。必须写成 `find . -name "*.txt"`。

**进阶技巧：**

1. **组合条件查找**：
   ```bash
   find /var/log -name "*.log" -size +100M          # 大于 100M 的日志
   find /home -name "*.jpg" -size +1M -mtime -30    # 30天内修改的大于1M的jpg
   find /etc -type f -name "*.conf" -perm 644       # 权限为644的conf文件
   find . -iname "*.TXT"                            # 不区分大小写搜索
   find . -type d -name "src"                       # 只搜索目录
   find . -type f -name "*.log"                     # 只搜索文件
   ```

2. **安全审计**：
   ```bash
   find / -perm -4000 -type f 2>/dev/null           # 查找 SUID 文件
   find / -perm -2000 -type f 2>/dev/null           # 查找 SGID 文件
   find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \)  # 找无粘滞位的全局可写目录
   ```

3. **批量操作**：
   ```bash
   find /var/log -name "*.log" -mtime +30 -exec gzip {} \;     # 压缩30天前的日志
   find . -name "*.tmp" -exec rm -f {} +                       # 删除临时文件（+ 比 \; 快）
   find . -type f -exec chmod 644 {} \;                        # 批量改权限
   find . -type d -exec chmod 755 {} \;                        # 批量改目录权限
   ```

4. **"Argument list too long" 错误解决方案**：
   ```bash
   # 错误：rm *.log（文件太多会报错）
   # 正确：
   find . -name "*.log" -exec rm {} +
   ```

**安全性**：`find / -name "*.txt"` 会搜索整个文件系统，包括 `/proc`、`/sys` 等虚拟文件系统，速度很慢。建议加排除规则或限制搜索范围。

**最佳实践**：删除前先用 `-print` 或替换为 `ls -l` 预览结果：
```bash
find /var/log -name "*.log" -mtime +30 -exec ls -l {} \;  # 先预览
find /var/log -name "*.log" -mtime +30 -exec rm {} +      # 再执行
```

---

### grep -- 文本搜索

**基础知识：**
- `grep "pattern" file`：在文件中搜索
- `grep -r "pattern" /path`：递归搜索目录
- `grep -i "pattern" file`：忽略大小写
- `grep -v "pattern" file`：反向匹配（排除）
- `grep -n "pattern" file`：显示行号
- `grep -c "pattern" file`：只显示匹配行数
- `grep -l "pattern" *.txt`：只显示包含匹配的文件名

**进阶技巧：**

1. **上下文显示**：
   ```bash
   grep -A 3 "error" file.log    # 显示匹配行及后3行（After）
   grep -B 3 "error" file.log    # 显示匹配行及前3行（Before）
   grep -C 3 "error" file.log    # 显示匹配行前后各3行（Context）
   ```

2. **正则表达式**：
   ```bash
   grep -E "error|warning" file.log           # 匹配 error 或 warning（扩展正则）
   grep -P '\d{3}-\d{4}' contacts.txt         # Perl 正则，匹配电话号码
   grep -n --color=auto 'error' file.log      # 带行号高亮显示
   ```

3. **压缩文件搜索**：
   ```bash
   zgrep "error" *.gz              # 直接搜索 .gz 压缩文件
   zgrep "exception" app.log.*.gz  # 搜索多个压缩文件
   ```

4. **日志分析组合技**：
   ```bash
   grep "500" access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
   # 统计返回 500 错误的 IP 访问量 TOP 10
   ```

5. **冷知识**：`grep` 的名字来自 ed 编辑器的命令 `g/re/p`（global/regular expression/print）。

**常见误解**：`ps -ef | grep python` 本身也会匹配到 `grep python` 这个进程，通常需要 `ps -ef | grep '[p]ython'` 来排除。更好的做法是用 `pgrep python`。

---

### pwd / history / echo

**pwd：**
```bash
pwd          # 显示当前目录（逻辑路径）
pwd -P       # 显示物理路径（解析符号链接后的真实路径）
```

**history：**
```bash
history              # 查看命令历史
history | grep "tar" # 搜索历史
!123                 # 执行第 123 条历史命令
!!                   # 重复上一条命令
!$                   # 上一条命令的最后一个参数
sudo !!              # 以 sudo 重新执行上一条命令
!string              # 执行最近以 string 开头的命令
history -c           # 清空历史（仅当前会话）
history -w           # 将当前历史写入 ~/.bash_history 文件
```

**安全性**：`history` 会记录所有命令，包括含有密码的命令。敏感操作应使用 `export HISTFILE=/dev/null` 或在命令前加空格（如果配置了 `HISTCONTROL=ignorespace`）来避免记录。

**冷知识**：`!!` 配合 sudo 非常实用。忘记加 sudo 时，输入 `sudo !!` 即可以 root 权限重试。

**echo：**
```bash
echo $PATH           # 查看 PATH 环境变量
echo "Hello" > file  # 覆盖写入
echo "World" >> file # 追加写入
echo -e "line1\nline2"  # 解释转义字符
```

---

### chmod -- 修改权限

**基础知识：**
```bash
chmod 755 script.sh     # rwxr-xr-x
chmod +x script.sh      # 添加执行权限
chmod -R 755 /var/www   # 递归修改
```

> [!tip] 比喻：文件权限 rwx 和数字表示
> 文件权限就像房子的钥匙权限：
> - **r（读）= 能看窗户** -- 可以查看文件内容
> - **w（写）= 能搬家具** -- 可以修改文件内容
> - **x（执行）= 能开门进去** -- 可以运行文件
>
> 对于目录：
> - **r** = 能看窗户（列出目录内容 ls）
> - **w** = 能搬家具（创建、删除、重命名文件）
> - **x** = 能开门进去（进入目录 cd）
>
> 目录没有 `x` 权限时，即使有 `r` 权限也无法 cd 进入。

> [!tip] 比喻：4-2-1 编码
> 权限数字就像一个**三位密码锁**，每位只能选 4、2、1 或 0：
> - **4** = 读（r）
> - **2** = 写（w）
> - **1** = 执行（x）
> - **0** = 无权限
>
> 把选中的数字加起来就是该位的值：
> - `rwx` = 4+2+1 = **7**
> - `r-x` = 4+0+1 = **5**
> - `rw-` = 4+2+0 = **6**
> - `r--` = 4+0+0 = **4**
>
> 三位数字分别对应：所有者、所属组、其他人。例如 `755` = `rwxr-xr-x`。

**用户权限数字表示（4-2-1 编码）：**

| 数字 | 二进制 | 权限 | 含义 |
|------|--------|------|------|
| 0 | 000 | --- | 无权限 |
| 1 | 001 | --x | 执行 |
| 2 | 010 | -w- | 写 |
| 3 | 011 | -wx | 写+执行 |
| 4 | 100 | r-- | 读 |
| 5 | 101 | r-x | 读+执行 |
| 6 | 110 | rw- | 读+写 |
| 7 | 111 | rwx | 读+写+执行 |

**常用权限组合**：
- `chmod 644` -- 所有者 rw，组 r，其他 r（普通文件的标准权限）
- `chmod 755` -- 所有者 rwx，组 r-x，其他 r-x（可执行文件/目录的标准权限）
- `chmod 600` -- 只有所有者可读写（私密文件）

**特殊权限（s/t）：**

> [!tip] 比喻：SUID/SGID
> SUID 就像**借了别人的 VIP 卡进会员区**。当你运行一个设置了 SUID 的程序时，你暂时"借用"了文件所有者的权限。比如 `/usr/bin/passwd` 设置了 SUID，所以普通用户运行它时能临时获得 root 权限来修改 `/etc/shadow`。

> [!tip] 比喻：粘滞位 (sticky bit)
> 粘滞位就像**公共储物柜**。`/tmp` 目录所有人都能放东西进去（权限 777），但有了粘滞位，你只能动自己的东西，不能拿别人的。

| 权限 | 名称 | 数字 | 作用位置 | 含义 |
|------|------|------|----------|------|
| SUID | Set User ID | 4 | 所有者的 x 位 | 执行时以文件所有者身份运行 |
| SGID | Set Group ID | 2 | 所属组的 x 位 | 执行时以文件所属组身份运行；目录下新建文件继承目录属组 |
| Sticky Bit | 粘滞位 | 1 | 其他人的 x 位 | 目录下文件只有所有者和 root 能删除 |

**设置方式：**
```bash
chmod u+s /usr/bin/passwd   # 设置 SUID（符号方式）
chmod 4755 file             # 设置 SUID（数字方式，4 = SUID）
chmod g+s /shared/dir       # 设置 SGID
chmod 2755 file             # 设置 SGID（数字方式）
chmod o+t /tmp              # 设置 Sticky Bit
chmod 1777 /tmp             # 设置 Sticky Bit（数字方式）
```

**常见坑：**
- `chmod 777` 让所有人拥有完全控制权限，是严重的安全隐患。生产环境严禁使用。
- `chmod -R 777 /` 会破坏整个系统。
- 当特殊权限位显示为大写 `S` 或 `T`（而非小写 `s`/`t`），表示对应的 `x` 权限缺失，特殊权限实际上无效（通常是配置错误）。

**冷知识**：`/tmp` 的权限是 `drwxrwxrwt`，最后那个 `t` 就是粘滞位。

---

### chown -- 修改文件所有者

```bash
chown user:group file.txt       # 修改所有者和属组
chown -R www:www /var/www/html  # 递归修改
chown :developers project/      # 只修改属组（等同 chgrp）
chown user file                 # 只改变所有者
chown :group file               # 只改变组
chown user: file                # 改变所有者，并将组设为该用户的登录组
```

**分隔符说明**：`:` 是推荐的分隔符，`.` 是已废弃的旧语法（因为文件名可能包含 `.`，导致歧义）。

---

### sudo -- 以 root 权限执行

```bash
sudo command              # 以 root 执行单条命令
sudo -u username command  # 以指定用户执行
sudo -i                   # 切换到 root 的登录 shell
sudo visudo               # 安全编辑 sudoers 文件
```

**安全实践**：
- 生产环境 Java 服务，严禁用 root 用户运行，必须用普通用户运行。
- 使用 `visudo -f /etc/sudoers.d/developers` 分文件管理权限，而非直接编辑 `/etc/sudoers`。

---

### ping / ifconfig

**ping：**
```bash
ping -c 4 google.com     # 只发送 4 个包（默认会一直 ping）
ping -i 2 host           # 每 2 秒发一次
ping -W 5 host           # 5 秒超时
ping -s 1024 host        # 指定包大小
```

**发行版差异**：某些最小化安装可能不包含 `ping`，需要安装 `iputils-ping`（Debian/Ubuntu）或 `iputils`（RHEL/CentOS）。

> [!warning] 过时命令
> `ifconfig` 自 2010 年代起已被标记为废弃。现代 Linux 推荐使用 `ip` 命令（来自 `iproute2` 包）。很多现代发行版默认不安装 `net-tools`。

| 旧命令 | 新命令 | 功能 |
|--------|--------|------|
| `ifconfig` | `ip addr show` / `ip a` | 查看网络接口 |
| `ifconfig eth0 up` | `ip link set eth0 up` | 启用接口 |
| `route -n` | `ip route show` | 查看路由表 |
| `netstat -tlnp` | `ss -tlnp` | 查看监听端口 |

```bash
# 旧式命令（已废弃）
ifconfig                 # 查看网络接口信息

# 新式命令（推荐）
ip addr show             # 查看网络接口
ip route show            # 查看路由表
```

---

### ssh -- 远程登录

```bash
ssh user@host                    # 基本连接
ssh -p 22022 user@host           # 指定端口
ssh -i ~/.ssh/id_rsa user@host   # 使用指定密钥
ssh user@host 'command'          # 远程执行命令
ssh -L localport:remotehost:remoteport user@host  # 本地端口转发
```

**免密登录配置：**
```bash
ssh-keygen -t rsa -b 4096        # 生成密钥对
ssh-copy-id user@host            # 将公钥复制到远程主机
```

**SSH 配置文件**：`~/.ssh/config` 可以配置连接别名和默认参数，简化常用连接。

**安全性**：
- 应禁用 root 直接 SSH 登录（`PermitRootLogin no`）
- 应使用密钥认证而非密码认证
- 建议使用非标准端口、fail2ban 等防护措施

---

### shutdown / reboot / poweroff

```bash
shutdown -h now       # 立即关机
shutdown -r now       # 立即重启
shutdown -h +10       # 10 分钟后关机
shutdown -h 22:00     # 在 22:00 关机
shutdown -c           # 取消计划中的关机
reboot                # 重启
poweroff              # 关机
```

**说明**：在 systemd 系统上，`reboot` 和 `poweroff` 实际上是 `systemctl reboot` 和 `systemctl poweroff` 的封装。`shutdown` 支持定时和向登录用户发送警告消息。这些命令通常需要 root/sudo 权限。

---

### yum / apt -- 包管理

**发行版差异：**
- CentOS/RHEL 7：`yum`
- CentOS/RHEL 8+：`dnf`（yum 的下一代替代品）
- Ubuntu/Debian：`apt`

> [!warning] 过时命令
> `yum` 在 RHEL/CentOS 8+ 已被 `dnf` 取代。

```bash
# yum (CentOS 7) / dnf (CentOS 8+)
yum install nginx         # 安装
yum install -y nginx      # 自动确认（脚本中常用）
yum remove nginx          # 卸载
yum update                # 更新所有包
yum search keyword        # 搜索
yum info nginx            # 查看包信息
yum list installed        # 列出已安装的包
yum autoremove            # 移除不再需要的依赖

# apt (Ubuntu/Debian)
apt install nginx         # 安装
apt remove nginx          # 卸载（保留配置）
apt purge nginx           # 卸载包并删除配置
apt update                # 刷新包索引（不安装或升级任何东西）
apt upgrade               # 安装更新
apt update && apt upgrade # 更新（配合使用）
apt full-upgrade          # 升级并允许移除冲突的包
apt search keyword        # 搜索
apt show nginx            # 查看包信息
apt autoremove            # 移除不再需要的依赖
```

**常见误解**：`apt update` 不会更新任何已安装的软件，它只更新本地的包索引。初学者经常混淆 `update` 和 `upgrade`。

---

### man -- 查看手册

```bash
man ls           # 查看 ls 的手册
man -k keyword   # 搜索手册页（等同于 apropos keyword）
man 5 passwd     # 查看 passwd 文件格式的手册（第5节）
info ls          # 更详细的信息（GNU info 文档）
tldr ls          # 简化版帮助（需安装 tldr）
command --help   # 快速查看帮助（比 man 简洁）
```

**冷知识**：man 手册分 8 个章节：1-用户命令、2-系统调用、3-库函数、4-特殊文件、5-文件格式、6-游戏、7-杂项、8-系统管理命令。`man 1 printf` 和 `man 3 printf` 显示的内容不同。

---

### tail / head -- 查看文件首尾

**基础知识：**
```bash
head -20 file.txt    # 显示前 20 行
tail -20 file.txt    # 显示后 20 行
```

**进阶技巧：**
```bash
tail -f /var/log/syslog              # 实时跟踪文件末尾（日志监控神器）
tail -F /var/log/syslog              # 实时跟踪文件名（适合日志轮转场景）
tail -f app.log | grep --line-buffered "error"  # 实时跟踪并过滤
tail -n +5 file.txt                  # 从第 5 行开始显示到末尾
tail -n 100 -f logfile               # 先显示最后100行，然后实时跟踪
head -c 100 file.txt                 # 显示前 100 个字节
head -n -10 file.txt                 # 显示除最后 10 行以外的所有内容（GNU 扩展）
```

**tail -f vs tail -F**：

| 选项 | 行为 | 适用场景 |
|------|------|----------|
| `tail -f` | 跟踪文件末尾，文件被轮转（logrotate）时会停止跟踪 | 普通日志查看 |
| `tail -F`（`--follow=name --retry`） | 持续跟踪文件名，文件被轮转后自动重新打开 | 日志监控（推荐） |

---

## 第三组：压缩、用户、磁盘、删除

> [!info] 这组覆盖了系统管理的日常操作
> 打包备份、用户管理、磁盘空间、文件删除。

### tar -- 打包压缩

> [!tip] 比喻：tar 打包压缩
> **打包**就像把衣服装进行李箱——把很多文件合成一个文件，方便搬运。
> **压缩**就像用真空袋抽掉空气——减小文件体积，节省空间。
> 所以 `tar -czf` = 先装箱（打包），再抽真空（压缩）。

**基础知识（记忆口诀：打包压缩 czf，解压 xzf）：**

```bash
tar -czvf archive.tar.gz dir/      # 压缩（gzip 格式）
tar -xzvf archive.tar.gz           # 解压
tar -xzvf archive.tar.gz -C /dest  # 解压到指定目录
tar -tzvf archive.tar.gz           # 查看压缩包内容（不解压）
```

**核心参数：**
| 参数 | 含义 |
|------|------|
| `-c` | 创建归档（create） |
| `-x` | 解压归档（extract） |
| `-z` | gzip 压缩/解压（.tar.gz） |
| `-j` | bzip2 压缩/解压（.tar.bz2） |
| `-J` | xz 压缩/解压（.tar.xz） |
| `-f` | 指定文件名（**必须在参数最后**） |
| `-v` | 显示过程 |
| `-C` | 指定解压目录 |

**进阶技巧：**

1. **排除特定目录/文件**：
   ```bash
   tar -czvf backup.tar.gz /data --exclude=/data/logs --exclude=/data/tmp
   ```

2. **压缩算法选择**：

   | 算法 | 参数 | 速度 | 压缩率 | 适用场景 |
   |------|------|------|--------|----------|
   | gzip | `-z` | 快 | 一般 | 推荐日常使用 |
   | bzip2 | `-j` | 慢 | 高 | 需要更高压缩率时 |
   | xz | `-J` | 最慢 | 极高 | 大文件归档 |

3. **分卷压缩**（大文件传输）：
   ```bash
   tar -czvf - /data | split -b 100M - backup.tar.gz.part-
   # 合并解压：
   cat backup.tar.gz.part-* | tar -xzvf -
   ```

4. **多线程压缩**（使用 pigz 替代 gzip）：
   ```bash
   tar -cvf - /bigdata | pigz -p 8 > backup.tar.gz    # 8线程并行压缩
   ```

5. **安全解压实践**：解压前先用 `tar tzvf` 查看内容，避免覆盖现有文件。

**常见误解**：`-f` 必须是最后一个选项（因为它后面紧跟文件名），这是一个常见错误源。`-v` 在解压大文件时会产生海量输出，脚本中通常省略。

**冷知识**：`tar` 命令名字来自 "tape archive"（磁带归档），因为最初设计用于将文件备份到磁带上。

---

### useradd / passwd -- 用户管理

**useradd：**
```bash
useradd -m -s /bin/bash newuser     # 创建用户，创建家目录，指定shell
useradd -d /home/custom -m user     # 指定家目录
useradd -g groupname -G g1,g2 user  # 指定初始组和附加组
userdel -r username                  # 删除用户及其家目录
```

**重要说明**：仅 `useradd tom` 不会创建家目录，默认 shell 也可能是 `/bin/sh`。建议始终使用 `useradd -m -s /bin/bash tom`。

**发行版差异**：

| 特性 | useradd | adduser（Debian/Ubuntu） |
|------|---------|--------------------------|
| 类型 | 底层二进制 | Perl 脚本（高级封装） |
| 交互式 | 否 | 是 |
| 自动创建家目录 | 需要 `-m` | 默认创建 |
| 跨发行版 | 是 | 主要 Debian 系 |

在 RHEL/CentOS 系统上，`adduser` 是 `useradd` 的符号链接，行为相同。

**passwd：**
```bash
passwd username         # 修改用户密码
passwd -l username      # 锁定用户
passwd -u username      # 解锁用户
```

**说明**：root 用户修改其他用户密码时**不需要知道旧密码**。普通用户修改自己的密码时需要先输入旧密码。密码存储在 `/etc/shadow` 中（加密存储）。

**安全性**：`passwd` 二进制文件设置了 SUID 位，使其能够修改 `/etc/shadow`。这是一个典型的 SUID 使用场景。

---

### du / df -- 磁盘管理

**du（disk usage）-- 看具体目录/文件：**
```bash
du -sh /var/log              # 查看目录总大小
du -sh *                     # 查看当前目录各文件/目录大小
du -h --max-depth=1 /var     # 只显示第一级子目录大小
du -sh * | sort -rh | head   # 按大小排序
```

**df（disk free）-- 看文件系统整体：**
```bash
df -h                        # 人性化显示各分区使用情况
df -hT                       # 显示文件系统类型
df -ih                       # 查看 inode 使用情况（inode 耗尽也会导致无法写入）
```

**区别**：`df` 看的是文件系统级别的磁盘使用情况，`du` 看的是具体目录/文件级别的磁盘占用。两者结果可能不同（因为已删除但未释放的文件、文件系统元数据等）。

**磁盘满排查组合技**：
```bash
df -h                                    # 先看哪个分区满了
du -h / | sort -rh | head -10            # 找根目录下最大的目录
find / -type f -size +100M 2>/dev/null | xargs du -sh | sort -hr  # 找大文件
lsof +L1                                 # 找被删除但未释放的文件（进程还持有句柄）
```

---

### rm -- 删除文件/目录

**基础知识：**
```bash
rm file.txt             # 删除文件
rm -r directory/        # 递归删除目录
rm -f file.txt          # 强制删除（不询问）
rm -rf directory/       # 强制递归删除（最危险）
```

> [!danger] 安全警告
> `rm -rf` 是**不可逆**的操作。Linux 没有回收站（命令行下），文件直接删除。数据恢复非常困难（需要专业工具且不保证成功）。

**生产环境严禁操作**：
```bash
rm -rf /              # 绝对禁止！会删除整个系统（现代系统有 --preserve-root 保护）
rm -rf *              # 极度危险！先用 ls 确认路径
rm -rf ~/*            # 删除家目录所有文件
```

**安全删除习惯**：
```bash
# 删除前先用 ls 确认
ls /var/log/old_logs/
rm -rf /var/log/old_logs/

# 使用 -i 参数交互确认
rm -ri directory/

# 优先用 find -delete 更安全
find /tmp -name "*.tmp" -mtime +7 -delete
```

**封印 rm 命令**（最佳实践）：
```bash
# 在 ~/.bashrc 中添加
alias rm='rm -i'    # 每次删除前确认
```

**删除大量文件**（rm 慢或报错时）：
```bash
# 方法1：rsync 空目录覆盖（比 rm -rf 更快）
mkdir /tmp/empty
rsync -a --delete /tmp/empty/ /target/dir/

# 方法2：find + delete
find /target/dir -type f -delete
```

**建议**：使用 `trash-put`（来自 `trash-cli` 包）替代 `rm`，提供回收站功能。

---

## 第四组：传输与网络请求

> [!info] 文件传输和网络调试
> 远程运维的必备技能。

### scp -- 远程文件复制

**基础知识：**
```bash
scp file.txt user@host:/remote/path/       # 本地传到远程
scp user@host:/remote/file.txt ./           # 远程传到本地
scp -r directory/ user@host:/remote/path/   # 递归复制目录
```

**进阶技巧：**
```bash
scp -P 22022 file.txt user@host:/path/     # 指定端口（大写 P，与 ssh -p 不同）
scp -i ~/.ssh/id_rsa file.txt user@host:/  # 指定密钥
scp -C file.txt user@host:/path/           # 启用压缩传输
scp -l 1000 file.txt user@host:/path/      # 限速（Kbit/s）
```

> [!warning] 过时命令
> OpenSSH 9.0（2022 年 4 月）起，`scp` 协议被标记为废弃，推荐使用 `sftp` 或 `rsync` 替代。`scp` 基于 RCP 协议，存在已知安全漏洞（CVE-2020-15778）。OpenSSH 9.0+ 的 `scp` 默认使用 SFTP 协议传输。

**替代方案**：
```bash
# sftp -- OpenSSH 官方推荐的文件传输方式
sftp user@host

# rsync -- 增量传输，支持断点续传、压缩、带宽限制
rsync -avz -e ssh src/ user@host:/dest/
rsync -avz --progress /local/dir/ user@host:/remote/dir/
```

**rsync vs scp**：

| 特性 | rsync | scp |
|------|-------|-----|
| 传输方式 | 增量传输（只传变化部分） | 每次全量传输 |
| 适用场景 | 大量文件和重复同步 | 一次性复制 |
| 镜像同步 | 支持 `--delete` | 不支持 |
| 断点续传 | 支持 | 不支持 |

---

### curl -- 网络请求工具

**基础知识：**
```bash
curl https://example.com                 # GET 请求（输出到终端）
curl -O https://example.com/file.zip     # 下载文件（保留原文件名）
curl -o save.zip https://example.com/file.zip  # 下载并指定文件名
```

**进阶技巧：**

1. **HTTP 请求方法**：
   ```bash
   curl -X GET https://api.example.com/users
   curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com
   curl -X POST -d "key=value" URL                    # 发送 POST 请求带表单数据
   curl -X PUT -d '{"name":"new"}' https://api.example.com/users/1
   curl -X DELETE https://api.example.com/users/1
   ```

2. **调试和诊断**：
   ```bash
   curl -I https://example.com           # 只显示 HTTP 响应头（发送 HEAD 请求）
   curl -i https://example.com           # 显示响应头 + 响应正文
   curl -v https://example.com           # 显示详细请求/响应信息
   curl -L https://example.com           # 跟随重定向（很多场景必须加）
   curl -k https://self-signed.cert/     # 忽略 SSL 证书验证（不推荐在生产环境使用）
   ```

3. **实用场景**：
   ```bash
   curl ifconfig.me                      # 查看本机公网 IP
   curl -sS https://api.example.com | jq .   # 配合 jq 格式化 JSON
   ```

**常见误解**：
- `curl -I`（大写 i）发送 HEAD 请求，只获取响应头，不下载正文。
- `curl -i`（小写 i）获取响应头 + 响应正文。
- `curl -X POST` 只是指定了方法，还需要配合 `-d` 或 `--data` 发送数据。

---

## 补充知识

> [!info] 补充知识
> 基础概念的深入解释，帮助理解权限系统和文件系统。

### 权限位 rwx 详解

对于**文件**：
- `r`（读）：可查看文件内容（cat、less、more）
- `w`（写）：可修改文件内容
- `x`（执行）：可将文件作为程序运行

对于**目录**（含义不同）：
- `r`（读）：可列出目录内容（ls）
- `w`（写）：可在目录中创建、删除、重命名文件
- `x`（执行）：可进入目录（cd），是最基本的目录权限

**关键点**：目录没有 `x` 权限时，即使有 `r` 权限也无法 cd 进入。

---

### /tmp 权限 drwxrwxrwt

- `d` -- 目录
- `rwxrwxrwx` -- 所有用户可读、可写、可进入
- `t` -- Sticky Bit，只有文件所有者、目录所有者或 root 可以删除/重命名目录中的文件

**安全性**：没有 sticky bit 的话，任何用户都可以删除 `/tmp` 中其他用户的文件。POSIX 标准要求 `/tmp` 和 `/var/tmp` 设置 sticky bit。

---

### 软链接 vs 硬链接

> [!tip] 比喻：软链接 vs 硬链接
> **软链接（符号链接）= 快捷方式/指路牌**：它指向另一个文件的路径，就像 Windows 的快捷方式。如果原文件被删除，软链接就变成"断链"。
>
> **硬链接 = 同一栋房子有两个门牌号**：两个门牌号指向同一个房子（inode）。删除一个门牌号，房子还在。只有所有门牌号都被删除，房子才会被拆除。

```bash
# 创建软链接
ln -s target link_name

# 创建硬链接
ln target link_name
```

---

### 进程 PID

> [!tip] 比喻：进程 PID
> PID 就像每个人的**身份证号**。每个进程在系统中都有一个唯一的 PID，通过这个号码可以找到并管理它（查看状态、发送信号、终止等）。

---

### 存储单位

| 单位 | 缩写 | 换算 |
|------|------|------|
| Byte（字节） | B | 基本单位 |
| Kilobyte | KB / KiB | 1 KB = 1000 B / 1 KiB = 1024 B |
| Megabyte | MB / MiB | 1 MB = 1000 KB / 1 MiB = 1024 KiB |
| Gigabyte | GB / GiB | 1 GB = 1000 MB / 1 GiB = 1024 MiB |
| Terabyte | TB / TiB | 1 TB = 1000 GB / 1 TiB = 1024 GiB |
| Petabyte | PB / PiB | 1 PB = 1000 TB / 1 PiB = 1024 TiB |

**说明**：
- **二进制（1024 进制）**：使用 KiB、MiB、GiB 等单位（IEC 标准）
- **十进制（1000 进制）**：使用 KB、MB、GB 等单位
- 硬盘制造商使用十进制，所以标称 1 TB 的硬盘在系统中显示约 931 GiB
- Linux 工具的 `--si` 选项使用 1000 进制（如 `df -h` vs `df -H`）

---

## 第五组：拓展命令

> [!info] 以下命令是大纲之外的实用补充
> 掌握它们能让你的 Linux 技能更上一层楼。

### alias -- 命令别名

```bash
alias ll='ls -alF'                  # 临时别名
alias grep='grep --color=auto'      # 让 grep 默认高亮
alias ..='cd ..'
alias myip='curl ifconfig.me'
# 在 ~/.bashrc 中添加使其永久生效
unalias ll                          # 删除别名
alias                               # 查看所有别名
```

**安全实践**：`alias rm='rm -i'` 可以防止误删。

---

### xargs -- 参数传递

```bash
# 基本用法
find . -name "*.txt" | xargs rm                    # 删除所有 txt 文件
echo "a,b,c" | xargs -d , echo                     # 自定义分隔符
echo "a b c" | xargs -n 1 echo                     # 每次传递一个参数

# 占位符 -I{}
find . -name "*.jpg" | xargs -I{} cp {} /backup/   # {} 是占位符
ls | xargs -I{} mv {} {}.bak                       # 批量重命名

# 并行执行
echo "1 2 3 4" | xargs -n 1 -P 4 gzip             # 4 个并行进程

# 处理文件名含空格的文件
find . -name "*.txt" -print0 | xargs -0 rm         # -print0 和 -0 配合使用
```

**冷知识**：管道 `|` 不能直接给 `ls`、`cp`、`rm`、`mv` 传递参数，需要通过 `xargs` 中转。

---

### awk -- 文本处理

```bash
# 基本结构：awk 'pattern {action}' file
awk '{print $1}' file.txt                  # 打印第一列
awk -F: '{print $1, $3}' /etc/passwd       # 指定分隔符，打印用户名和UID
awk '{sum+=$3} END {print sum}' data.txt   # 计算第三列总和
awk 'NR>=10 && NR<=20' file.txt            # 打印第10-20行

# 日志分析
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
# 统计访问 IP TOP10

awk '{url[$7]++} END {for (k in url) {print url[k], k}}' access.log | sort -rn
# 统计 URL 访问量排行
```

---

### sed -- 流编辑器

```bash
# 替换
sed 's/old/new/' file.txt            # 替换每行第一个匹配
sed 's/old/new/g' file.txt           # 替换所有匹配（全局）
sed -i 's/old/new/g' file.txt        # 原地修改文件（危险，建议先备份）
sed -i.bak 's/old/new/g' file.txt    # 修改前自动创建 .bak 备份

# 删除
sed '/^$/d' file.txt                 # 删除空行
sed '1,5d' file.txt                  # 删除前5行

# 批量修改配置文件
grep -rnl --include="*.conf" "old_value" | xargs -i sed -i.bak 's/old_value/new_value/g' {}
```

---

### wc -- 统计

```bash
wc -l file.txt    # 统计行数
wc -w file.txt    # 统计单词数
wc -c file.txt    # 统计字节数
wc file.txt       # 行数、单词数、字节数全部显示
```

---

### free -- 内存查看

```bash
free -h              # 人性化显示
free -m              # 以 MB 为单位
free -g              # 以 GB 为单位
free -s 5            # 每 5 秒刷新一次
```

**解读**：关注 `available` 列而非 `free` 列。`available` 包含了可回收的缓存，代表实际可用内存。

---

### top / htop -- 系统监控

**top：**
```bash
top                  # 进入交互式界面
top -b -n 1          # 非交互模式，输出一次（适合脚本使用）
# 交互模式中：按 M 按内存排序，按 P 按 CPU 排序，按 k 杀进程，按 q 退出
```

**htop**（top 的增强版，需安装）：
```bash
htop                 # 带颜色、可鼠标操作、更直观
# F6 排序，F9 杀进程，F5 树状视图
```

---

### rsync -- 文件同步

```bash
rsync -avz /src/ /dst/                    # 本地同步
rsync -avz /src/ user@host:/dst/          # 远程同步（通过 SSH）
rsync -avz --progress /src/ /dst/         # 显示进度
rsync -avz --exclude='*.log' /src/ /dst/  # 排除特定文件
rsync -avz --delete /src/ /dst/           # 镜像同步（删除目标中源不存在的文件）
rsync -av --update /src/ /dst/            # 只传输更新的文件
```

**冷知识**：rsync 删除大量文件比 `rm -rf` 更快：
```bash
mkdir /tmp/empty
rsync -a --delete /tmp/empty/ /target/dir/
```

---

### wget -- 网络下载

```bash
wget https://example.com/file.zip              # 下载文件
wget -c https://example.com/large.iso          # 断点续传
wget -O save.zip https://example.com/file      # 指定保存文件名
wget --limit-rate=1m https://example.com/iso   # 限速下载（避免带宽打满）
wget -r -l 0 https://example.com               # 递归下载整个网站
wget -q https://example.com/file               # 静默模式
```

**wget vs curl**：

| 特性 | wget | curl |
|------|------|------|
| 定位 | 专注于下载 | 多功能传输工具 |
| 递归下载 | 支持 | 不支持 |
| 断点续传 | 支持 | 支持 |
| 协议支持 | HTTP/HTTPS/FTP | 更多协议（含 SMTP、IMAP 等） |
| HTTP 方法 | 仅 GET/POST | 全部（GET/POST/PUT/DELETE 等） |
| 适用场景 | 文件批量下载 | API 调试、灵活请求 |

---

### nc (netcat) -- 网络调试工具

```bash
nc -zv host 80                    # 测试端口是否开放
nc -l 8080                        # 监听端口（简易服务器）
nc host 8080 < file.txt           # 发送文件
echo "hello" | nc host 80         # 发送数据到端口
nc -w 3 host 22                   # 3秒超时测试 SSH 端口
```

**实用场景**：快速测试端口连通性，比 telnet 更轻量。

---

### 其他实用命令速查

| 命令 | 功能 | 示例 |
|------|------|------|
| `sort` | 排序 | `sort -rn` 数字倒序 |
| `uniq` | 去重（需先排序） | `sort file \| uniq -c` 统计频次 |
| `cut` | 按列截取 | `cut -d: -f1 /etc/passwd` |
| `tee` | 输出到屏幕同时写文件 | `cmd \| tee log.txt` |
| `which` | 查找命令路径 | `which python` |
| `whereis` | 更广泛查找 | `whereis nginx` |
| `file` | 判断文件类型 | `file mystery_file` |
| `stat` | 查看文件详细信息 | `stat file.txt` |
| `lsof` | 查看打开的文件 | `lsof -i :80` 查看占用80端口的进程 |
| `uname` | 系统信息 | `uname -a` |
| `date` | 日期时间 | `date "+%Y-%m-%d %H:%M:%S"` |
| `watch` | 周期性执行命令 | `watch -n 2 df -h` 每2秒刷新 |
| `screen/tmux` | 终端复用 | 保持 SSH 断开后进程继续运行 |

---

### 管道与重定向

> [!tip] 比喻：管道 `|`
> 管道就像**工厂流水线**，上一个工序的产出直接传给下一个工序。比如 `cat file | grep error | sort` 就是：先读取文件，然后过滤出包含 error 的行，最后排序。

> [!tip] 比喻：重定向 `>` 和 `>>`
> `>` 是把杯子里的水**倒掉重新接**——覆盖文件内容。
> `>>` 是往杯子里**继续加水**——追加到文件末尾。

**基础知识**：
```bash
echo "123" > a.txt     # 覆盖写入（文件不存在则创建）
echo "456" >> a.txt    # 追加写入（文件不存在则创建）
```

**进阶**：
```bash
2>                     # 重定向标准错误
2>>                    # 追加标准错误
&>                     # 重定向标准输出和标准错误
> file                 # 会无警告地覆盖文件内容
```

**安全建议**：在脚本中使用 `set -o noclobber` 防止意外覆盖（用 `>|` 强制覆盖）。

---

## 实战命令组合精选

> [!example] 实战命令组合精选
> 把单个命令串起来解决实际问题，这才是命令行的真正威力。

**1. 系统状态概览（一条命令搞定）：**
```bash
top -b -n1 | awk '/%Cpu/ {print "CPU: "$2"%"}' && \
free -h | awk '/Mem/ {print "内存: "$3"/"$2" (可用: "$7")"}' && \
df -h | awk '/\/$/ {print "根分区: "$5" (剩余: "$4")"}'
```

**2. 查找大文件释放磁盘：**
```bash
find / -type f -size +100M 2>/dev/null | xargs du -sh | sort -hr
```

**3. 日志分析：统计错误最多的 URL：**
```bash
grep "500" access.log | awk '{print $7}' | sort | uniq -c | sort -nr | head -10
```

**4. 批量杀进程（先确认再执行）：**
```bash
# 先预览
ps aux | grep "keyword" | grep -v grep
# 再执行
ps aux | grep "keyword" | grep -v grep | awk '{print $2}' | xargs kill -15
```

**5. 查找最近被修改的敏感文件（安全审计）：**
```bash
find /etc -mtime -1 -type f 2>/dev/null
```

**6. 排查 SSH 暴力破解：**
```bash
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head
```

**7. 列出你最常用的 10 条命令：**
```bash
history | awk '{a[$2]++}END{for(i in a){print a[i] " " i}}' | sort -rn | head
```

**8. 批量修改配置文件并备份：**
```bash
grep -rnl --include="*.conf" "old_ip" | xargs -i sed -i.bak 's/old_ip=192.168.1.10/old_ip=192.168.1.20/g' {}
```

---

## 附录：过时命令与现代替代方案速查

| 过时命令 | 现代替代 | 说明 |
|----------|----------|------|
| `ifconfig` | `ip addr show` / `ip a` | net-tools 已废弃，iproute2 是现代标准 |
| `route -n` | `ip route show` | 同上 |
| `netstat -tlnp` | `ss -tlnp` | 同上 |
| `scp` | `sftp` / `rsync` | OpenSSH 9.0 起 scp 协议已废弃 |
| `more` | `less` | less 功能更强大，是现代默认分页器 |
| `yum` | `dnf` | RHEL/CentOS 8+ 已切换到 dnf |
| `service` | `systemctl` | systemd 是现代 Linux 标准 |

---

> **整合来源**：本文整合自研究员深度研究笔记和批评家准确性审查报告，整合日期 2026-04-30。
