# MariaDB Install on CentOS7

## Contents
1. MariaDB Repository 설정 
2. MariaDB 설치
3. MariaDB 설치 확인
4. 새로운 Data 디렉토리 생성 및 데이터 복사하기
5. MariaDB  Config file 수정
6. SELinux 보안 context 추가 및 서비스 시작
7. MariaDB 원격 접속
8. conf file 수정

## 1. MariaDB Repository 설정 
- 원활한 설치를 위하여 root로 접속
```
$ su
```
- MariaDB.repo 파일 유무 확인, 없으면 생성 후 수정
```
# ls /etc/yum.repos.d/
```
```
# touch /etc/yum.repos.d/MariaDB.repo
```
```
# vi /etc/yum.repos.d/MariaDB.repo
```
- MariaDB.repo
```
# MariaDB 10.4 CentOS repository list - created 2020-01-15 08:12 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1  
```
## 2. MariaDB 설치
```
# yum install -y MariaDB
```
## 3. MariaDB 설치 확인
```
# rpm -qa | grep MariaDB
```
## 4. 새로운 Data 디렉토리 생성 및 데이터 복사하기
(Data 디렉토리를 /home/data/mysql 로 한다는 가정 하에)
```
# mkdir /home/data/
# rsync -av /var/lib/mysql /home/data/
# chown -R mysql:mysql /home/data/mysql
```
## 5. MariaDB  Config file 수정
- 최적화 부분 참조
```
# vi /etc/my.cnf 
```
## 6. SELinux 보안 context 추가 및 서비스 시작
- MaraiDB 서버의 서비스 포트를 변경하려면, 먼저 SELinux 활성화 여부를 확인
```
# sestatus 
```
```
# semanage port -a -t mysqld_port_t -p tcp 16033 
# semanage port -l | grep mysqld_port_t 
```
- 방화벽 포트 설정
```
# firewall-cmd --permanent --zone=public --add-port=16033/tcp 
# firewall-cmd --reload 
# firewall-cmd --zone=public --list-all

public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eno1
  sources:
  services: dhcpv6-client ssh
  ports: 22/tcp 16033/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

```
- mariadb.service 수정
```
# vi /usr/lib/systemd/system/mariadb.service
```
- "=" 기호 뒤에 있는 정보를 삭제 후 저장
```
ProtectSystem=
ProtectHome=
```
- daemon reload 실행
```
# systemctl daemon-reload
```
- selinux Data 위치 설정
```
# semanage fcontext -a -t mysqld_db_t "/home/data/mysql(/.*)?"
# restorecon -R /home/data/mysql
# systemctl start mariadb
```
## 7. MariaDB 원격 접속 설정

Mysql 접속
```
# mysql -u root -p
```
```
MariaDB [(none)]> use mysql
```
- 권한 설정
```
MariaDB [(mysql)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.100.%' IDENTIFIED BY 'its@1234' WITH GRANT OPTION;
MariaDB [(mysql)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.101.%' IDENTIFIED BY 'its@1234' WITH GRANT OPTION;
MariaDB [(mysql)]> Flush privileges;
```

## 8. conf file 수정(포함된 파일 참조)
- /etc/my.cnf
- /etc/my.cnf.d/client.cnf
- /etc/my.cnf.d/mysql-clients.cnf
- /etc/my.cnf.d/server.cnf

> 수정할 부분만 작성됨.