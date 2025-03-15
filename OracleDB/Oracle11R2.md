---

# 🚀 快速搭建 Oracle 11g R2 单实例数据库并配置开机自启

在本教程中，我们将演示如何在 **Oracle Linux** 上快速安装 **Oracle 11g R2 数据库** 并配置开机自启。通过这一简单的步骤，你将能够轻松部署 Oracle 数据库，并确保它在每次重启后自动启动。

## 📋 环境准备

- **操作系统**：Oracle Linux 7（推荐版本：OracleLinux-R7-U9-Server-x86_64-dvd.iso）
- **数据库版本**：Oracle Database 11g R2 (112040_Linux-x86-64)
- **硬件配置**：
  - 1 GB 内存
  - 1 个 CPU
  - 虚拟机环境（推荐 VMware 或 VirtualBox）
- **开放端口**：默认监听端口 **1521**

---

## 🔧 步骤一：安装 Oracle Linux 和进行网络配置

1. **安装 Oracle Linux 系统**：根据安装向导完成安装，并配置静态 IP 或 DHCP（根据需求）。
2. **更新系统并配置防火墙**：

```bash
yum -y update && yum -y upgrade
sudo firewall-cmd --add-port=1521/tcp --permanent
sudo firewall-cmd --reload
```

---

## ⚙️ 步骤二：安装 Oracle 环境预配置包

1. 安装 **oracle-rdbms-server-11gR2-preinstall** 包：

```bash
yum -y install oracle-rdbms-server-11gR2-preinstall.x86_64
```

2. 设置 Oracle 用户密码：

```bash
passwd oracle
```

---

## 📂 步骤三：创建目录并设置权限

```bash
mkdir -p /u01/app/
chown -R oracle:oinstall /u01/app/
chmod -R 775 /u01/app/
```

---

## 🔄 步骤四：上传并解压 Oracle 安装包

1. 使用 FTP 工具将 Oracle 安装包上传至 `/home/oracle` 目录。
2. 解压安装包：

```bash
yum -y install unzip
cd /home/oracle
unzip p13390677_112040_Linux-x86-64_1of7.zip
unzip p13390677_112040_Linux-x86-64_2of7.zip
```

---

## 📝 步骤五：创建响应文件

创建一个响应文件 `db_install.rsp`，并填写如下内容：

```bash
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_AND_CONFIG
ORACLE_HOSTNAME=your-hostname
INVENTORY_LOCATION=/u01/app/oraInventory
UNIX_GROUP_NAME=oinstall
SELECTED_LANGUAGES=zh_CN,en
ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oinstall
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=orcl
oracle.install.db.config.starterdb.SID=orcl
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.memoryLimit=256
oracle.install.db.config.starterdb.password.SYS=StrongPassword1
oracle.install.db.config.starterdb.password.SYSTEM=StrongPassword2
oracle.install.db.config.starterdb.password.DBSNMP=StrongPassword3
oracle.install.db.config.starterdb.password.SYSMAN=StrongPassword4
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.storageType=FILE_SYSTEM_STORAGE
oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=/u01/app/oracle/oradata
oracle.install.db.config.starterdb.fileSystemStorage.recoveryLocation=/u01/app/oracle/fast_recovery_area
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.control=DB_CONTROL
DECLINE_SECURITY_UPDATES=true
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
oracle.installer.autoupdates.option=SKIP_UPDATES
```

---

## 🔧 步骤六：静默安装 Oracle 数据库

1. 切换到 `oracle` 用户并运行安装命令：

```bash
su oracle
cd /home/oracle/database
./runInstaller -silent -responseFile /home/oracle/db_install.rsp -ignorePrereq
```

2. 安装过程中，系统会提示执行两个脚本：

```bash
/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/product/11.2.0/dbhome_1/root.sh
```

---

## 🌱 步骤七：配置环境变量

设置 Oracle 环境变量，确保系统正确运行：

```bash
su oracle
echo 'export ORACLE_BASE=/u01/app/oracle' >> ~/.bash_profile
echo 'export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1' >> ~/.bash_profile
echo 'export ORACLE_SID=orcl' >> ~/.bash_profile
echo 'export PATH=$ORACLE_HOME/bin:$PATH' >> ~/.bash_profile
source ~/.bash_profile
```

---

## 🚀 步骤八：配置开机自动启动

### 1. 修改 `/etc/oratab` 文件

编辑 `/etc/oratab`，将 `orcl:/u01/app/oracle/product/11.2.0/dbhome_1:N` 改为：

```bash
orcl:/u01/app/oracle/product/11.2.0/dbhome_1:Y
```

### 2. 创建启动脚本 `/etc/init.d/oracle`

使用 `root` 用户创建启动脚本，并编辑：

```bash
sudo vi /etc/init.d/oracle
```

脚本内容如下：

```bash
#!/bin/bash
# Oracle 自动启动脚本
# chkconfig: 345 99 10
# description: Oracle auto start-stop script

export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_SID=orcl
export PATH=$ORACLE_HOME/bin:$PATH

case "$1" in
  start)
    echo "Starting Oracle Listener..."
    su - oracle -c "lsnrctl start"
    echo "Starting Oracle Database..."
    su - oracle -c "dbstart $ORACLE_HOME"
    ;;
  stop)
    echo "Stopping Oracle Database..."
    su - oracle -c "dbshut $ORACLE_HOME"
    echo "Stopping Oracle Listener..."
    su - oracle -c "lsnrctl stop"
    ;;
  restart)
    $0 stop
    $0 start
    ;;
  status)
    su - oracle -c "lsnrctl status"
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit 1
    ;;
esac
exit 0
```

### 3. 设置脚本执行权限

```bash
sudo chmod +x /etc/init.d/oracle
```

### 4. 设置开机启动

```bash
sudo chkconfig --add oracle
sudo chkconfig oracle on
```

### 5. 启动与停止测试

启动 Oracle：

```bash
sudo /etc/init.d/oracle start
```

停止 Oracle：

```bash
sudo /etc/init.d/oracle stop
```

---

## 🌟 总结

通过本教程，你已经成功在 Oracle Linux 上搭建了 Oracle 11g R2 数据库并配置了开机自启。该简化的安装过程非常适合开发和测试环境，避免了复杂的配置，易于管理和维护。

---

如果本教程对你有所帮助，别忘了在 GitHub 上点赞、评论和分享！🎉 你还有其他问题或优化建议吗？欢迎留言，我们一起交流！
