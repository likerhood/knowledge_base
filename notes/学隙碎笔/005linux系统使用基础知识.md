# linux使用基础

[toc]

# 1. **Linux 系统结构**

Linux 是一个多用户、多任务的操作系统。它的核心部分是 **内核**，负责管理硬件资源，如 CPU、内存、硬盘等。它的文件系统结构是类似树形的，根目录 `/` 是最顶层，所有的文件和文件夹都从 `/` 目录开始。

- **/bin**：基本命令的二进制文件，如 `ls`, `cp`, `mv` 等。
- **/etc**：系统配置文件。
- **/home**：用户的家目录，每个用户都有自己的目录（如 `/home/user`）。
- **/root**：超级用户（root）的家目录。
- **/var**：可变文件，通常包含日志文件、缓存文件等。
- **/tmp**：临时文件存储目录。
- **/usr**：用户安装的程序和数据。

# 2. **文件和目录操作**

- **查看当前目录**：

  ```bash
  pwd
  ```

  `pwd`（print working directory）命令用来显示当前的工作目录。

- **列出目录中的文件和子目录**：

  ```bash
  ls
  ```

  `ls` 命令用于列出当前目录下的文件和目录。如果想查看详细信息，可以使用 `-l` 参数：

  ```bash
  ls -l
  ```

  使用 `-a` 参数可以显示所有文件，包括隐藏文件（以 `.` 开头的文件）：

  ```bash
  ls -la
  ```

- **切换目录**：

  ```bash
  cd [目录路径]
  ```

  `cd`（change directory）命令用于改变当前工作目录。例如，切换到 `/home/user` 目录：

  ```bash
  cd /home/user
  ```

  如果要返回上一级目录，可以使用：

  ```bash
  cd ..
  ```

  如果要回到家目录（当前用户的主目录），可以使用：

  ```bash
  cd ~
  ```

- **创建目录**：

  ```bash
  mkdir [目录名称]
  ```

  例如，创建一个名为 `myfolder` 的目录：

  ```bash
  mkdir myfolder
  ```

- **删除目录**：

  ```bash
  rmdir [目录名称]
  ```

  注意：`rmdir` 只能删除空目录。如果目录内有文件，可以使用 `rm -r` 来递归删除目录：

  ```bash
  rm -r myfolder
  ```

- **删除文件**：

  ```bash
  rm [文件名称]
  ```

  例如，删除 `test.txt` 文件：

  ```bash
  rm test.txt
  ```

  如果文件不可写，系统会提示你确认删除。你也可以使用 `-f` 参数强制删除：

  ```bash
  rm -f test.txt
  ```

# 3. **文件操作**

- **复制文件**：

  ```bash
  cp [源文件] [目标文件]
  ```

  例如，复制文件 `file1.txt` 到 `file2.txt`：

  ```bash
  cp file1.txt file2.txt
  ```

  如果想要复制目录，可以使用 `-r` 参数进行递归复制：

  ```bash
  cp -r dir1 dir2
  ```

- **移动或重命名文件**：

  ```bash
  mv [源文件] [目标文件]
  ```

  例如，移动文件 `file1.txt` 到 `dir1` 目录：

  ```bash
  mv file1.txt dir1/
  ```

  也可以用 `mv` 命令来重命名文件：

  ```bash
  mv oldname.txt newname.txt
  ```

- **查看文件内容**：

  - 使用 `cat` 命令查看文件的全部内容：

    ```bash
    cat [文件名]
    ```

  - 使用 `less` 命令逐页查看文件内容：

    ```bash
    less [文件名]
    ```

  - 使用 `head` 查看文件的前 10 行内容：

    ```bash
    head [文件名]
    ```

  - 使用 `tail` 查看文件的后 10 行内容：

    ```bash
    tail [文件名]
    ```

    如果想查看文件的最新内容，可以加上 `-f` 参数：

    ```bash
    tail -f [文件名]
    ```

# 4. **文件权限**

在 Linux 中，每个文件都有不同的权限，控制哪些用户可以读、写和执行该文件。可以使用 `chmod` 命令来修改文件权限。

- **查看文件权限**：

  ```bash
  ls -l [文件名]
  ```

  输出的第一个列就是文件权限，例如：

  ```
  -rwxr-xr-x 1 user user 1234 Jan 1 12:00 file1.txt
  ```

  这表示文件 `file1.txt` 拥有读取（r）、写入（w）和执行（x）权限。权限从左到右依次为文件所有者、同组用户和其他用户的权限。

- **修改文件权限**：

  使用 `chmod` 命令来修改文件权限。例如，赋予所有用户可执行权限：

  ```bash
  chmod +x file1.sh
  ```

  将文件的权限修改为只读：

  ```bash
  chmod 444 file1.txt
  ```

- **修改文件所有者**：

  使用 `chown` 命令来修改文件的拥有者：

  ```bash
  chown user:user file1.txt
  ```

  其中 `user:user` 指定了新的文件所有者和所属组。

# 5. **用户和权限管理**

- **查看当前用户**：

  ```bash
  whoami
  ```

- **查看当前系统的所有用户**：

  ```bash
  cat /etc/passwd
  ```

- **切换到其他用户（需要 root 权限）**：

  ```bash
  su [用户名]
  ```

- **添加新用户**（需要 root 权限）：

  ```bash
  sudo useradd [用户名]
  ```

  然后可以设置密码：

  ```bash
  sudo passwd [用户名]
  ```

- **删除用户**（需要 root 权限）：

  ```bash
  sudo userdel [用户名]
  ```

# 6. **系统管理**

- **查看磁盘空间**：

  ```bash
  df -h
  ```

  `-h` 参数让输出更容易阅读，以 GB 或 MB 为单位显示。

- **查看内存使用情况**：

  ```bash
  free -h
  ```

- **查看系统资源使用情况**：

  ```bash
  top
  ```

  该命令会显示系统中当前的进程、CPU 使用率、内存使用情况等。

- **查看进程**：

  ```bash
  ps aux
  ```

  该命令显示当前系统中所有的进程。

# 7. **网络管理**

- **查看网络连接情况**：

  ```bash
  netstat -tuln
  ```

- **查看 IP 地址**：

  ```bash
  ip addr show
  ```

- **ping 命令测试网络连通性**：

  ```bash
  ping [IP 地址或域名]
  ```

# 8. **安装和管理软件**

- **使用 `yum` 安装软件（适用于 CentOS/RHEL 系统）**：

  ```bash
  sudo yum install [软件包]
  ```

- **使用 `apt-get` 安装软件（适用于 Ubuntu/Debian 系统）**：

  ```bash
  sudo apt-get install [软件包]
  ```

# 9. vim使用

- **打开进入文件**

  ```bash
  vim 文件名
  ```

- **修改文件**

  ```bash
  键盘输入  i
  ```

- **保存文件并退出**

  ```bash
  键盘输入 :wq
  ```

  





























