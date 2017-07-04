# 윈도우 개발머신에 설치하기


: [`RailsFTW`](http://railsftw.bryanbibat.net)를 이용하여 한번에 설치하기


`RailsFTW`(Ruby on Rails **F**or **T**he **W**indows)는 윈도우즈 환경에만 설치가 가능한 레일스 인스톨러다.

### 장단점

레일스 개발을 위한 최소한의, 그러나 하나의 완전한 패키지로서 2014년 5월 20일 현재 `RailsInstaller`보다 더 높은 레일스 버전을 지원하는 것이 장점이다.

![](/assets/2014-05-20_12-39-34_zps11cc924b.png)

`RailsFTW`는 `윈도우즈 8`과는 충돌이 발생할 수도 있으며 `RubyInstaller DevKit`이 포함되어 있지 않기 때문에 `DevKit`을 필요로 하는 젬을 설치하기 위해서는 [`RubyInstaller` 웹사이트](http://rubyinstaller.org/add-ons/devkit/)에서 직접 다운로드 해야 한다는 단점을 가지고 있다.

### 설치방법

1. `RailsFTW` 웹사이트에서 최신 버전의 `RailsFTW 인스톨러`를 다운로드 받아 설치한다.
2. 끝이다. 정말.


### 참고사항

1. 루비 콘솔을 사용하기 위해서는 `시작 메뉴 > RailsFTW > Start Command Prompt with Ruby`를 실행한다. 다른 개발환경의 매뉴얼을 참고하고 있다면 레일스와 관련한 커맨드를 사용할 때 해당 콘솔을 사용하면 된다.
2. `Git`, `Vim`을 비롯한 "기본적인" 개발 관련 유틸리티는 1번의 콘솔에서 사용이 불가하므로 콘솔과는 별개로 설치해서 사용한다.
3. [`msysgit`](http://code.google.com/p/msysgit/)(윈도우즈용 git)을 별도로 설치하면 `git`을 사용할 수 있다.
4. 소스코드 에디터는 [`Sublime Text Editor`](http://www.sublimetext.com)를 사용하면 불편함이 없다.
5. [`Console2`](http://sourceforge.net/projects/console/)을 설치하여 사용하면 `cmd.exe`의 불편한 점을 다소 해소할 수 있다.
6. [`ImageMagick`](http://www.imagemagick.org/script/binary-releases.php#windows)을 다운로드 받아 설치하면 이미지 업로드시 이미지 작업을 할 수 있다.
7. [`Ghostscript`](http://www.ghostscript.com/download/gsdnld.html)를 다운로드 받아 설치하면 PDF 파일을 업로드시 쎔네일 이미지를 생성할 수 있다.

### 문제해결

* 혹시 `rake db:migrate` 할 때 타임존 데이터가 없다는 에러가 발생하면 Gemfile에 gem 'tzinfo-data', platforms: [:mingw, :mswin] 를 추가하고 config.ru 파일에 require 'tzinfo' 추가하면된다.

---

_**References:**_

1. https://www.facebook.com/l.php?u=https%3A%2F%2Fgithub.com%2Fleehankyeol&h=OAQEeXPnX



