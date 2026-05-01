以下是一份 **Docker Compose** 的 YAML 配置文件实例，附带逐行中文解释。即使你从未使用过电脑或 Docker，只要按顺序阅读，就能理解每一部分的作用。

---

## 示例目标
创建一个**极简的个人博客环境**，包含三个服务：
- **WordPress**（博客程序）
- **MySQL**（数据库）
- **phpMyAdmin**（数据库管理界面，可选）

所有服务通过 Docker Compose 一键启动，无需手动安装软件。

---

## 完整的 `docker-compose.yml` 文件

```yaml
# 告诉 Docker Compose 使用哪个版本的配置文件格式
# 3.8 是一个常见且稳定的版本，支持大部分功能
version: '3.8'

# “services” 代表你要运行的多个容器（每个服务等于一个容器）
services:
  # ---------- 第一个服务：数据库（MySQL） ----------
  # 服务的名称（类似人的小名），可以在其他服务里直接使用这个名字来连接
  db:
    # 使用哪个镜像（相当于软件的安装包）
    image: mysql:8.0
    # 容器总是自动重启（除非手动停止）
    restart: unless-stopped
    # 设置环境变量（相当于告诉 MySQL 初始化时该怎么做）
    environment:
      MYSQL_ROOT_PASSWORD: mysecretpw   # root 用户的密码
      MYSQL_DATABASE: wordpress         # 自动创建一个名叫 wordpress 的数据库
      MYSQL_USER: wpuser                # 创建一个普通用户
      MYSQL_PASSWORD: wpuserpw          # 普通用户的密码
    # 将容器里的数据库文件保存到宿主机的指定目录（永久保存，不会随容器删除而丢失）
    volumes:
      - ./mysql_data:/var/lib/mysql

  # ---------- 第二个服务：WordPress 博客程序 ----------
  wordpress:
    # 依赖 db 服务，等 db 启动后再启动本服务（确保数据库已准备好）
    depends_on:
      - db
    image: wordpress:latest
    restart: unless-stopped
    # 端口映射：将容器内部的 80 端口映射到宿主机的 8080 端口
    # 这样在电脑浏览器访问 http://localhost:8080 就能看到博客
    ports:
      - "8080:80"
    # 环境变量：告诉 WordPress 怎么连接数据库
    environment:
      WORDPRESS_DB_HOST: db:3306        # 数据库地址 = 服务名 db + 端口 3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wpuserpw
      WORDPRESS_DB_NAME: wordpress
    # 存储 WordPress 上传的图片、插件等文件
    volumes:
      - ./wordpress_data:/var/www/html

  # ---------- 第三个服务：phpMyAdmin（管理数据库的可视化界面） ----------
  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    restart: unless-stopped
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 1    # 允许连接到任意数据库服务器
```

---

## 怎么使用这个文件？

### 第 1 步：安装 Docker 和 Docker Compose
- **Windows / macOS**：安装 [Docker Desktop](https://docker.com/products/docker-desktop)，它自带 Compose。
- **Linux**：安装 Docker Engine，再单独安装 Docker Compose（一般用 `sudo apt install docker-compose`）。

### 第 2 步：创建一个文件夹并保存文件
1. 在电脑上新建一个文件夹，例如 `myblog`。
2. 用记事本（或 VS Code、Notepad++ 等）打开一个**新文件**。
3. 把上面的 YAML 内容完整复制粘贴进去。
4. 将文件保存为 `docker-compose.yml`（注意文件名必须是**复数** `compose`，扩展名是 `.yml` 或 `.yaml`）。

### 第 3 步：启动所有服务
- 打开终端（Windows 用 PowerShell 或 CMD，Mac 用“终端”）。
- 用 `cd` 命令进入之前创建的 `myblog` 文件夹。
- 运行命令：
  ```bash
  docker-compose up -d
  ```
  - `up` 表示启动。
  - `-d` 表示后台运行（不会一直显示日志）。

第一次启动会下载镜像（约几百 MB），之后会快很多。

### 第 4 步：访问你的博客
- 打开浏览器，输入 `http://localhost:8080` – 你会看到 WordPress 安装界面。
- 数据库管理：`http://localhost:8081` – 用 root 用户 `mysecretpw` 登录即可管理数据库。

### 第 5 步：停止和清理
```bash
docker-compose down          # 停止并删除容器（数据卷还在，数据不丢失）
docker-compose down -v       # 停止并删除容器 + 数据卷（所有数据会消失）
```

---

## 逐行深入解释（给完全零基础的读者）

### 第一行 `version: '3.8'`
- 就像给作文标版本号，Docker Compose 根据这个版本号决定哪些语法可以使用。用 `3.8` 安全可靠。

### `services:`
- 从此处开始定义你要运行的**服务**。一个服务就是一个容器（可以想象成一个轻量级的虚拟机）。

### `db:`、`wordpress:`、`phpmyadmin:`
- 服务的名字，可以随意取（比如 `database`、`myblog`）。其他服务可以通过这个名字访问它，例如 `db:3306`。

### `image: mysql:8.0`
- Docker 镜像类似于一个“干净的软件安装包”。`mysql:8.0` 表示从官方仓库（Docker Hub）下载 MySQL 8.0 版本。

### `restart: unless-stopped`
- 如果容器意外挂掉（比如程序崩溃），自动重启。除非你手动停止它，否则永远保持运行。

### `environment:`
- 设置容器内部的环境变量。MySQL 容器会根据这些变量自动创建数据库和用户，WordPress 容器会读取这些变量来连接数据库。

### `volumes: - ./mysql_data:/var/lib/mysql`
- 冒号左边：宿主机上的目录（例如当前目录下的 `mysql_data` 文件夹）。
- 冒号右边：容器内部的目录。
- 作用：容器里产生的文件（数据库文件、博客图片）会写到右边的目录，但实际保存在左边。即使删除容器，左边目录的文件还在，下次启动时自动恢复。

### `depends_on: - db`
- WordPress 和 phpMyAdmin 都依赖 `db` 服务，所以 Docker Compose 会先启动 `db`，再启动它们。

### `ports: - "8080:80"`
- 左边 8080 是你电脑（宿主机）的端口，右边 80 是容器内部的端口。访问 `localhost:8080` 就等于访问了容器内部的 80 端口。

### `WORDPRESS_DB_HOST: db:3306`
- 这里的 `db` 就是上面定义的服务的名字。在同一份 Compose 文件中，服务名会自动转换成容器的 IP 地址。

---

## 常见问题（小白必看）

**Q：我不小心写错了缩进怎么办？**  
A：YAML 对缩进非常敏感，必须使用**空格**，不能用 Tab。一般每层缩进 2 个空格。如果启动时报错 `yaml.scanner.ScannerError`，检查缩进。

**Q：端口冲突怎么办？**  
A：如果你的电脑上已经有软件占用了 8080 或 8081 端口，可以把左边的数字改成没被占用的，比如 `"8888:80"`。

**Q：怎么看到运行日志？**  
A：执行 `docker-compose logs -f`，可以实时查看所有服务的输出。

**Q：我想增加一个 Nginx 服务器，怎么写？**  
A：在 `services:` 下再添加一个新服务，例如：
```yaml
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
  volumes:
    - ./my_html:/usr/share/nginx/html
```

**Q：这个 YAML 文件能在生产环境用吗？**  
A：上面的例子适合本地学习和测试。生产环境需要进一步优化（比如数据库密码不要明文写在文件里，可以用 `.env` 文件或密钥管理）。

---

根据这份实例，你可以举一反三，搭建任何你想要的组合（比如 Redis + Node.js + MongoDB）。只要记住三个核心部分：**镜像**、**端口映射**、**环境变量**，再参考官方文档就能写出任意 YAML 文件。