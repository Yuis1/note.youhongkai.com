---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/OS/Linux系统踩坑杂记/","noteIcon":"","created":"2024-01-05T05:25:47.685+08:00","updated":"2024-04-24T00:38:30.093+08:00"}
---


作者：游鱼思

---
## CentOS 的麻烦事

在CentOS 7下，如果yum install php-devel php-pear ，只会默认给你安装5.4版本的，配合宝塔真是乱上加乱。

当您发现通过 `yum` 提供的软件包版本过低时，您有几个选项可以尝试获取更新的版本：

### 1. 添加新的仓库

您可以添加提供更新软件包的第三方仓库。对于许多流行的软件，如 PHP、MySQL、Nginx 等，都有专门的仓库提供最新版本。

- **EPEL (Extra Packages for Enterprise Linux)**:
  对于 CentOS 和 Red Hat 系统，EPEL 仓库提供了一些更新的包。可以通过运行以下命令来添加 EPEL 仓库：
  ```bash
  sudo yum install epel-release
  ```

- **Remi Repository**:
  对于 PHP，Remi 仓库是一个流行的选择，提供了最新版本的 PHP。您可以通过访问 Remi 的网站获取安装说明：[Remi's RPM repository](https://rpms.remirepo.net/wizard/)

- **其他专用仓库**:
  根据您需要更新的软件，可能还有其他专用的仓库可用。

### 2. 编译安装

如果通过添加仓库仍然无法获取所需版本的软件，您可以选择从源代码编译安装。这通常意味着下载软件的最新源代码，然后在您的系统上编译和安装它。这种方法更复杂，但允许您完全控制安装过程和配置。

编译安装的基本步骤通常包括：

- 下载最新的源代码。
- 安装必要的编译工具和依赖项。
- 配置、编译和安装软件。

### 3. 使用软件包管理工具

对于某些软件，您可以使用其他软件包管理工具，如 `dnf`（在较新的 Fedora 和 RHEL 版本中替代 `yum`）或 `pip`（Python），`npm`（Node.js）等。

### 4. 检查官方源或预编译的二进制文件

一些软件项目提供了预编译的二进制文件或官方的软件仓库，这些可以直接用于安装最新版本的软件。

### 注意事项

- 添加第三方仓库可能会引入新的依赖和潜在的安全风险。在添加新仓库之前，请确保它是可信赖和安全的。
- 从源代码编译软件可能需要较多的时间和更高的技术水平。此外，编译安装的软件可能难以通过标准的包管理工具进行管理和更新。
- 在生产环境中升级关键软件时，请先在测试环境中验证，以避免可能的兼容性问题。

Command to install the EPEL repository configuration package:

    yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

Command to install the Remi repository configuration package:

    yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm

Command to install the yum-utils package (for the yum-config-manager command):

    yum install yum-utils

启用php 8.3 仓库： yum-config-manager --enable remi-php83

安装PHP 开发工具包（php-devel），这对于编译 PECL 扩展是必需的。并且此时发现，待确认安装的包已经变成了8.3版本。但是要注意，这时安装的php是和宝塔面板的php独立的，并且php-cli的默认命令也将被切到新安装的位置。

yum install php-devel

安装PECL：yum install php-pear

也可以直接找PECL的安装包：yum search php83-php-pecl

这下直接安装php扩展了，绕过中间商pear、pecl等乱七八糟的：

对于php8.3： yum install php83-php-pecl-uploadprogress.x86_64

安装的所有包都在 /opt/remi/php83/root/usr/lib64/php/modules

需要在php8.3 的 ini 文件中指定 extension=/opt/remi/php83/root/usr/lib64/php/modules/uploadprogress.so

## 一团乱麻的环境变量配置

/etc/profile：这个文件是系统级别的配置文件，所有用户的 shell 启动时都会读取这个文件。通常这个文件中包含的是对所有用户都有效的设置。

~/.bash_profile、~/.bash_login、~/.profile：这些文件是用户级别的配置文件，只对当前用户有效。当一个登录 shell 启动时（例如你在终端中输入用户名和密码登录时），它会首先查找 ~/.bash_profile，如果这个文件存在，就只读取这个文件。如果 ~/.bash_profile 不存在，它会查找 ~/.bash_login，如果这个文件存在，就只读取这个文件。如果 ~/.bash_login 也不存在，它会查找 ~/.profile，如果这个文件存在，就读取这个文件。

~/.bashrc：这个文件也是用户级别的配置文件，只对当前用户有效。但它只在非登录 shell 启动时读取（例如你在图形界面中打开一个新的终端窗口时）。注意，很多系统的 ~/.bash_profile 文件中会包含一行代码 [[ -f ~/.bashrc \| -f ~/.bashrc ]] && . ~/.bashrc，这行代码的意思是，如果 ~/.bashrc 文件存在，就读取它。这样，无论是登录 shell 还是非登录 shell 启动时，~/.bashrc 都会被读取。

~/.zshrc：这个文件是 zsh 的配置文件，只在 zsh 启动时读取。它的作用类似于 bash 的 ~/.bashrc。

总的来说，/etc/profile 和 ~/.bash_profile（或 ~/.bash_login 或 ~/.profile）是在登录 shell 启动时读取的，~/.bashrc 是在非登录 shell 启动时读取的，~/.zshrc 是在 zsh 启动时读取的。

## Linux 服务

要将一个 Python 脚本（如 `service.py`）添加为 Linux 系统服务，并使其在启动时自动运行，你可以创建一个 systemd 服务单元文件。systemd 是大多数现代 Linux 发行版所使用的初始化系统和服务管理器。以下是将 Python 脚本设置为服务并使其自启的步骤：

1. **创建 systemd 服务单元文件**:
   - 创建一个新的服务单元文件（例如 `myservice.service`）在 `/etc/systemd/system/` 目录下。
   - 编辑这个文件以定义服务的属性。

2. **编辑服务单元文件**:
   - 指定服务的描述、启动类型、执行命令等。
   - 例如：

	 ```ini
	 [Unit]
	 Description=My Python Service
	 After=network.target

	 [Service]
	 ExecStart=/usr/bin/python3 /path/to/service.py
	 Restart=on-failure

	 [Install]
	 WantedBy=multi-user.target
	 ```

	 - 替换 `/usr/bin/python3` 和 `/path/to/service.py` 为你的 Python 解释器路径和你的脚本路径。

3. **启用和启动服务**:
   - 重新加载 systemd，以便它知道新的服务：
	 ```bash
	 sudo systemctl daemon-reload
	 ```
   - 启用服务，使其在启动时自动运行：
	 ```bash
	 sudo systemctl enable myservice.service
	 ```
	 

	 删除服务：

	 sudo systemctl disable myservice.service

	 
   - 启动服务：
	 ```bash
	 sudo systemctl start myservice.service
	 ```

4. **检查服务状态**:
   - 你可以使用以下命令检查服务的状态：
	 ```bash
	 sudo systemctl status myservice.service
	 ```

5. **日志和调试**:
   - 如果服务没有按预期工作，你可以查看其日志来进行调试：
	 ```bash
	 sudo journalctl -u myservice.service
	 ```

6. **更新服务脚本**:
   - 如果你更改了 `service.py` 脚本，你需要重启服务以应用这些更改：
	 ```bash
	 sudo systemctl restart myservice.service
	 ```

确保你的 Python 脚本是可执行的，并且具有正确的 shebang（如 `#!/usr/bin/env python3`），这样它就可以作为一个独立的程序运行。此外，请确保脚本在你计划运行服务的用户权限下可以正常运行，包括所有的路径和权限设置。

## 查看服务输出内容

如果你想要在启动服务的同时查看其输出，你可以使用 journalctl 命令。journalctl 是一个用于查看 systemd 管理的服务日志的工具。

以下是如何使用 journalctl 来查看 quant_backend_test 服务的输出：

systemctl start quant_backend_test

journalctl -u quant_backend_test -f

在这个命令中，-u 参数用于指定服务名，-f 参数用于持续查看新的日志输出。

请注意，你可能需要 sudo 权限来运行这些命令。

## 服务不会读取用户环境变量！

Systemd 服务默认不会读取用户的 `.bashrc` 文件。Systemd 服务运行时，它使用的是一个非交互式、非登录的 shell 环境，这意味着 `.bashrc` 和其他类似的 shell 初始化脚本不会自动执行。

如果你的 `quant_backend_test.service` 依赖于在 `www` 用户的 `.bashrc` 中设置的特定环境变量或其他配置，你有几种方法可以确保这些设置被应用：

1. **在服务文件中直接设置环境变量**：
   - 你可以在 Systemd 服务文件中使用 `Environment` 或 `EnvironmentFile` 指令来设置所需的环境变量。例如：
	 ```ini
	 [Service]
	 Environment="VAR1=value1" "VAR2=value2"
	 ```
   - 或者，如果环境变量很多，可以将它们放在一个文件中，然后使用 `EnvironmentFile` 指向该文件：
	 ```ini
	 [Service]
	 EnvironmentFile=/path/to/env_file
	 ```

2. **在 `ExecStart` 前执行脚本**：
   - 如果需要执行更复杂的初始化脚本，可以在 `ExecStart` 前使用 `ExecStartPre` 来运行一个脚本，该脚本可以负责设置环境、准备资源等。
   - 例如：
	 ```ini
	 [Service]
	 ExecStartPre=/path/to/preparation_script.sh
	 ExecStart=python3 /www/wwwroot/quant_backend_test/server.py
	 ```

3. **创建一个包装脚本**：
   - 创建一个 shell 脚本，该脚本首先源（source）`.bashrc` 或执行任何必要的初始化，然后启动你的 Python 服务。
   - 然后在 Systemd 服务文件中使用这个包装脚本作为 `ExecStart`：
	 ```ini
	 [Service]
	 ExecStart=/path/to/wrapper_script.sh
	 ```
   - 在 `wrapper_script.sh` 中，你可以有类似于这样的内容：
	 ```bash
	 #!/bin/bash
	 source /home/www/.bashrc
	 python3 /www/wwwroot/quant_backend_test/server.py
	 ```

选择适合你的场景的方法，确保在 Systemd 服务中包含所有必要的环境设置。

在 Linux 系统中设置全局环境变量，可以通过修改系统级的配置文件来实现。这样设置的环境变量将对所有用户和服务有效。以下是几种常见的设置全局环境变量的方法：

1. **`/etc/environment`**:
   - 这是设置环境变量的一个通用方法，适用于所有类型的会话和用户。
   - 编辑 `/etc/environment` 文件，并添加你的环境变量。例如：
	 ```
	 MY_VAR="my_value"
	 ```
   - 此文件仅用于设置环境变量，不是一个脚本文件，所以不需要 `export` 命令。

2. **`/etc/profile`**:
   - `/etc/profile` 文件在登录时被执行，并且是设置全局环境变量的传统位置。
   - 对于使用 Bash 作为登录 shell 的用户，此文件将在登录时执行。
   - 在此文件中添加 `export` 命令设置环境变量。例如：
	 ```bash
	 export MY_VAR="my_value"
	 ```

3. **`/etc/profile.d/`**:
   - 你可以在 `/etc/profile.d/` 目录下创建一个新的脚本文件来设置环境变量。
   - 这个目录中的所有脚本文件都会在登录时自动执行。
   - 例如，创建一个名为 `my_vars.sh` 的文件，并添加以下内容：
	 ```bash
	 export MY_VAR="my_value"
	 ```

4. **对于 Systemd 服务**:
   - 尽管上述方法会对登录用户的 shell 环境变量产生影响，但对于通过 Systemd 启动的服务可能不一定适用。
   - 对于 Systemd 服务，通常需要在服务的配置文件中明确设置环境变量（使用 `Environment` 或 `EnvironmentFile`）或通过修改服务的启动脚本来包含所需的环境变量。

5. **重新加载和应用更改**:
   - 修改环境变量配置文件后，你可能需要注销并重新登录，或重启系统来使更改生效。
   - 对于 Systemd 服务，修改后运行 `sudo systemctl daemon-reload` 以重新加载配置，然后重新启动相关服务。

选择适合你需求的方法。如果是针对所有用户和服务的全局配置，`/etc/environment` 是最简单直接的选择。如果需要更复杂的设置或脚本执行，可以选择 `/etc/profile` 或 `/etc/profile.d/`。对于 Systemd 服务，你可能仍然需要在服务文件中指定环境变量，特别是当这些变量是服务运行所特有的。

## 服务器安全

为了增强SSH安全性并自动阻止多次登录失败尝试的IP地址，您可以采取以下措施：

1. **安装和配置 `fail2ban`**:
   - `fail2ban` 是一个流行的工具，它监控日志文件（如 `/var/log/auth.log`）并基于配置的规则封锁频繁失败尝试的IP地址。
   - 安装 `fail2ban`（通常在大多数Linux发行版的官方仓库中可找到）。
   - 配置 `fail2ban` 的`jail.conf`文件，为SSH设置一个“监狱”（jail），指定封锁策略和持续时间。

2. **修改SSH配置**:
   - 编辑 `/etc/ssh/sshd_config` 文件，可以设置如密码尝试次数限制等。
   - 禁用基于密码的登录，强制使用SSH密钥。

3. **更改SSH端口**:
   - 通过在 `/etc/ssh/sshd_config` 中更改端口号，将SSH从默认的22端口移至其他端口。

4. **使用TCP Wrappers**:
   - TCP Wrappers提供了另一种限制对SSH服务访问的方法。通过编辑 `/etc/hosts.allow` 和 `/etc/hosts.deny` 文件，您可以控制哪些IP地址被允许或拒绝访问SSH。

5. **设置防火墙规则**:
   - 使用防火墙（如 iptables 或 ufw）来限制对SSH端口的访问。例如，仅允许来自特定IP地址或网络的访问。

6. **使用端口敲门技术** (Port Knocking):
   - 这是一种安全技术，其中SSH端口默认情况下是关闭的，只有在收到特定的端口敲门序列后才会打开。

7. **定期审查和监控日志**:
   - 定期检查SSH日志，确保没有未授权的访问。

### 例子: 配置 `fail2ban` 以保护SSH

1. 安装 `fail2ban`:
   ```bash
   sudo apt-get install fail2ban
   ```

2. 创建 `jail.local` 文件以覆盖默认设置:
   ```bash
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   sudo nano /etc/fail2ban/jail.local
   ```

3. 在 `jail.local` 文件中配置SSH规则:
```
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 3600
```

   - `enabled`: 激活监狱。
   - `maxretry`: 最大尝试次数。
   - `bantime`: 封锁时间（秒）。

4. 重新启动 `fail2ban` 服务以应用更改:
   ```bash
   sudo systemctl restart fail2ban
   ```

通过以上措施，您可以有效增强SSH的安全性，并且自动阻止频繁失败尝试登录的IP地址。

要配置 SSH 以仅支持通过密钥登录，您需要在 SSH 服务的配置文件中做一些更改。这样可以提高安全性，因为密钥登录通常比密码登录更难被破解。以下是配置步骤：

1. **编辑 SSH 配置文件**：
   打开 `/etc/ssh/sshd_config` 文件进行编辑。您可以使用您喜欢的文本编辑器，例如使用 `nano`：
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **禁用密码认证**：
   在 `sshd_config` 文件中，找到以下行（如果某些行不存在，您可能需要添加它们）：
   - 找到 `PasswordAuthentication`，将其更改为 `PasswordAuthentication no`。
   - 确保 `ChallengeResponseAuthentication` 设置为 `no`。
   - 找到 `UsePAM`，将其更改为 `UsePAM no`。

   这些更改将禁用密码认证，仅允许通过 SSH 密钥进行认证。

3. **确保公钥认证已启用**：
   确保文件中的 `PubkeyAuthentication` 选项设置为 `yes`。通常这是默认设置，但最好检查一下。

4. **保存并关闭文件**：
   对于 `nano`，按下 `Ctrl+O` 保存文件，然后按 `Ctrl+X` 退出。

5. **重启 SSH 服务**：
   为了使更改生效，重启 SSH 服务：
   ```bash
   sudo systemctl restart sshd
   ```

6. **测试配置**：
   在完全断开当前 SSH 会话之前，最好在新的终端窗口中测试密钥登录是否工作。这样做是为了确保新的设置没有问题，并且您不会被锁在外面。

   试图使用 SSH 密钥连接到服务器，确保能够成功登录。如果出现任何问题，请回到原始会话窗口撤销所做的更改或进行故障排除。

7. **生成并分发 SSH 密钥（如果还没做）**：
   如果您或其他用户还没有生成 SSH 密钥，需要在客户端机器上生成它们，并将公钥部分添加到服务器的 `~/.ssh/authorized_keys` 文件中。

通过以上步骤，您将能够配置 SSH 服务，使其仅支持通过密钥进行认证，从而提高安全性。在进行这些更改时，请务必谨慎行事，以免意外地被锁定在系统外。

## 文件系统

在Linux操作系统下，单个目录可以存储的最大文件数量主要受文件系统类型和其设计限制的影响。不同的文件系统有不同的设计和限制，以下是一些常见文件系统类型及其单个目录可以存储的最大文件数量限制的概述：

1. **ext3**：
   - 最大文件数：目录内的文件数量理论上没有硬性限制，但性能会随着文件数量的增加而下降。在实际应用中，数千到数万个文件可能是一个性能的临界点。
   - 注意：ext3使用链接列表来存储目录项，因此当目录非常大时，目录访问效率会显著下降。

2. **ext4**：
   - 最大文件数：理论上最大支持约4 billion（2^32）个文件，但实际上，单个目录下存储的文件数量对性能的影响会在更小的数量级出现。
   - 特点：ext4引入了HTree索引（一种基于哈希的目录索引技术），大大提高了大目录的访问效率，使得在单个目录下存储成百上千万个文件成为可能，同时保持较好的性能。

3. **XFS**：
   - 最大文件数：理论上无硬性限制，XFS支持极大数量的文件存储在单个目录中。
   - 特点：XFS是为高性能和大容量存储而设计的文件系统，能够高效管理大量的文件，适用于大型数据库和数据仓库。

4. **Btrfs**：
   - 最大文件数：理论上无硬性限制，类似于XFS，Btrfs也是设计来支持大量文件的存储。
   - 特点：Btrfs支持高级功能，如快照、动态inode分配等，这使得它在处理大量文件时更加灵活和高效。

5. **ReiserFS**（主要是ReiserFS v3）：
   - 最大文件数：同样没有明确的硬性限制，但设计上更适合处理大量小文件。
   - 注意：虽然ReiserFS因其在处理大量小文件方面的优势而闻名，但该文件系统的使用和发展已经相对减少。

总的来说，现代Linux文件系统（如ext4、XFS、Btrfs）都设计有能力处理大量文件存储在单个目录下，但从实际性能和管理的角度出发，推荐的做法是避免在单个目录中存储过多文件。当目录中的文件数量达到数十万或更多时，即使文件系统技术上支持，性能也可能会受到影响，操作如列表目录内容或搜索文件可能会变得缓慢。因此，合理规划目录结构，将文件分布到不同的目录中，是管理大型文件系统的一个好方法。

## 电源管理

禁用休眠 sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target

问题：Ubuntu在未登录时，没多久就自动休眠了。

解决办法：

通过修改`/etc/systemd/logind.conf`文件来调整`systemd-logind`服务的行为，包括控制在未登录时是否允许系统挂起。`logind.conf`提供了多个相关设置，可以帮助你实现这个目的。

要修改这些设置，请按照以下步骤操作：

1. /etc/systemd/logind.conf
2. **修改相关设置**：

   查找以下几个相关的设置项（它们可能被注释掉了，以`#`开头）：

   - `HandleLidSwitch=`：控制当笔记本电脑盖子关闭时的行为。
   - `HandleLidSwitchExternalPower=`：控制当笔记本电脑连接到外部电源并关闭盖子时的行为。
   - `HandleLidSwitchDocked=`：控制当笔记本电脑处于底座并关闭盖子时的行为。
   - `IdleAction=`：控制当系统处于空闲状态时的行为。
   - `IdleActionSec=`：定义了系统被认为处于空闲状态的时间。

   你可以将这些设置项的值改为`ignore`（忽略盖子的关闭）、`poweroff`（关机）、`reboot`（重启）、`halt`（停止）、`kexec`（使用kexec加载新内核）或`suspend`（挂起）。例如，如果你想在未登录时不让系统挂起，可以设置：

   ```
   HandleLidSwitch=ignore
   HandleLidSwitchExternalPower=ignore
   HandleLidSwitchDocked=ignore
   IdleAction=ignore
   ```

   这会让`systemd-logind`忽略盖子的关闭和空闲状态，从而避免自动挂起。

3. **保存并关闭文件**：

   如果你使用的是`nano`，可以按`Ctrl + O`保存更改，然后按`Ctrl + X`退出。

4. **重新加载`systemd-logind`服务**：

   保存更改后，需要重新加载`systemd-logind`服务以应用更改。运行以下命令：

   ```bash
   sudo systemctl restart systemd-logind
   ```

   注意：重新加载`systemd-logind`服务可能会导致当前会话断开，所以最好在执行这个操作前保存所有未保存的工作。

通过这些步骤，你可以配置系统在未登录时不自动挂起。这些设置应当解决系统在未登录时自动挂起的问题。