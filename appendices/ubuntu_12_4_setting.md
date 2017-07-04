# 우분투 12.04 서버 환경 구축하기

로컬머신에 가상머신 툴인 오라클사의 `VM VirtualBox`를 설치하고 [VirtualBoxes – Free VirtualBox® Images](http://sourceforge.net/projects/virtualboximage/files/Ubuntu%20Linux/12.04/ubuntu_12.04-x86.7z)를 다운로드 받아 우분투 12.04 버전을 설치한다.

다운로드 받은 파일 7-zip 파일로 압축되어 있어서 [7-zip.org](http://www.7-zip.org/download.html)를 방문하여 프로그램을 다운로드 받아 설치하고 압축해제하면 된다.

압축해제하면 `ubuntu_12.04.vbox`와 `ubuntu_12.04.vdi` 파일 두 개가 나타난다. 이미 `VirtualBox`가 설치되어 있기 때문에 `ubuntu_12.04.vbox` 파일을 더블 클릭하면 자동으로 `VirualBox`에 로드된다. 시작버튼을 클릭할 때 확장팩을 설치하라는 메시지가 나타나면 [다운로드](http://download.virtualbox.org/virtualbox/4.3.12/Oracle_VM_VirtualBox_Extension_Pack-4.3.12-93733.vbox-extpack) 후 마우스로 더블 클릭하면 자동으로 설치된다.

그리고 설정에서 네트워크의 어댑터를 하나 더 추가하여 `호스트 전용 어댑터`로지정한다.

![](/assets/2014-07-10_08-54-46_zps7f056bb5.png)

디폴트로 등록되어 있는 관리자 계정은 `ubuntu`이고 비밀번호는 `reverse`이다.

![](/assets/2014-07-10_11-16-59_zpsccf5916d.png)

### 서버셋팅

방금 `VirtualBox`로 설치한 우분투 서버에 `ubuntu` 계정으로 로그인한다. 그리고 터미널을 열고 아래와 같이 `ssh` 서버를 설치한다.

```bash
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install openssh-server -y
```
그리고 할당된 `ip` 를 알기 위해서 아래와 같이 명령을 실행한다.

```bash
ubuntu@ubuntu-VirtualBox:~$ ip addr
```

![](/assets/2014-07-10_10-19-20_zps33fba946.png)


이제 로컬 터미널을 열고 아래와 같이 우분투 가상 서버에 `ssh`로 접속한다.

```bash
$ ssh ubuntu@ubuntu.vb
```

> **Info** 위에서 사용한 도메인 `ubuntu.vb` 는 가상도메인으로  로컬 머신의 `/etc/hosts` 파일에 `192.168.59.103` 을 `ubuntu.vb`로 지정해 놓았다.


#### 서버 설치과정

```bash
# System update
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get update

# To add a language to Ubuntu using the Command line:
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install language-pack-ko language-pack-ko-base -y
ubuntu@ubuntu-VirtualBox:~$ sudo vi /etc/default/locale
LANG="ko_KR.UTF-8"
LANGUAGE="ko_KR:ko:en_US:en"
ubuntu@ubuntu-VirtualBox:~$ sudo dpkg-reconfigure locales

# Build-essential
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get -y install git curl build-essential openssl libssl-dev python-software-properties python g++ make

# Nginx
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install -y nginx

# MySQL
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

# Imagemagick
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get -y install libmagickwand-dev imagemagick

# Nodejs
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get -y install nodejs
```

#### 배포용 계정 만들기

```bash
ubuntu@ubuntu-VirtualBox:~$ sudo adduser deployer
```
위의 명령으로 deployer 계정과 그룹이 생성되고 deployer 사용자는 deployer 그룹에 속하게 된다. 그리고 /home/deployer/ 디렉토리가 계정 홈디렉토리로 생성된다.

이제 deployer 계정에 루트권한을 주기 위해서 admin 그룹으로 등록한다.

```bash
ubuntu@ubuntu-VirtualBox:~$ sudo addgroup admin
ubuntu@ubuntu-VirtualBox:~$ sudo usermod -a -G admin deployer
```

_**References:**_

1. [OpenSSH Server](https://help.ubuntu.com/10.04/serverguide/openssh-server.html)
2. [How to SSH to a VirtualBox guest externally through a host?](http://stackoverflow.com/questions/5906441/how-to-ssh-to-a-virtualbox-guest-externally-through-a-host)
3. [How to SSH into a VirtualBox Linux guest from your host machine](http://community.ulyaoth.net/threads/how-to-ssh-into-a-virtualbox-linux-guest-from-your-host-machine.4/)
4. [Ubuntu: How to Change the Computer Name](http://www.tech-recipes.com/rx/2732/ubuntu_how_to_change_computer_name/)
5. [Error message when I run sudo: unable to resolve host (none)](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none)
6. [Fixing “No such file or directory” locale errors](http://backdrift.org/fixing-no-such-file-or-directory-locale-errors)
7. [ubuntu 사용자 계정 root 권한 주기](http://uiandwe.tistory.com/911)

