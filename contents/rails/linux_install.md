# 리눅스 개발머신에 설치하기

## 우분투 14.04 LTS (Trusty Tahr)의 경우

[gorail.com 사이트 참조](https://gorails.com/setup/ubuntu/14.04)

### Ruby 설치 (2.1.2)

Ruby 설치를 위해 필요한 라이브러리 등을 우선 설치합니다.

``` bash
$ sudo apt-get update
$ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties
```

설치 방법은 rbenv나 rvm을 이용하거나, 혹은 소스로 직접 설치하는 방법이 있습니다. 공식적으로 rbenv를 이용할 것을 권장하고 있습니다.

#### rbenv 를 이용하는 방법

rbenv를 이용해서 ruby를 설치하는 것은 매우 간단합니다. 우선 rbenv를 설치하고, ruby-build를 하면 됩니다.

``` bash
$ cd
$ git clone git://github.com/sstephenson/rbenv.git .rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
$ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
$ exec $SHELL

$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
$ exec $SHELL

$ rbenv install 2.1.2
$ rbenv global 2.1.2
$ ruby -v
```

마지막 작업은 각 패키지들의 문서들을 설치하지 말라고 지정하는 것입니다.

``` bash
$ echo "gem: --no-ri --no-rdoc" > ~/.gemrc
```

#### rvm 을 이용하는 방법

``` bash
$ sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
$ curl -L https://get.rvm.io | bash -s stable
$ source ~/.rvm/scripts/rvm
$ echo "source ~/.rvm/scripts/rvm" >> ~/.bashrc
$ rvm install 2.1.2
$ rvm use 2.1.2 --default
$ ruby -v
```

마지막 작업은 각 패키지들의 문서들을 설치하지 말라고 지정하는 것입니다.

``` bash
$ echo "gem: --no-ri --no-rdoc" > ~/.gemrc
```


#### 소스로 직접 설치하는 방법

``` bash
$ cd
$ wget http://ftp.ruby-lang.org/pub/ruby/2.1/ruby-2.1.2.tar.gz
$ tar -xzvf ruby-2.1.2.tar.gz
$ cd ruby-2.1.2/
$ ./configure
$ make
$ sudo make install
$ ruby -v
```

마지막 작업은 역시 각 패키지들의 문서들을 설치하지 말라고 지정하는 것입니다.

``` bash
$ echo "gem: --no-ri --no-rdoc" > ~/.gemrc
```

### Git 설정

본 가이드의 [`Git 설치하기`](git.html) 참조


### Rails 설치

Rails는 상당히 많은 프로그램에 의존하기 때문에 NodeJS와 같은 Javascript runtime을 설치해야 합니다. 이는 Rails에서 Coffeescript와 the Asset Pipeline을 사용할 수 있도록 해주고, javascript를 최소화해 더 빠른 제작환경을 만들어줍니다.

NodeJS를 설치하기 위해서는 PPA repository를 추가해야 합니다.

``` bash
$ sudo add-apt-repository ppa:chris-lea/node.js
$ sudo apt-get update
$ sudo apt-get install nodejs
```

그리고 rails를 설치합니다.
``` bash
$ gem install rails
```

rbenv를 사용하고 있다면 rails를 사용하기 위해서는 다음의 명령어를 실행해야 합니다.

``` bash
$ rbenv rehash
```

rails가 제대로 설치되었다면 rails -v 명령어로 버전을 확인하면 제대로 설치되었는지 알 수 있습니다.

``` bash
$ rails -v
# Rails 4.1.1
```

만일 다른 결과가 나온다면, 설치가 제대로 되지 않았다는 의미입니다.
