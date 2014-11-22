# SSHKit 사용 예제


###다른 계정으로 명령 실행하기


``` ruby
on hosts do |host|
  as 'www-data' do
    puts capture(:whoami)
  end
end
```

###디폴트 환경 변수로 실행하기


``` ruby
SSHKit.config.default_env = { path: '/usr/local/libexec/bin:$PATH' }
on hosts do |host|
  puts capture(:env)
end
```

###다른 디렉토리에서 명령 실행하기


``` ruby
on hosts do |host|
  within '/var/log' do
    puts capture(:head, '-n5', 'messages')
  end
end
```

###특정 환경 변수로 명령 실행하기


``` ruby
on hosts do |host|
  with rack_env: :test do
    puts capture("env | grep RACK_ENV")
  end
end
```

###로깅 메소드를 이용하여 출력하기


``` ruby
on hosts do |host|
  f = '/some/file'
  if test("[ -d #{f} ]")
    execute :touch, f
  else
    info "#{f} already exists on #{host}!"
  end
end
```

`SSHKit.config.output_vervosity` 의 현재 로그 레벨에 따라 `debug()`, `info()`, `warn()`, `error()`, `fatal()` 메소드를 사용할 수 있다.

###다른 계정으로 다른 디렉토리에서 명령 실행하기


``` ruby
on hosts do |host|
  as 'www-data' do
    within '/var/log' do
      puts capture(:whoami)
      puts capture(:pwd)
    end
  end
end
```

이것은 아래와 같은 결과를 보여 준다.

``` sh
www-data
/var/log
```

주의사항 : 이 예제는 약간 오해의 소지가 있다. 즉, `www-data` 계정은 쉘이 정의되어 있지 않아서 이 계정으로 변경할 수 없다.

###디스크로부터 파일 업로그하기


``` ruby
on hosts do |host|
  upload! '/config/database.yml', '/opt/my_project/shared/database.yml'
end
```

주의사항 : `upload!()` 메소드는 `within()`, `as()` 등의 값을 받지 않는다. 이 문제는  라이브러리가 성숙해 짐에 따라 해결될 것이지만 현재로서는 아직 그대로다.

###스트림으로부터 파일 업로드하기


``` ruby
on hosts do |host|
  file = File.open('/config/database.yml')
  io   = StringIO.new(....)
  upload! file, '/opt/my_project/shared/database.yml'
  upload! io,   '/opt/my_project/shared/io.io.io'
end
```

IO 스트리밍하는 것은 "cat" 하기 보다는 뭔가를 업로드하는데 유용하다. 예를 들면,

``` ruby
on hosts do |host|
  contents = StringIO.new('ALL ALL = (ALL) NOPASSWD: ALL')
  upload! contents, '/etc/sudoers.d/yolo'
end
```

이것은 "echo(:cat, '...?...', '> /etc/sudoers.d/yolo')"와 같이 정확한 이스케이핑 순서를 기술해야 하는 수고를 덜어 준다.

주의사항 : `upload!()` 메소드는 `within()`, `as()` 등의 값을 받지 않는다. 이 문제는  라이브러리가 성숙해 짐에 따라 해결될 것이지만 현재로서는 아직 그대로다.

###디렉토리내의 파일을 업로드하기


``` ruby
on hosts do |host|
  upload! '.', '/tmp/mypwd', recursive: true
end
```

이 경우에 `recursive: true` 옵션은 `Net::{SCP, SFTP}`에서 사용가능한 옵션과 같은 것이다.

###전역(global) SSH 옵션 설정하기


이 옵션들은 호스트마다 옵션을 달리 지정할 수 있다.

``` ruby
Netssh.configure do |ssh|
  ssh.connection_timeout = 30
  ssh.ssh_options = {
    keys: %w(/home/user/.ssh/id_rsa),
    forward_agent: false,
    auth_methods: %w(publickey password)
  }
end
```

###다른 group ID로 명령 실행하기


``` ruby
on hosts do |host|
  as user: 'www-data', group: 'project-group' do
    within '/var/log' do
      execute :touch, 'somefile'
      execute :ls, '-l'
    end
  end
end
```

위의 설정에서,  생성된 파일은 `www-data` 계정과 `project-group` 그룹이 소유하게 된다.

`umask` 구성 옵션과 함께 사용하면, 로그인을 공유하지 않은 상태에서 팀 멤버간에 배포 스크립트를 쉽게 공유할 수 있다.

###중첩된 디렉토리에서 작성하기


``` ruby
on hosts do
  within "/var" do
    puts capture(:pwd)
    within :log do
      puts capture(:pwd)
    end
  end
end
```

이것은 아래와 같이 출력된다.

``` sh
/var/
/var/log
```

디렉토리 경로는 `File.join()`를 이용해서 조합된다. 이 메소드는 경로 앞뒤에 오는 슬래시에 신경쓰지 않고도 각 부분을 정확하게 연결해서 조합해 준다. 이것은 `File.join()`이 코드를 실행하는 머신에서 동작하기 때문에 오해의 여지가 있다. 만약 윈도우즈 운영환경에서 실행된다면 명령을 받는 머신에 따라 경로가 부정확하게 조합될 수 있다.

###백그라운드에서 task 수행하기


``` ruby
on hosts do
  within '/opt/sites/example.com' do
    background :rails, :server
  end
end
```

이것은 `nohup /usr/bin/env rails server > /dev/null &`와 같이 실행하는데, 레일스 프로세스를 백그라운드에서 실행하도록 하여 파일 시스템을 nohup 로그 파일로 더럽히지 않게 된다.

[역주] `nohup` - 리눅스, 유닉스에서 쉘스크립트파일(*.sh)을 데몬형태로 실행시키는 프로그램

주의사항 : `background()` 메소드에 `sleep 5` 문자열을 넘겨 주면 제대로 동작하지 않을 것이다. 이것은 명령을 처리하는 규칙에 따른 것이어서, `background(:sleep, "5")`와 같이(즉,  sleep이 명령이고, 5는 인수) 호출해야 한다.

더 주의해야 할 사항 : background() task가 어떤 상황에서 특정 명령을 `nohup ... &`로 감싸면, SSH 세션이 종료되어도 프로그램이 멈출 것이다.

###host 블록에 대해서 신경쓰지 않기


``` ruby
on hosts do
  # |host| 블록 인수는 옵션이다.
  # 블록 인수로 넘기지 않으면 블록내에서는 nil 값을 가지게 된다.
end
```

###모든 출력을 `/dev/null` 로 리디렉트하기


``` ruby
SSHKit.config.output = File.open('/dev/null')
```

###매우 간단한 formatter 클래스 구현하기


``` ruby
class MyFormatter < SSHKit::Formatter::Abstract
  def write(obj)
    case obj.is_a? SSHKit::Command
      # Do something here, see the SSHKit::Command documentation
    end
  end
end

SSHKit.config.output = MyFormatter.new($stdout)
SSHKit.config.output = MyFormatter.new(SSHKit.config.output)
SSHKit.config.output = MyFormatter.new(File.open('log/deploy.log', 'wb'))
```


###특정 호스트에 대한 비밀번호 지정하기


``` ruby
host = SSHKit::Host.new('user@example.com')
host.password = "hackme"

on host do |host|
  puts capture(:echo, "I don't care about security!")
end
```





###뭔가 잘 못 됐을 때 실행 후 에러 발생하기


``` ruby
on hosts do |host|
  execute!(:echo, '"Example Message!" 1>&2; false')
end
```

이것은 `#message` "Example Message!" 와 함께 'SSHKit::Command:Failed` 예외를 발생할 것이고 명령은 취소될 것이다.




###에러를 발생하지 않은 채 실패(fail)할 수 있는 테스트하기 또는 명령 실행하기


``` ruby
on hosts do |host|
  if test "[ -d /opt/sites ]"
    within "/opt/sites" do
      execute :git, :pull
    end
  else
    execute :git, :clone, 'some-repository', '/opt/sites'
  end
end
```

`test()` 명령는 `execute()`와 동일하게 동작하지만 0가 아닌 값으로 명령이 종료할 때 false 값을 반환한다(`man 1 test`가 하듯이). 논리값을 반환하기 때문에 블록내에서 제어 흐름의 방향을 지정할 때 사용될 수 있다.



###호스트 property에 따라 각 호스트마다 다른 작업 하기


``` ruby
host1 = SSHKit::Host.new 'user@example.com'
host2 = SSHKit::Host.new 'user@example.org'

on hosts do |host|
  target = "/var/www/sites/"
  if host.hostname =~ /org/
    target += "dotorg"
  else
    target += "dotcom"
  end
  execute! :git, :clone, "git@git.#{host.hostname}", target
end
```


###가장 쉽게 호스트에 연결하기


``` ruby
on 'example.com' do |host|
  execute :uptime
end
```

이것은 `example.com` 호스트명을 `SSHKit::Host` 객체로 변환해서 해당 호스트에 대한 정확한 구성 상태를 가져온다.



###명령 맵핑 없이 명령 실행하기


호출하려는 명령이 스페이스 문자를 포함할 경우 맵핑되지 않을 것이다.

``` ruby
Command.new(:git, :push, :origin, :master).to_s
# => /usr/bin/env git push origin master
# (also: execute(:git, :push, :origin, :master)

Command.new("git push origin master").to_s
# => git push origin master
# (also: execute("git push origin master"))
```

이것은 ( `if`와 `test`와 같은) 쉘 내장 명령을 접근할 때 사용할 수 있다.



###heredoc 와 함께 명령 실행하기


위의 기능을 확장한 것으로 아래와 같이 명령을 작성할 수 있다.

``` ruby
c = Command.new <<-EOCOMMAND
  if test -d /var/log
  then echo "Directory Exists"
  fi
EOCOMMAND
c.to_s
# => if test -d /var/log; then echo "Directory Exists; fi
# (also: execute <<- EOCOMMAND........))
```

주의사항 : 스크립트를 한줄로 표시하는 로직은 세련되어 보이지 않지만 모든 테스트에서 제대로 동작한다. 핵심 사항은 `if` 가 `/usr/bin/env if`로 매핑되지 않는다는 것인데, 이것은 문법상의 오류를 유발하지 않는다.


###Rake 사용하기


`Rakefile`에 아래와 같은 것을 둘 수 있다.

``` ruby
require 'sshkit/dsl'

SSHKit.config.command_map[:rake] = "./bin/rake"

desc "Deploy the site, pulls from Git, migrate the db and precompile assets, then restart Passenger."
task :deploy do
  on "example.com" do |host|
    within "/opt/sites/example.com" do
      execute :git, :pull
      execute :bundle, :install, '--deployment'
      execute :rake, 'db:migrate'
      execute :rake, 'assets:precompile'
      execute :touch, 'tmp/restart.txt'
    end
  end
end
```


###DSL 없이 사용하기


*Coordinator* 가 모든 호스트를 *Host* 객체로 변환해 줄 것이다.

``` ruby
Coordinator.new("one.example.com", SSHKit::Host.new('two.example.com')).each in: :sequence do
  puts capture :uptime
end
```

`./lib/sshkit/dsl.rb` 파일을 찾고할 수 있는 데, 여기에서 위와 거의 똑같은 코드를 볼 수 있지만 `on()` 메소드에 대해서 구현한 것이다.


###Host properties 속성 이용하기


`v0.0.6`부터 구현됨

``` ruby
servers = %w{one.example.com two.example.com
             three.example.com four.example.com}.collect do |s|
  h = SSHKit::Host.new(s)
  if s.match /(one|two)/
    h.properties.roles = [:web]
  else
    h.properties.roles = [:app]
  end
end

on servers do |host|
  if host.properties.roles.include?(:web)
    # 웹서버에 관련된 작업을 함.
  elsif host.properties.roles.include?(:app)
    # 어플리케이션 서버에 관련된 작업을 함.
  end
end
```

`SSHKit::Host#properties`는 [`OpenStruct`](http://ruby-doc.org/stdlib-1.9.3/libdoc/ostruct/rdoc/OpenStruct.html)이며 어쨌던 유효성 검증이 되지 않기 때문에 이에 관해서는 개발자에게 달려 있거나 라이브러리가 이 메카니즘에 의미나 규칙을 추가해 주어야 한다.


###로컬 명령 실행하기


`on`을 `run_locally`로 대체한다.

``` ruby
run_locally do
  within '/tmp' do
    execute :whoami
  end
end
```

----

_**References:**_

1. [Usage Examples of SSHKit](https://github.com/capistrano/sshkit/blob/master/EXAMPLES.md)


