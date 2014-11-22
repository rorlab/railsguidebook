# SSHKit README

![SSHKit 이미지](https://raw.github.com/leehambley/sshkit/master/assets/images/logo.png)

**SSHKit**는 하나 또는 그 이상의 서버에서 구조화된 방식으로 명령을 실해아기 위한 툴킷이다.

###어떻게 동작하는가?


일반적인 사용법은 아래와 같다.

``` ruby
require 'sshkit/dsl'

on %w{1.example.com 2.example.com}, in: :sequence, wait: 5 do
  within "/opt/sites/example.com" do
    as :deploy  do
      with rails_env: :production do
        rake   "assets:precompile"
        runner "S3::Sync.notify"
        execute "node", "socket_server.js"
      end
    end
  end
end
```

SSHKit는 매우 낮은 수준이지만 편리한 API 를 제공해 주는데, `as()`, `within()`, `with()` 는 어떤 순서라도 상관없이 중첩할 수 있고 반복할 수 있으며 스택에 푸시할 수도 있다.

이와 같이 블록 내에서 사용할 때, `as()`와 `within()`는 포함하는 블록을 체크하여 보호할 것이다.

`within()`의 경우에는, 해당 디렉토리가 존재하지 않을 에러을 발생하고, `as()`에 대해서는, `sudo su -<user> whoami` 를 호출하여 성공여부를 체크하고 실패할 경우 에러를 발생한다.

디렉토리 유무 체크는 아래와 같이 실행된다.

``` sh
if test ! -d <directory>; then echo "Directory doesn't exist" 2>&1; false; fi
```

그리고 사용자 변경 테스트는 다음과 같이 실행된다.

``` sh
if ! sudo su <user> -c whoami > /dev/null; then echo "Can't switch user" 2>&1; false; fi
```

디폴트 상태에서 0 이외의 상태 값을 가지는 모든 명령은 에러를 발생한다(물론 이러한 설정을 변경할 수 있다). 메시지 내용은 처리과정에서 stdout으로 작성된 모든 것을 포함한다. `1>&2` 는 echo의 표준 출력을 표준 에러 채널로 리디렉트 시켜 주어 에러 발생시 메시지를 볼 수 있게 된다.

`execute(:rails, "runner", ...)` 와 `execute(:rake, ...)` 로 확장되는 `runner()`와 rake()` 와 같은 헬퍼 메소드들은 루비와 레일스 어플리케이션을 위한 편리한 헬퍼들입니다.

###병행(Parallel)


`on()` 호출시 `in: :sequence` 옵션을 주의해서 사용한다.

``` ruby
on(in: :parallel) { ... }
on(in: :sequence, wait: 5) { ... }
on(in: :groups, limit: 2, wait: 5) { ... }
```

디폴트는 `in: :parallel` 상태로 실행하며 제한이 없다. 400대의 서버가 있을 경우에는 문제가 발생할 수 있기 때문에 `groups` 또는 `sequence` 상태로 실행하도록 변경하는 것이 더 좋을 것이다.

groups는, 하나의 (Git) 리소스에 대해서 너무 많은 접속을 하여 DDOS 공격을 원치 않는 경우(대량으로 Git 체크아웃을 하는)를 방지하기 위해서 고안되었다.

순차적인 실행(sequence)은 다를 사용 예 중에서도 (순차적으로) restart를 하고자 할 때 사용한다.

###동기화(Synchronisation)


`on()` 블록은 동기화의 단위이다. 하나의 `on()` 블록은 모든 서버가 작업을 완료한 후에 반환하게 된다.

예를 들면,

``` ruby
all_servers = %w{one.example.com two.example.com three.example.com}
site_dir    = '/opt/sites/example.com'

# 백업 task를 시뮬레이션 해 보자.
# 어떤 서버들은 다른 것에 비해 시간이 더 걸릴 수 있다는 가정을 한다.
on all_servers do |host|
  in site_dir do
    execute :tar, '-czf', "backup-#{host.hostname}.tar.gz", 'current'
    # 이것은 다음과 같이 실행될 것이다. "/usr/bin/env tar -czf backup-one.example.com.tar.gz current"
  end
end

# 이제 이러한 백업을 가지고 무언가를 할 수 있다. 이미 모든 백업들이 존재한다고 알고 있는 상태에서이다.
# (모든 tar 명령은 성공상태로 종료되었거나 하나라도 실패한 경우 예외가 발생했을 것이다.
on all_servers do |host|
  in site_dir do
    backup_filename = "backup-#{host.hostname}.tar.gz"
    target_filename = "backups/#{Time.now.utc.iso8601}/#{host.hostname}.tar.gz"
    puts capture(:s3cmd, 'put', backup_filename, target_filename)
  end
end
```

###명령 맵(Command Map)


프로그램 상 접근하는 SSH 세션은 인터액티브(interactive) 세션과 동일한 환경 변수를 가지지 않는다는 문제가 종종 있다.

`$PATH` 경로 상에 존재할 것으로 기대하는 실행파일을 호출할 때 문제가 종종 발생한다. dotfile이나 다른 환경 구성이 없는 상황에서는, `$PATH`는 제대로 설정 되지 못하기 때문에 실행파일들을 해당 위치에서 발견하지 못할 수 있다.

이러한 문제를 해결하기 위해서 `with()` 헬퍼는 변수들로 구성된 해시를 취해서 환경에서 사용할 수 있도록 해 준다.

``` ruby
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute :ruby, '--version'
end
```

이것은 다음과 같이 실행될 것이다.

``` sh
( PATH=/usr/local/bin/rbenv/shims:$PATH /usr/bin/env ruby --version )
```

이와는 대조적으로, 아래의 스크립트는 명령을 전혀 변경하지 못할 것이다.

``` ruby
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute 'ruby --version'
end
```

이것은 다음과 같이 실행될 것이다.

``` sh
ruby --version
```

(이와 같은 현상은 때때로 혼란스럽지만, 대부분이 쉘 이스케이핑(shell escaping)과 관계가 있는데, whitespace가 명령에 포함된 경우나 개행문자가 있는 경우, 입력한 내용으로 정확한 쉘 명령을 작성한 방법이 없기 때문이다.)

*따라서 이런 경우를 대비해서 명령 맵을 사용하는 것을 종종 더 선호하기도 한다.*

*Command* 객체를 생성할 때, 디폴트로 명령 맵을 사용할 수 있게 된다.

명령 맵은 구성 객체 상에 존재하고 원칙적으로 매우 간단하다. 디폴트 키를 factory 블록에 명시한 해시 구조를 가진다. 예를 들면,

``` sh
puts SSHKit.config.command_map[:ruby]
# => /usr/bin/env ruby
```

환경 설정을 확실히 하기 위해 `/usr/bin/env`가 모든 명령 앞에 붙게 된다. 이것은 단순하게 `ruby`를 실행할 때 일어나는 것이지만, 명확히 함으로써 사람들이 문서를 찾아 보기를 바란다.

각 명령에 대한 해시 맵을 변경할 수 있다.

``` ruby
SSHKit.config.command_map[:rake] = "/usr/local/rbenv/shims/rake"
puts SSHKit.config.command_map[:rake]
# => /usr/local/rbenv/shims/rake
```

또 다른 방법은 명령 앞에 붙일 다른 명령을 추가하는 것이다.

``` ruby
SSHKit.config.command_map.prefix[:rake].push("bundle exec")
puts SSHKit.config.command_map[:rake]
# => bundle exec rake

SSHKit.config.command_map.prefix[:rake].unshift("/usr/local/rbenv/bin exec")
puts SSHKit.config.command_map[:rake]
# => /usr/local/rbenv/bin exec bundle exec rake
```

명령 맵을 완전히 새로운 것으로 변경할 수 있는데, 이것은 현명하지 못한 일이지만, 가능한 일이다. 예를 들면,

``` ruby
SSHKit.config.command_map = Hash.new do |hash, command|
  hash[command] = "/usr/local/rbenv/shims/#{command}"
end
```

이것은 해당 디렉토리에 실행 파일을 제공해 주지 않았던 어떤 명령도 호출할 수 없게 되지만 때로는 이러한 상황이 필요할 수 있다.

주의사항 : 명령 맵에서 해당 키를 찾기 전에 *Command*  객체는 첫번째 인수를 심볼화 할 것이기 때문에, 모든 키는 심볼이어야 한다.

###출력물을 처리하기


![콘솔 그림](https://raw.github.com/leehambley/sshkit/master/assets/images/example_output.png)

디폴로, 콘솔 출력물 포맷은 `:pretty`로 설정되어 있다.

``` ruby
SSHKit.config.format = :pretty
```

그러나, 출력을 최소한만 보이도록 원할 경우 `:dot` 포맷으로 지정하면 명령 실행의 성패에 따라 빨간색이나 초록색으로 표시될 것이다.

포맷을 사용하지 않고 직접 $stdout으로 출력하기 위해서는 아래와 같이 지정할 수 있다.

``` ruby
SSHKit.config.output = $stdout
```

###출력물의 정보 표시(Output Verbosity)


디폴트 상태에서는 `capture()` 와 `test()` 호출시 로그가 남지 않는다. 이 명령들은 종종 백엔드 task에서 환경 설정을 체크하기 위해서 사용된다.
`Logger::DEBUG`의 `Command` 인스턴스에 verbosity 옵션을 지정할 수 있다. 디폴트 구성은 `SSHKit.config.output_verbosity=` 로 변경할 수 있고 디폴트는 `Logger::INFO` 이다.

현재 `Logger::WARN`, `ERROR`, `FATAL` 는 사용하지 않는다.

###연결 풀링(Connection Pooling)


SSHKit는, 모든 `on()` 블록에 대해 새로운 SSH 연결을 시도하는 것에 대한 비용을 줄이기 위해서, 단순한 연결 풀(디폴트로 활성화됨)을 사용한다. 용도와 네트워크 상황에 따라서, 상당한 시간 절약을 할 수 있다. 한 테스트에서는, 기본적인 `cap deploy` 명령이 SSHKit의 최근 버전에 추가된 연결 풀 덕분에 15-20초 정도 더 빠르게 실행되었다.

연결 상태를 활성 상태로 유지하기 위해서, 기존의 풀링된 연결은 30초이상 사용되지 않을 경우 새로운 연결로 대체될 것이다. 이와 같은 타임아웃은 아래와 같이 변경할 수 있다.

``` ruby
SSHKit::Backend::Netssh.pool.idle_timeout = 60 # seconds
```

연결 풀이 문제를 일으킨다고 생각될 때는, idle_timeout 값을 0으로 설정하므로써 연결 풀 기능을 해제할 수도 있다.

``` ruby
SSHKit::Backend::Netssh.pool.idle_timeout = 0 # disabled
```

###알려진 문제들


* 느리거나 타이아웃된 연결을 처리하지 못함
* 느리거나 중단된 원격 명령을 처리하지 못함
* ~~백그라운드 작업 처리 기능이 없음~~
* 환경 처리기능 없음(sshkit는 이에 대한 걱정을 할 필요없다)
* ~~`host` 객체에 임의의 속성을 추가할 수 없음(서버에 `roles`을 저장하거나 `on()` 블록에서 유용하게 사용할 수 있는 메타데이타 등)~~
* ~~로그나 경고 메시지를 보여주는 기능이 없음(로그 메시지를 출력으로 넘기는 것이 작동한다)  하나의 로그 객체는, 전역적으로 사용할 수 있어야 하며, 로그 메시지에 대한 관심을 가지는 포맷터들이 인식할 수 있는 LogMessage 형 객체를 발송할 것이다.~~
* ~~verbosity를 제어할 수 없음. 명령은 자신에 대한 `Logger::LEVEL` 을 가져야 한다. 사용자가 생성한 것은 고수준의 정보를 보여줘야 하고, `as()` 와 `within()` 로부터 권한 체크를 위해 자동으로 생성되는 명령들은 저수준의 정보를 보여줘야 한다.~~
* ~~`execute()` 류 명령들이 0가 아닌 종료 상태에 대해서 에러를 발생할지 여부를 결정해야 함. 아마도 비슷한 이름을 가진 !(bang) 메소드군은 에러를 발생해야 한다. (아마도 `test()`는 에러 발생시키지 않고 `execute()`를 실행하는 방법이고 `execute()` 류 명령들은 항상 에러를 발생해야 한다.)~~
* ~~`SSHKit.config.formatter = :pretty` 라고 설정할 수 있고, 메소드 setter가 `SSHKit::config.output`을 기존 출력 스트림을 래핑(wrapping)하는 정확한 포맷터 클래스 인스턴스로 업데이트할 수 있게 한다면 멋질 것이다.~~
* 내부적으로 디버깅하기 위한 "trace" 레벨이 없음. 디버그 레벨은 클라이어트 측 디버깅을 위해서 예비해 두어야 하고, 네트워크 연결, 종료, 타임아웃에 대한 로그를 남기 위해서 내부적으로 trace(int -1)를 사용해야 한다.
* 파일 업로드나 다운로드를 위한 메소드가 없음. 또는 원격 파일로 문자열을 저장하거나 가져오는 메소드가 없음.
* 네트워크 연결은 중단할 수 없음. 백엔드 abstract 클래스는 비어있지만 다르게 방법으로 구현하여 변경할 수 있는 cleanup 메소드를 포함해야 한다.
* ~~연결 풀링 기능이 없음.  NetSSH 백엔드의 `connection`  메소드는, 연결 팩토리에서 연결 객체들을 찾아 볼 수 있도록 쉽게 변경할 수 있어야 하고, 이로써, 여러 개의 `on()` 블록을 실행할 때 0.5초 정도 시간을 절약할 수 있게 된다.~~
* 문서화(YARD 형태)가 필요하다.
* 모든 명령을 알려진 쉘로 래핑(wrapping)할 수 있어야 함. 즉, `execute('uptime')`은 일관된 쉘 실행 환경을 유지하기 위해서 `sh -c 'uptime'`으로 변환되어야 한다.
* ~~`Host.new('user@ip:port')`와 같이 호출하는 것을 수용하는 적당한 호스트 파서(parser)가 없음. 이것은 `user@hostname:port`을 디코딩하지만 IP 주소는 디코딩하지 못할 것이다.~~
* Net::SSH가 `IOError`를 발생(인증 실패와 같이)할 때 에러를 잡아 낼 수 있어야 하고, ConnectionFailed와 같은 에러를 다시 발생할 수 있어야 한다.

----

_**References:**_

1. [SSHKit Readme](https://github.com/capistrano/sshkit/blob/master/README.md)

