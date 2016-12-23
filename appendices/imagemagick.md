# ImageMagick 설치하기

* `ImageMagick`은 이미지를 생성하고 수정하고 변환하는 시스템 툴이다.
* 대개는 서버에서 업로드된 이미지의 썸네일 버전을 생성하거나 다양한 변화를 주기 위해 사용한다.
* 공식 사이트 : https://www.imagemagick.org/script/index.php

### 맥 OSX에서 설치하기

: `homebrew`가 설치되어 있는 상태에서 아래와 같이 한 줄 명령으로 설치할 수 있다.

```
$ brew install ghostscript imagemagick
```

### 우분투에서 설치하기

아래와 같이 한 줄 명령으로 설치가 된다.

```
$ sudo apt-get install ghostscript imagemagick
```

### 윈도우에서 설치하기

[ImageMagick 윈도우 바이너리 릴리스](http://www.imagemagick.org/script/binary-releases.php)를 [다운로드](http://www.imagemagick.org/download/binaries/ImageMagick-6.8.9-1-Q16-x64-dll.exe) 받아 설치한다. 그리고, [Ghostscript for Windows](http://www.ghostscript.com/download/gsdnld.html)를 다운로드([32비트용](http://downloads.ghostscript.com/public/gs914w32.exe)|[64비트용](http://downloads.ghostscript.com/public/gs914w64.exe)) 받아 설치한다.

> **Note** [Ghostscript](http://www.ghostscript.com)는 PDF 파일에 대한 썸네일 이미지를 생성하기 위해 사용된다.
