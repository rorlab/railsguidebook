# 공동집필에 참여하기

### NPM 설치

npm v0.6.3 부터는 node.js 설치시에 함께 설치되기 때문에 별도로 npm을 설치할 필요가 없다. 따라서 시스템에 node.js가 설치되어 있지 않다면 [http://www.nodejs.org](http://www.nodejs.org) 를 참고하여 설치한다.

> #### \#\#\#\#Note::참고
> 한글로 작성된 설치 안내문을 원하면  [Nodejs-설치-및-개발환경-세팅하기](http://inspiredjw.tistory.com/entry/Nodejs-설치-및-개발환경-세팅하기)를 참고하기 바란다.

### 로컬 머신에 gitbook 설치하기

: 터미널 쉘에서 아래와 같이 명령을 실행한다.

```bash
$ npm install gitbook -g
```

### 로컬 머신에 gitbook editor 설치하기

: [리눅스/맥/윈도우용 다운로드](https://github.com/GitbookIO/editor/releases) 후 설치하면 된다.

### 가이드북 Github 저장소 Forking 하기

[Github](https://github.com)에 로그인한 상태에서 [rorlab/railsguidebook](https://github.com/rorlab/railsguidebook)로 접속한 후 우측 상단에 있는 `Fork` 버튼을 클릭한다.

![](/assets/2016-12-22_20-12-10_zpsxnmtoly0.png)

### 저장소 clone 하기

: github에서 본인 계정으로 Fork된 저장소를 clone한다.

```bash
$ git clone https://github.com/[user-account]/railsguidebook.git
$ cd railsgidebook
```

### 문서작업

[GitBook Editor](https://github.com/GitbookIO/editor)를 실행한 후 방금 전에 clone 받은 폴더를 `Import` 후 문서작업을 한다.

### 작업내용 확인

문서 작업을 종료한 후, 터미널 쉘에서 아래와 같이 명령을 실행하여 작업한 내용이 제대로 브라우저에서 보이는지를 확인한다. \(브라우저에서 [http://localhost:4000](http://localhost:4000) 주소로 접속한다.\)

```bash
$ gitbook serve
```

> ####Note::노트
> 이때 아래와 같은 에러가 발생하면, `$ ulimit -n 1200` 명령을 실행한 후 다시 실행하면 된다. \([참고문서](https://github.com/GitbookIO/gitbook/issues/214)\)

```bash
$ gitbook serve
Press CTRL+C to quit ...

Starting build ...
Successfuly built !

Starting server ...
Serving book on http://localhost:4000
Error: EMFILE: Too many opened files.
```

### 커밋 후 푸시하기

작업한 내용을 커밋하고 푸시한다.

```bash
$ git add .
$ git commit -m "작업내용을 기술"
$ git push
```

> **Info** 작업내용 중에 삭제된 파일이 있는 경우에는 `git add .` 대신에 `git add -A`와 같이 실행하여 삭제된 파일도 커밋에 반영되도록 한다.

### Pull Request 하기

에러가 없이 작성된 것이 확인되면 아래와 같이 본인의 `github` 계정 저장소에서 아래와 같이 `New pull request` 버튼을 클릭하면 본인의 커밋내용을 원조 저장소로 머지 요청을 하게 된다.\("나의 커밋을 pull해 주세요."\)

![](/assets/2016-12-22_21-03-58_zps6pg01sgt.png)

이상으로 간단하게 `Gitbook`으로 공동집필하는 방법을 소개했다.

