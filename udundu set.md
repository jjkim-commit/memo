> Robot Install
Rpi Ubuntu 22.04 64

> Update
$ ip address show
$ sudo apt-get update
$ sudo apt install net-tools
$ sudo apt install openssh-server -y
$ sudo apt update
$ sudo apt install unzip

> LAN2 추가 Install Mode 및 RTC 셋팅 (맨 아래 ALL부분)
$ sudo nano /boot/firmware/config.txt

# daminrobot lan2 config
# Added in Dec 6 2023 for ubuntu 10.4 server W5500 SPI ethernet protocol
dtoverlay=anyspi,spi0-0,dev="w5500",speed=30000000
dtoverlay=w5500

# daminrobot rtc config
# Added in Feb 3 2026
dtparam=i2c_arm=on                         <--- 상단에 이미 되어 있을수 있음
dtoverlay=i2c-rtc,pcf85063a,i2c_csi_dsi

$ sudo reboot
$ sudo timedatectl set-ntp false
$ sudo timedatectl set-time "2026-02-03 13:06:00"
$ sudo hwclock --systohc

$ sudo reboot

> RTC 적용 확인
$ sudo dmesg | grep rtc








> (참고사항) UART 추가 설정
출처: https://periar.tistory.com/230 [Hslee:티스토리]
$ dtoverlay -a | grep uart
UART0 의 경우 Debug Console 로 사용되고
UART1 의 경우 Bluetooth 에 연결되어있다.

$ sudo dmesg | grep tty    <-- enable 활성화 확인

$ sudo nano /boot/firmware/config.txt
# daminrobot uart config
enable_uart=1
dtoverlay=uart2
dtoverlay=uart3
#dtoverlay=uart4          <-- damin 확장랜 충돌됨
dtoverlay=uart5

> (필요시) Create User
$ sudo adduser daminrobot
$ sudo usermod -aG sudo daminrobot

> 컴퓨터 이름 변경
$ sudo hostnamectl set-hostname dmbot-ctrl
$ (확인) sudo nano /etc/hostname
dmbot-ctrl

$ sudo nano /etc/hosts      # --->  127.0.1.1 dmbot-ctrl 
127.0.0.1       localhost dmbot-ctrl
127.0.1.1       dmbot-ctrl

$ sudo reboot

> /dev/ttyUSB0 를 접근 권한 일반 user 도 사용 할 수 있도록 설정
$ sudo usermod -a -G dialout $USER
$ ls -al /dev/ttyA*         <--- 확인용
/dev/ttyACM0   <-- MCU
/dev/ttyAMA0


> 컴파일 패키지 설치
$ sudo apt-get update -y
$ sudo apt install linux-headers-generic build-essential git -y
$ sudo apt install build-essential nghttp2 libnghttp2-dev libssl-dev -y

   
> Maria DB 설치
$ sudo apt update
$ sudo apt upgrade
$ sudo apt -y install mariadb-server mariadb-client
$ sudo mariadb-secure-installation

PS) 데이타 베이스 패스워드 제외한 모두 엔터를 입력 

$ mysql -u root -p
show databases;
create database dmbot;
create user 'dmbot'@'127.0.0.1' identified by 'qwer****';
create user 'dmbot'@'%' identified by 'qwer****';
grant all privileges on dmbot.* to 'dmbot'@'127.0.0.1';
grant all privileges on dmbot.* to 'dmbot'@'%';
flush privileges;
show grants for 'dmbot'@'127.0.0.1';
show grants for 'dmbot'@'%';

실행후 exit로 데이터 베이스에서 나감

$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
[mysqld]
port = 33306
bind-address = 0.0.0.0
$ sudo systemctl restart mariadb

(참고) ufw 방화벽 사용시
$ sudo ufw allow 33306

> mosquitto
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install -y mosquitto
$ sudo systemctl restart mosquitto
$ sudo systemctl start mosquitto
$ sudo apt install -y mosquitto-clients
$ sudo nano /etc/mosquitto/mosquitto.conf        --> 아래 내용을 맨 하단에 입력
listener 31883 0.0.0.0
protocol mqtt
listener 39001
protocol websockets
allow_anonymous true

$ sudo service mosquitto restart

-- (ufw 방화벽 사용시)
$ sudo ufw allow 31883

-- test
$ mosquitto -version   ## 버전확인하는 방법 명령어 수정 필요
$ mosquitto_sub -t /dmbot/test/# -h localhost -p 31883
$ mosquitto_pub -t /dmbot/test -m "hi hello" -h localhost -p 31883
cmd> mosquitto_pub -t /dmbot/test -m "hi hello" -h 10.10.30.62 -p 31883
cmd> mosquitto_sub -t /dmbot/test/# -h 10.10.30.62 -p 31883


> ffmpeg 
$ sudo apt install ffmpeg

> 참고 os 비트 확인 방법
$ uname -m
32bit: armv7l / armv6l
64bit: aarch64

> ARM Rpi Dotnet 7 Install
(설치참고) https://learn.microsoft.com/ko-kr/dotnet/iot/deployment
- .net 모듈중 일부 필요 및 .net 설치
$ sudo apt install libcanberra-gtk-module libcanberra-gtk3-module
$ sudo apt-get install curl
$ curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel STS
$ echo 'export DOTNET_ROOT=$HOME/.dotnet' >> ~/.bashrc
$ echo 'export PATH=$PATH:$HOME/.dotnet' >> ~/.bashrc
$ source ~/.bashrc

- (참고) Visual Studio 빌드시 arm
dotnet publish --runtime linux-arm64 --self-contained
32비트 OS를 사용하는 경우 런타임을 linux-arm 대상으로 지정해야 합니다.

$ 설치확인
  dotnet --list-runtimes

> (참고 Mono 필요시) Rpi ARM mono 설치 test 못함 (x-window 가 설치 되어 있어야함)
$ sudo apt update
==> 설치 안함 sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common
$ sudo apt-key adv --keyserver http://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
$ sudo apt-add-repository 'deb https://download.mono-project.com/repo/ubuntu stable-focal main'
$ sudo apt install mono-complete -y
$ mono --version

> 업그레이드
$ sudo apt update
$ sudo apt upgrade

> (필요시) 클린
$sudo apt clean && sudo apt autoclean

> 참고 os 비트 확인 방법
$ uname -m
32bit: armv7l / armv6l
64bit: aarch64

========= 여기까지만 우선 설치 =========
> nodejs 설치
$ sudo apt update
$ sudo apt install curl -y
$ sudo apt-get install -y curl software-properties-common
$ sudo curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

> nodejs & npm 설치
$ sudo apt install nodejs -y 
$ node -v
$ sudo rm /etc/apt/sources.list.d/nodesource.list*
$ sudo apt-get install npm -y
$ npm -v

> (참고) npm 설치 오류시 조치
  sudo apt install aptitude
  sudo aptitude install npm -y

> (참고) nodejs 설치중 오류시 조치
$ sudo dpkg --remove --force-remove-reinstreq libnode-dev
$ sudo dpkg --remove --force-remove-reinstreq libnode72:amd64
$ sudo apt remove nodejs-doc

> node-red 설치
$ sudo npm install -g --unsafe-perm node-red node-red-admin
$ node-red    <-- 테스트
