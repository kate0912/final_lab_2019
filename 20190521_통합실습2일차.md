# Install CM 5.15
## 들어가기 전
```
ssh -i skcc.pem centos@15.164.146.95
ssh -i skcc.pem centos@15.164.151.13
ssh -i skcc.pem centos@15.164.86.198
ssh -i skcc.pem centos@15.164.5.68
ssh -i skcc.pem centos@15.164.63.69
```

## 1. Enable user / password login for each of the 5 nodes 
### a. Create a password for user “centos” 
### b. Modify sshd_config to allow password login 
### c. Restart the sshd.service 
```
sudo passwd centos

sudo vi /etc/ssh/sshd_config
>> PasswordAuthentication yes

sudo systemctl restart sshd.service
```
## 2. Setup /etc/hosts with the following information for each of the 5 hosts 
### a. Private_IP FQDN Shortcut 
```
sudo vi /etc/hosts

172.31.7.227    master01.cdhcluster.com mn1
172.31.12.148   util01.cdhcluster.com   util01
172.31.8.146    data01.cdhcluster.com   dn1
172.31.10.234   data02.cdhcluster.com   dn2
172.31.3.22     data03.cdhcluster.com   dn3
```
## 3. Change the hostname as necessary to the FQDN that you setup above 
### a. Reboot the host 
```
hostname -f
sudo hostnamectl set-hostname util01.cdhcluster.com
hostname -f

init 6
```
## 4. Install JDK on each of the hosts (you may choose to install on just the host where you will install CM and use the Wizard later to install on the rest)
```
sudo yum install -y openjdk java-1.8.0-openjdk-devel.x86_64 
```
## 5. On the host that you will install CM: 
### a. Configure the repository for CM 5.15.2 
```
-- Configure repository
sudo yum install -y wget
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo \
-P /etc/yum.repos.d/

sudo vi /etc/yum.repos.d/cloudera-manager.repo
>> baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.15.2/

-- change the baseurl within cloudera-manager.repo to fit the version you want to install
baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.7.4/
for example: https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.14.4/

sudo rpm --import \
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera

java -version
```
### b. Install CM 
```
sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```
### c. Install and enable Maria DB (or a DB of your choice) 
    i. Don’t forget to secure your DB installation 
```
sudo yum install -y mariadb-server
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo /usr/bin/mysql_secure_installation
>> 나머지 Y (disable root login: n)
```
### d. Install the mysql connector or mariadb connector 
```
sudo yum install -y wget
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz

tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar

cd /usr/share/java/
sudo yum install mysql-connector-java
>> Y
```
### e. Create the necessary users and dbs in your database 
    i. Grant them the necessary rights 
    f. Setup the CM database 
    g. Start the CM server and prepare to install the cluster through the CM GUI installation process
```
-- Create the databases and users in MariaDB
mysql -u root -p

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE rmon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rmon.* TO 'rmon-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie-user'@'%' IDENTIFIED BY 'somepassword';

FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;

-- Setup the CM database
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user somepassword
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
sudo systemctl start cloudera-scm-server
```

# Install CDH 5.15.2 Cluster
http://15.164.146.95:7180/cmf/login
>> admin / admin

-- 선택
enterprise 체험판, trial 

--  호스토 지정
mn1, util01, dn1, dn2, dn3

-- 리포지토리 설치
디폴트 답

-- JDK 설치 
체크하지 않고

-- ssh 로그인 인증 
유저/ 패스워드

-- 클러스터 설치

-- 기본 서비스 설치
>> https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_ig_host_allocations.html#host_role_assignments 
>> 3 - 10 Worker Hosts without High Availability
HDFS

YARN

ZOOKEEPER


172.31.7.227    master01.cdhcluster.com mn1    >> util1 CDH, Maria DB : SNN, Balancer,   
172.31.12.148   util01.cdhcluster.com   util01 >> mn1   : NN, CM
172.31.8.146    data01.cdhcluster.com   dn1     :
172.31.10.234   data02.cdhcluster.com   dn2     :
172.31.3.22     data03.cdhcluster.com   dn3     :



-- 데이터베이스 설정
active monior: amon/amon-user/somepassword
report monior:rmon/rmon-user/somepassword



1. Specify hosts for your CDH cluster installation 
2. You may choose to use a common user / password 
3. Allow CM to install CM Agent on each of the nodes 
4. Allow CM to download, distribute and activate CDH on each of the nodes 
5. Choose the type of installation you want a. We will require Flume, Hive, Impala later on 
6. Assign roles to each of the hosts: Please refer here for hints.



------------------------------------
yum install unzip

hue: training / training >> 처음 들어간 유저가 admin

hdfs 디렉토리 자동으로 만듦
sudo training traning

user modidify -add G wheel group ??? >> training 그룹 wheel 그룹에 넣기

all.zip 파일 unzip >> Mysql, 작업 노드(Data 노드)

Mysql, 작업노드 안에서 돌리기
setup.sh 안 돌거, 권한 문제 등으로 인해서
(accounts 테이블, 환경 설정 해줌)
-------------------------------------
-- 계정 확인
cat /etc/passwd | grep training

sudo useradd training

sudo passwd training

sudo usermod -aG wheel training
