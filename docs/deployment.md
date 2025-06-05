# 微信小游戏MUD量子叙事部署文档

## 部署概述

本文档详细说明了微信小游戏MUD量子叙事项目的部署流程，包括服务器配置、环境搭建、应用部署和维护指南。该项目采用单台腾讯云服务器部署后端服务，前端通过微信小游戏平台发布。

## 服务器配置要求

### 1. 硬件配置建议

根据预期的用户规模和系统复杂度，推荐以下腾讯云服务器配置：

#### 初期阶段（0-500用户）
- **实例类型**：轻量应用服务器或云服务器CVM
- **CPU**：2核
- **内存**：4GB
- **存储**：50GB SSD云硬盘
- **带宽**：5Mbps（按使用流量计费）
- **地域**：中国大陆（建议选择靠近目标用户群的地域，如华东、华南或华北）

#### 中期阶段（500-2000用户）
- **实例类型**：云服务器CVM
- **CPU**：4核
- **内存**：8GB
- **存储**：100GB SSD云硬盘
- **带宽**：10Mbps（按使用流量计费）
- **地域**：同上

#### 后期阶段（2000+用户）
- **实例类型**：云服务器CVM
- **CPU**：8核
- **内存**：16GB
- **存储**：200GB SSD云硬盘
- **带宽**：20Mbps或更高（按使用流量计费）
- **地域**：同上，考虑多地域部署

### 2. 操作系统选择

推荐使用以下操作系统：
- **Ubuntu Server 20.04 LTS**（推荐，对Node.js支持良好）
- **CentOS 7/8**（稳定性好，但部分软件包可能较旧）

### 3. 网络配置

- **安全组设置**：
  - 开放80端口（HTTP）
  - 开放443端口（HTTPS）
  - 开放服务器WebSocket端口（如8080）
  - 开放SSH端口（22），限制IP访问
  - 开放MongoDB端口（27017），限制IP访问
  - 开放Redis端口（6379），限制IP访问

- **域名配置**：
  - 购买域名并完成备案（中国大陆服务器必须）
  - 设置A记录指向服务器IP
  - 配置SSL证书（推荐使用Let's Encrypt免费证书）

## 服务器环境搭建

### 1. 基础环境安装

以Ubuntu Server 20.04为例：

```bash
# 更新系统
sudo apt update
sudo apt upgrade -y

# 安装基础工具
sudo apt install -y git curl wget vim build-essential

# 安装Node.js (使用NVM)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
source ~/.bashrc
nvm install 16  # 或更新版本
nvm use 16
nvm alias default 16

# 验证Node.js安装
node -v
npm -v

# 安装PM2进程管理器
npm install -g pm2
```

### 2. MongoDB安装

```bash
# 导入MongoDB公钥
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -

# 添加MongoDB源
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list

# 更新包列表
sudo apt update

# 安装MongoDB
sudo apt install -y mongodb-org

# 启动MongoDB并设置开机自启
sudo systemctl start mongod
sudo systemctl enable mongod

# 验证MongoDB安装
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
```

### 3. Redis安装

```bash
# 安装Redis
sudo apt install -y redis-server

# 配置Redis
sudo nano /etc/redis/redis.conf
# 修改以下配置:
# 1. 将 supervised no 改为 supervised systemd
# 2. 设置密码: requirepass your_strong_password
# 3. 如果需要远程访问，注释掉 bind 127.0.0.1 ::1

# 重启Redis服务
sudo systemctl restart redis.service
sudo systemctl enable redis.service

# 验证Redis安装
redis-cli ping  # 应返回PONG
```

### 4. Nginx安装与配置

```bash
# 安装Nginx
sudo apt install -y nginx

# 启动Nginx并设置开机自启
sudo systemctl start nginx
sudo systemctl enable nginx

# 配置Nginx
sudo nano /etc/nginx/sites-available/mud-game

# 添加以下配置
server {
    listen 80;
    server_name your-domain.com;  # 替换为你的域名

    location / {
        proxy_pass http://localhost:3000;  # Node.js服务端口
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /socket.io/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

# 创建符号链接启用站点
sudo ln -s /etc/nginx/sites-available/mud-game /etc/nginx/sites-enabled/

# 测试Nginx配置
sudo nginx -t

# 重启Nginx
sudo systemctl restart nginx
```

### 5. SSL证书配置（Let's Encrypt）

```bash
# 安装Certbot
sudo apt install -y certbot python3-certbot-nginx

# 获取SSL证书
sudo certbot --nginx -d your-domain.com

# 证书自动续期
sudo systemctl status certbot.timer
```

## 应用部署流程

### 1. 代码部署

#### 手动部署

```bash
# 创建应用目录
mkdir -p /var/www/mud-game
cd /var/www/mud-game

# 克隆代码仓库
git clone https://github.com/your-username/mud-quantum-narrative.git .

# 安装依赖
npm install --production

# 创建环境配置文件
cp .env.example .env
nano .env  # 编辑环境变量
```

#### 使用部署脚本

创建部署脚本 `deploy.sh`：

```bash
#!/bin/bash

# 部署脚本
echo "开始部署MUD量子叙事游戏..."

# 进入项目目录
cd /var/www/mud-game

# 拉取最新代码
git pull

# 安装依赖
npm install --production

# 重启应用
pm2 restart mud-game

echo "部署完成!"
```

设置脚本权限并执行：

```bash
chmod +x deploy.sh
./deploy.sh
```

### 2. 数据库初始化

```bash
# 进入项目目录
cd /var/www/mud-game

# 运行数据库初始化脚本
node scripts/init-db.js
```

### 3. 使用PM2启动应用

```bash
# 进入项目目录
cd /var/www/mud-game

# 使用PM2启动应用
pm2 start app.js --name "mud-game" --watch

# 保存PM2配置
pm2 save

# 设置PM2开机自启
pm2 startup
```

### 4. 微信小游戏前端部署

1. **打开微信开发者工具**
2. **导入项目**：选择项目中的 `minigame` 目录
3. **配置服务器地址**：
   - 打开 `minigame/js/network/config.js`
   - 修改 `SERVER_URL` 为你的服务器域名
   ```javascript
   export const SERVER_URL = 'wss://your-domain.com';
   ```
4. **上传代码**：
   - 点击"上传"按钮
   - 填写版本号和项目备注
   - 等待上传完成
5. **提交审核**：
   - 在微信小游戏后台提交审核
   - 等待审核通过
   - 发布小游戏

## 数据备份与维护

### 1. 数据库备份

#### MongoDB备份脚本

创建备份脚本 `backup-mongodb.sh`：

```bash
#!/bin/bash

# 设置变量
BACKUP_DIR="/var/backups/mongodb"
DATE=$(date +"%Y%m%d_%H%M%S")
MONGODB_DATABASE="mud_game"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 执行备份
mongodump --db $MONGODB_DATABASE --out $BACKUP_DIR/$DATE

# 压缩备份
cd $BACKUP_DIR
tar -zcvf $DATE.tar.gz $DATE
rm -rf $DATE

# 删除7天前的备份
find $BACKUP_DIR -name "*.tar.gz" -type f -mtime +7 -delete

echo "MongoDB备份完成: $BACKUP_DIR/$DATE.tar.gz"
```

设置定时任务：

```bash
# 编辑crontab
crontab -e

# 添加定时任务，每天凌晨3点执行备份
0 3 * * * /var/www/mud-game/scripts/backup-mongodb.sh >> /var/log/mongodb-backup.log 2>&1
```

#### Redis备份

创建备份脚本 `backup-redis.sh`：

```bash
#!/bin/bash

# 设置变量
BACKUP_DIR="/var/backups/redis"
DATE=$(date +"%Y%m%d_%H%M%S")
REDIS_PASSWORD="your_redis_password"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 执行备份
redis-cli -a $REDIS_PASSWORD --rdb $BACKUP_DIR/dump_$DATE.rdb

# 删除7天前的备份
find $BACKUP_DIR -name "dump_*.rdb" -type f -mtime +7 -delete

echo "Redis备份完成: $BACKUP_DIR/dump_$DATE.rdb"
```

设置定时任务：

```bash
# 编辑crontab
crontab -e

# 添加定时任务，每天凌晨3点30分执行备份
30 3 * * * /var/www/mud-game/scripts/backup-redis.sh >> /var/log/redis-backup.log 2>&1
```

### 2. 日志管理

#### 配置日志轮转

创建日志轮转配置 `/etc/logrotate.d/mud-game`：

```
/var/www/mud-game/logs/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
    sharedscripts
    postrotate
        [ -s /var/www/mud-game/logs/pm2.pid ] && kill -USR2 `cat /var/www/mud-game/logs/pm2.pid`
    endscript
}
```

#### 设置应用日志

在应用中配置日志系统，将日志输出到 `/var/www/mud-game/logs/` 目录。

### 3. 监控系统

#### 使用PM2监控

```bash
# 查看应用状态
pm2 status

# 查看应用日志
pm2 logs mud-game

# 查看资源使用情况
pm2 monit
```

#### 设置服务器监控

使用腾讯云监控或第三方监控工具（如Prometheus + Grafana）监控服务器状态。

### 4. 安全维护

#### 定期更新系统

```bash
# 更新系统包
sudo apt update
sudo apt upgrade -y

# 重启系统（如需要）
sudo reboot
```

#### 防火墙配置

```bash
# 安装UFW
sudo apt install -y ufw

# 设置默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 允许SSH
sudo ufw allow ssh

# 允许HTTP和HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 允许WebSocket端口
sudo ufw allow 8080/tcp

# 启用防火墙
sudo ufw enable

# 查看状态
sudo ufw status
```

#### 安全检查清单

- 定期更改SSH密钥和密码
- 定期检查系统日志查找异常活动
- 确保MongoDB和Redis只监听本地接口或使用强密码保护
- 定期更新Node.js和npm包以修复安全漏洞
- 使用HTTPS确保通信加密
- 实施DDoS防护措施

## 扩展与优化

### 1. 性能优化

#### Node.js优化

```bash
# 设置Node.js环境变量
export NODE_ENV=production

# 优化V8引擎参数
export NODE_OPTIONS="--max-old-space-size=4096"
```

#### MongoDB优化

编辑MongoDB配置文件 `/etc/mongod.conf`：

```yaml
# 增加连接池大小
net:
  port: 27017
  bindIp: 127.0.0.1

# 优化WiredTiger缓存
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 1  # 根据服务器内存调整，通常为总内存的一半
```

#### Redis优化

编辑Redis配置文件 `/etc/redis/redis.conf`：

```
# 调整最大内存使用
maxmemory 1gb  # 根据服务器内存调整
maxmemory-policy allkeys-lru

# 优化持久化设置
save 900 1
save 300 10
save 60 10000
```

### 2. 扩展方案

当用户增长超过单台服务器容量时，可以考虑以下扩展方案：

#### 垂直扩展

- 升级服务器配置（增加CPU、内存、存储）
- 优化代码和数据库查询
- 实现更高效的缓存策略

#### 水平扩展

- 使用负载均衡器分发流量到多台服务器
- 将MongoDB和Redis拆分到独立服务器
- 实现数据库分片
- 使用Redis集群处理高并发

## 故障排除

### 1. 常见问题与解决方案

#### 应用无法启动

```bash
# 检查日志
pm2 logs mud-game

# 检查Node.js版本
node -v

# 检查依赖是否安装
npm install

# 检查配置文件
cat .env
```

#### 数据库连接失败

```bash
# 检查MongoDB服务状态
sudo systemctl status mongod

# 检查MongoDB日志
sudo cat /var/log/mongodb/mongod.log

# 检查MongoDB连接
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
```

#### WebSocket连接问题

```bash
# 检查Nginx配置
sudo nginx -t

# 检查防火墙设置
sudo ufw status

# 检查应用日志
pm2 logs mud-game
```

### 2. 紧急恢复流程

#### 数据库恢复

```bash
# 从备份恢复MongoDB
cd /var/backups/mongodb
tar -zxvf 20230101_030000.tar.gz
mongorestore --db mud_game 20230101_030000/mud_game

# 从备份恢复Redis
sudo systemctl stop redis
sudo cp /var/backups/redis/dump_20230101_033000.rdb /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo systemctl start redis
```

#### 应用回滚

```bash
# 回滚到之前的版本
cd /var/www/mud-game
git log --oneline  # 查看提交历史
git reset --hard <commit-hash>  # 回滚到指定提交
npm install --production
pm2 restart mud-game
```

## 结论

本部署文档提供了在单台腾讯云服务器上部署微信小游戏MUD量子叙事项目的完整指南。通过遵循这些步骤，单人开发者可以成功部署和维护游戏服务器，确保游戏的稳定运行和良好的用户体验。

随着用户规模的增长，可能需要考虑更高级的扩展方案，但对于初期阶段，单台服务器配置足以支持游戏的正常运行。定期的维护和监控对于确保系统的稳定性和安全性至关重要。