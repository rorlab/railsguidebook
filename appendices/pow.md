# 맥전용 랙서버 POW

[`POW`](http://pow.cx)는 [Basecamp](http://basecamp.com/)에서 만든 무료 맥전용 랙서버로 자체 HTTP 서버와 DNS 서버를 포함하고 있다.

맥 OSX에서 별다른 설정없이 1분이내에 로컬 웹서버를 구동하여 포트사용없이 여러개의 레일스 프로젝트를 서빙할 수 있다. (아래의 이미지를 클릭하면 사용법에 대한 동영상을 볼 수 있다.)

[![](http://pow.cx/images/logo-pow.png)](http://get.pow.cx/media/screencast.mov)


즉, `http://localhost:3000`과 같이 포트를 추가하지 않고 "프로젝트 폴더명" + ".dev"와 같은 도메인으로 해당 프로젝트에 접근할 수 있다.

### 초간단 설치

```bash
$ curl get.pow.cx | sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6887  100  6887    0     0  14944      0 --:--:-- --:--:-- --:--:-- 14939
*** Installing Pow 0.4.3...
*** Installing local configuration files...
/Users/hyo/Library/LaunchAgents/cx.pow.powd.plist
*** Installing system configuration files as root...
Password:
/etc/resolver/dev
/Library/LaunchDaemons/cx.pow.firewall.plist
*** Starting the Pow server...
*** Performing self-test...
*** Installed

For troubleshooting instructions, please see the Pow wiki:
https://github.com/37signals/pow/wiki/Troubleshooting

To uninstall Pow, `curl get.pow.cx/uninstall.sh | sh`
```

### 프로젝트 링크 생성하기

위에서 언급한 바와 같이 웹프로젝트명에 `.dev`를 붙인 URI로 연결하기 위해서는 아래와 같이 `~/.pow` 디렉토리로 이동하여 아래와 같이 프로젝트 절대경로에 대한 링크를 생성한다.

```bash
$ cd ~/.pow
$ link -s /Users/user-account/rcafe
$ ls -al
total 8
drwxr-xr-x  3 hyo  staff  102  5 21 08:13 ./
drwxr-xr-x  5 hyo  staff  170  5 21 06:45 ../
lrwxr-xr-x  1 hyo  staff   23  5 21 07:18 rcafe@ -> /Users/user-account/rcafe
```

### 브라우저에서 연결하기

터미널에서 `rails server`로 로컬서버를 구동하지 않고도 아래와 같이 웹어플리케이션에 연결된다.

```bash
$ open http://rcafe.dev
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/2014-05-21_08-17-53_zpsb286a308.png)


### 다른 컴퓨터에서 연결하기

동일한 LAN 환경에 있는 다른 컴퓨터에서 개발머신의 **Pow 가상호스트**에 접근할 수 있다. 이것은 모바일 디바이스나 윈도우 또는 리눅스 VM 상에서 프로젝트를 테스트하고자 할 때 유용하다.

`.dev` 도메인은 개발머신에서만 동작한다. 그러나 `.xip.io` 도메인을 사용하면 원격에서 **Pow 가상호스트**로 접근할 수 있다.

이를 위해서 개발머신의 LAN IP 주소를 프로젝트명과 `.xip.io` 사이에 추가하여 접속하면 된다. 모바일 디바이스나 윈도우/리눅스 가상머신에서 아래의 주소로 접속하면 된다.

http://rcafe.xxx.xxx.xx.xxx.xip.io



### Foreman과 함께 사용하기

로컬에서 레일스 서버와 함께 delayed_job과 같은 별개의 프로세스들을 동시에 실행해야 할 것 경우에는, [`foreman`](https://github.com/ddollar/foreman)을 사용하면 편리하다. 

[`여기`](https://gist.github.com/dre1080/3980988)를 참고하면 쉽게 설정할 수 있다. 이를 위해서는 먼저 로컬에 foreman이 설치되어 있어야 한다. 

1. 어플리케이션 루트 디렉토리에 `.powenv` 파일을 생성하고 아래와 같은 내용을 추가한다. 
```
# In your app's root.
# Make Pow!! export all the env variables contained in the .env file used by Foreman.
export $(cat .env)
```
2. 역시 어플리케이션 루트 디렉토리에 `.configuration.sh`파일을 생성하고 아래와 같은 내용을 추가한다. 

  ```
  # This should be run once only
  cd /path/to/myapp
  echo 7000 > .port # Whatever port your app server is using
  ln -s $PWD/.port ~/.pow/`basename $PWD`
  echo "port: 7000" > .foreman
 
  # Then start your app normally everytime with:
  # > foreman start
  # Voila! App will be available at http://myapp.dev
  ```

  여기에서 수정이 필요한 곳은 3군데다. `/path/to/myapp`를 실제 어플리케이션 디렉토리로 변경하고, 포트를 원하는 것으로 변경하고자 한다면 7000이란 숫자를 두군데에서 변경하면 된다. 
3. 이제 `foreman`을 아래와 같이 실행한다. 
```
$ foreman start
```
4. 다음은 브라우저에서 `http://{프로젝트명}.dev`로 접속해 본다. 

  
---

매뉴얼에 대한 자세한 내용을 원하면 http://pow.cx/manual.html 를 참고하기 바랍니다.




