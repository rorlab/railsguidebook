# 맥 OSX 개발머신에 설치하기

최신 OSX인 매버릭스(10.9)를 기준으로 설명한다. 맥을 구입하면 기본적으로 ruby가 내장되어 있다. 즉, 시스템 루비를 바로 사용할 수 있게 된다. 설치된 루비 버전을 알기 위해 아래와 같은 명령을 실행해 본다. 시스템 루비는 2.0 버전으로 설치되어 있다.

```sh
$ ruby -v
ruby 2.0
```

그러나 [몇가지 이유](http://robots.thoughtbot.com/psa-do-not-use-system-ruby)로해서 시스템 루비는 가능하면 사용하지 않는 것이 좋다.

대신에 루비 버전 관리자를 설치하여 루비를 버전별로 사용할 수 있도록 한다. 이를 위해서 `rbenv`을 이용하는 것이 추천된다. [rbenv 설치](rbenv.html)를 참고하여 설치한 후, 루비의 최신 버전인 2.1.2를 설치해 보자. 우선 `rbenv` 플러그인인  [`ruby-build`](https://github.com/sstephenson/ruby-build) 목록에 설치가능한 루비 버전이 어떤 것이 있는지 알아보자.

```sh
$ rbenv install list
```

만약, 목록 중에 2.1.2 버전이 없으면 `ruby-build` 플러그인을 업데이트할 필요가 있다. `homebrew`를 이용하여 `ruby-build`를 설치했다면 아래와 같이 업데이트할 수 있다.

```sh
$ brew update ruby-build
```

이제 다시 `rbenv install list` 명령으로 2.1.2 버전이 등록되었는지를 확인하고 있다면 아래와 같이 루비 2.1.2 버전을 rbenv으로 설치한다.

```sh
$ rbenv install 2.1.2
```

이제 레일스를 설치해 보자.

```sh
$ gem install rails
```

그리고 아래와 같이 설치를 마무리 한다.

```sh
$ rbenv rehash
```

아마도 가장 최선버전의 레일스가 설치될 것이다. 2014년 5월 16일 현재 4.1.1 버전이 설치될 것이다. 아래와 같이 설치된 레일스 버전을 확인할 수 있다.

```sh
$ rails -v
Rails 4.1.1
```
