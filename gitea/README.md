`gitea`是一个很轻量化的git服务端程序

[官网文档](https://docs.gitea.com/zh-cn/)

## step.0

到[官网下载地址](https://dl.gitea.com/gitea/) 下载不同版本的对应版本

运行`git --version`命令检查git版本呢，git版本需要>=2.0

并将文件下载放到`/usr/local/bin`目录下，命名为`gitea`

并授权运行

```sh
chmod 755 /usr/local/bin/gitea
```

## stop.1

创建一个新用户，用于运行`gitea`

```sh
adduser \
   --system \
   --shell /bin/bash \
   --gecos 'Git Version Control' \
   --group \
   --disabled-password \
   --home /home/git \
   git
```

创建文件夹存放相关文件

```sh
# 存放日志数据等等
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
# 存放配置文件等
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea
```

## step.2

通过以下命令运行`gitea`

```sh
GITEA_WORK_DIR=/var/lib/gitea/ /usr/local/bin/gitea web -c /etc/gitea/app.ini
```

访问3000端口就可以配置系统了

之后就可以`ctrl c`退出系统了

## step.3

使用service方式运行

编辑`service`文件

```sh
sudo vim /etc/systemd/system/gitea.service
```

写入以下 内容

```properties
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
[Service]
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```

运行service

```sh
sudo systemctl enable gitea
sudo systemctl start gitea
```

## other

附上`app.ini`配置

```properties
APP_NAME = dbins git
RUN_USER = git
RUN_MODE = prod

[database]
DB_TYPE  = sqlite3
HOST     = 127.0.0.1:3306
NAME     = gitea
USER     = gitea
PASSWD   =
SCHEMA   =
SSL_MODE = disable
CHARSET  = utf8
PATH     = 
LOG_SQL  = false

[repository]
ROOT = 

[server]
SSH_DOMAIN       = git.dbinfun.net
DOMAIN           = git.dbinfun.net
HTTP_PORT        = 3000
ROOT_URL         = https://git.dbinfun.net/
# 是否禁用ssh
DISABLE_SSH      = false
# 开始ssh服务
START_SSH_SERVER = true
SSH_PORT         = 2222
LFS_START_SERVER = true
LFS_JWT_SECRET   = 
OFFLINE_MODE     = false

[lfs]
PATH = 

[mailer]
ENABLED = false

[service]
REGISTER_EMAIL_CONFIRM            = false
ENABLE_NOTIFY_MAIL                = false
DISABLE_REGISTRATION              = true
ALLOW_ONLY_EXTERNAL_REGISTRATION  = false
ENABLE_CAPTCHA                    = false
REQUIRE_SIGNIN_VIEW               = false
DEFAULT_KEEP_EMAIL_PRIVATE        = false
DEFAULT_ALLOW_CREATE_ORGANIZATION = true
DEFAULT_ENABLE_TIMETRACKING       = true
NO_REPLY_ADDRESS                  = noreply.localhost
SHOW_REGISTRATION_BUTTON          = false

[openid]
ENABLE_OPENID_SIGNIN = true
ENABLE_OPENID_SIGNUP = true

[cron.update_checker]
ENABLED = false

[session]
PROVIDER = file

[log]
MODE      = console
LEVEL     = info
ROOT_PATH = /home/git/gitea/log
ROUTER    = console

[repository.pull-request]
DEFAULT_MERGE_STYLE = merge

[repository.signing]
DEFAULT_TRUST_MODEL = committer

[security]
INSTALL_LOCK       = true
INTERNAL_TOKEN     = 
PASSWORD_HASH_ALGO = pbkdf2
```

