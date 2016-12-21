# 우분투 16.04 서버 세팅하기 (Virtual Box)

: 레일스 애플리케이션을 배포하기 위해서는 서버의 준비가 필요가 하다. 실제 운영 서버에 배포 테스트하는 것 보다는 **Virtual Box**와 같은 가상머신에 서버를 설치해 놓고 이를 이용하는 것이 여러가지로 유용하다. 이를 위해서 **Virtual Box**를 이용하여 **Ubuntu 서버**의 최신 버전인 **16.04.1 LTS (Xenial Xerus)**을 설치하고 웹서비스를 위한 서버 환경을 셋업하는 과정을 소개한다.

> **이미지 다운로드** : [http://releases.ubuntu.com/16.04/](http://releases.ubuntu.com/16.04/)를 방문하여 우분투 서버 버전(16.04.1 LTS (Xenial Xerus))을 다운로드 받는다.

로컬 머신에 각자의 OS에 맞는 **Virtual Box**가 설치되어 있다고 가정하고 아래와 같이 진행한다.

> **Virtual Box 다운로드**: https://www.virtualbox.org/wiki/Downloads

---

### 가상머신 생성

---


#### 1. 아래의 창에서 `1번`을 클릭하여 새로운 가상머신을 생성한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-07-11_zps3e73006b.png)

#### 2. 운영체제 이름을 설정한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-09-08_zpsd8c4cef1.png)

#### 3. 이름에 ubuntu라고 입력하면 자동으로 버전이 Ubuntu (64 bit)로 지정된다. 이름은 원하는 것으로 변경할 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-09-08_zpsd8c4cef1.png)

#### 4. 메모리는 1,024 MB이상이 권장된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-10-23_zpsb958b248.png)

#### 5. 하드 드라이브는 기본설정으로 두번째 항목으로 만들기 한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-10-52_zps6a953a70.png)

#### 6. 하드 드라이브 파일 종류는 기본값인 첫번째 항목으로 계속한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-11-05_zps90efff64.png)

#### 7. 물리적 하드 드라이브에 저장에서는 기본값인 첫번째 항목(동적할당)으로 계속한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-11-22_zps7ea91b5a.png)

#### 8. 파일 위치 및 크기는 기본값인 8.00 GB 상태로 만들기한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-11-57_zps98742c18.png)

#### 9. 가상머신 설정

: 방금 생성된 `1번` 항목에서 마우스의 오른쪽 버튼을 클릭하여...

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-12-16_zps7f52e6c7.png)

설정 항목을 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-13-41_zpsf2d15e8b.png)

#### 10. 네트워크 1번 어댑터 탭에서 `브리지 어댑터`를 선택하고 확인 버튼을 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-14-39_zps19417f89.png)

#### 11. 가상머신 시작하기

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-15-08_zpsa1a0eba0.png)

#### 12. 다운로드 받은 서버 이미지 선택하기

: 미리 다운로드 받아 놓은 우분투 서버 이미지 파일을 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-40-56_zpsbe3b9ab3.png)

----

### 게스트 서버의 설치

----


#### 13. 서버 설치 시작

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_17-41-16_zpsc5a91cd7.png)

#### 14. 사용할 언어를 선택하는 화면에서 기본값인 `English`가 반전된 상태에서 엔터키를 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-30-10_zps537967a4.png)

#### 15. 우분투 서버 설치 항목을 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-30-48_zps7f1d2bc2.png)

#### 16. 설치 진행시 사용할 언어를 선택하는 화면에서도 기본값인 `English`가 반전된 상태에서 엔터키를 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-31-06_zps23f878b3.png)

#### 17. 타임존 설정을 위한 지역을 선택하는 화면에서는 `other` 항목을 선택하고 엔터키를 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-31-30_zps912204f9.png)

이어지는 화면에서는 `Asia`를 선택하고 엔터키를 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-31-46_zpsc4d8a97f.png)

그리고 `Korea, Republic of`을 선택한 후 엔터키를 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-31-58_zps10e914fd.png)

#### 18. 로케일 설정 화면에서는 기본값인 `United States - en_US.UTF-8`이 반전된 상태에서 엔터키를 클릭한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-32-15_zpsdac493d4.png)

#### 19. 키보드 검색 화면에서는 기본값인 `<No>`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-32-30_zpsca25f5d6.png)

#### 20. 키보드 레이아웃 선택 화면(Country of origin)에서는 기본값인 `English (US)`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-32-41_zps14173eca.png)

#### 21. 키보드 레이아웃도 기본값인 `English (US)`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-32-53_zps5f9742d7.png)

#### 22. 컴포넌트 진행과정

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-33-21_zpscb4461c1.png)

#### 23. 가상머신에 사용한 호스트이름을 정한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-34-08_zpsb41a467c.png)

#### 24. 유저 이름을 입력한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-34-34_zpsc5e8d6e7.png)

#### 25. 계정 이름을 입력한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-35-04_zps38bc58e5.png)

#### 26. 암호를 입력한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-35-18_zps09f8cfd4.png)

동일한 암호를 한번 더 입력한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-35-33_zps5c73206a.png)

#### 27. 홈 디렉토리 암호화 여부는 기본값인  `<No>`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-35-45_zps08364769.png)

#### 28. 하드디스크 파티션 방법은 기본값인 `use entire disk and set up LVM`을 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-36-59_zps20604641.png)

#### 29. 파티션할 디스크 선택은 기본값을 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-37-38_zpsb8cabb4c.png)

#### 30. 변경 내용을 저장 화면에서 `<Yes>`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-37-59_zpsf221e67e.png)

#### 31. 디스크 볼륨도 기본값으로 선택하고 계속한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-38-36_zps25eb0cb4.png)

#### 32. 변경 내용을 저장하는 화면에서 `<Yes>`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-38-50_zps4f265967.png)

#### 33. 시스템 설치 과정

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-39-12_zpscf02672b.png)

#### 34. Proxy 정보를 지정하지 않고 계속해도 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-40-00_zpse1bda256.png)

#### 35. 설치 진행과정 (1)

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-40-26_zpse6d1e429.png)

설치 진행과정 (2)

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-41-32_zps8b563417.png)

#### 36. 시스템 업데이트는 수동으로 지정한다. 기본값으로 진행한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-41-40_zps4d77070f.png)

#### 37. 설치할 소프트웨어는 아래와 같이 선택한다(스페이스바로 선택).

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-19_08-40-59_zpspx48ymyk.png)

#### 38. 설치 진행과정

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-42-25_zpsd7732abf.png)

#### 39. GRUB 부트로더 설치는 `<Yes>`를 선택한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-43-04_zps038926fe.png)

#### 40. 설치완료

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rorla/2014-10-09_20-43-19_zpsb36c70a7.png)

#### 41. 시스템 부팅 시작 화면

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-19_09-24-15_zpsdye0pmgu.png)

#### 42. 로그인 화면

아래와 같은 로그인 화면이 나타나면 등록한 유저 계정명과 암호를 이용하여 로그인한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-19_08-47-46_zpsqiowrj09.png)

##### 43. 로그인 후 화면

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-19_08-50-52_zpsdrxzbgbv.png)

#### 44. 가상 머신의 ip 주소

프롬프트에서 `ifconfig` 명령을 실행하면 아래와 같이 자신(가상머신 - 게스트 머신 : Ubuntu 16.04 서버)의 ip를 알 수 있다. 잘 기억해 두었다가 로컬머신(호스트머신)의 터미널에서 ssh로 가상머신으로 접속시 사용한다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-19_08-54-09_zpsfg0dg4su.png)

#### 45. 로컬 머신에서 ssh로 접속시도

지금부터는 로컬 머신에서 별도의 터미널 창을 띄워 위에서 찾은 가상머신 ip 주소로 접속한다.
접속한 게스트 PC(가상머신)의 첫화면은 아래와 같다. 

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rcafe/2016-12-19_08-56-26_zpssm5juy3w.png)

## 서버 웹운영 환경 설정

이제부터는 [우분투 16.04 서버 세팅하기](/appendices/ubuntu16server.md) 챕터의 내용 중 `서버 설치과정`을 보고 따라하면 된다. 

중복되지만 아래에 삽입해 두었다.

#### 서버 설치과정

```bash
# 시스템 업데이트
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get update

# 시스템에 한국어 설치
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install language-pack-ko language-pack-ko-base -y
ubuntu@ubuntu-VirtualBox:~$ sudo vi /etc/default/locale
LANG="ko_KR.UTF-8"
LANGUAGE="ko_KR:ko:en_US:en"
ubuntu@ubuntu-VirtualBox:~$ sudo dpkg-reconfigure locales

# Build-essential 설치
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get -y install git curl build-essential openssl libssl-dev libreadline-dev python-software-properties python g++ make

# Nginx 서버 설치
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install -y nginx

# MySQL 서버 설치
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

# Imagemagick 이미지 툴 설치
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get -y install libmagickwand-dev imagemagick

# Ghostscript 설치
ubuntu@ubuntu-VirtualBox:~$ sudo apt-get install ghostscript

# Nodejs 설치
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



