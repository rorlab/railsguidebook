# Capistrano 3를 이용한 레일스 프로젝트의 배포

레일스 프로젝트를 배포할 때는 Capistrano를 이용하는 것이 거의 표준처럼 알고 있다. Capistrano는 최근 버전 3가 릴리스되어 완전히 새로운 배포 로직으로, 기존의 레시피를 다시 작성해야 할 상황인 듯하다.

Capistrano 2에서는 전용 DSL이 있었지만, 버전 3에서는 이 부분을 걷어 내고 Rake DSL로 대체했다.

따라서 이 글에서는 버전 3에서 변경된 내용을 알아 보고 실제 배포를 위한 스크립트를 다시 작성해 보도록 하겠다.

###Capistrano 3의 릴리스 발표


> 2013년 6월 1일 [Capistrano 3가 공식적으로 릴리스](http://capistranorb.com/2013/06/01/release-announcement.html) 되었다.
아래에는 그 발표 내용을 요약 정리한다.

5년만에 있는 Capistrano의 메인 릴리스이다. 이러한 시간 간격이 발생한 이유는 여러가지가 있지만 많은 사람들이 Capistrano을 이용해서 배포 작업을 하고 있고, 실제로 배포시 서비스가 중단되는 시간이 문제가 되고 있다. 그 동안 Capistrano의 중요한 부분에 대해서 기능 변경을 했다면 많은 사이트들이 서비스를 할 수 없었을 것이다. 지금까지는 이러한 문제점을 감수하고라도 업그레이드를 단행할 정도로 시기적으로 무르익지 않았다고 생각했다.

루비 1.9만 가지고 고집하는 것은 역사적으로나 도움이 되지 않았고, Bundler가 많이 퍼져 있어서 특정 젬을 특정 버전으로 고정하는 것은 의미없는 일이 되어 버렸다. 루비 생태계의 다른 툴을 사용하면 수많은 사람들이 의존하고 있는 특정 툴에 중대한 변경을 하는 것이 더 쉬워졌다.

###디자인 목적


이번 릴리스에는 몇가지 목표가 있다. 나열된 순서는 의미가 없다.

* 자체 DSL 제거 : Rake, Sake, Thor와 같은 훌륭한 DSL 대안들이 이미 널리 사용되고 있다.
* 더 훌륭한 모듈화 : 레일스 커뮤니티 밖의 사람들은 Capistrano의 모법 사례로부터 도움을 받을 수 있도록, 레일스 커뮤니티 사람들은 필요로 하는 컴포넌트(데이터베이스 마이그레이션, asset pipeline 등)에 대한 지원을 선별적으로 사용할 수 있도록 했다.
* 보다 쉬운 디버깅 작업 : Capistrano에 관련된 많은 문제는, rvm, rbenv, nvm와 같은 환경 관리자들을 궂이 언급하지 않더라도 PTY 대비 non-TTY 환경에 관련된 환경 문제를 둘러싼 이상한 현상과 로그인하는 쉘과 로그인이 필요없는 쉘들로 부터 발생한다.
* 속도 : 많은 환경에서 배포 속도는 중요한 요소라고 알고 있다. 레일스가 Asset Pipeline을 도입한 이후, 이전에 5초 걸렸던 배포가 5분정도 걸리는 현상이 드물지 않다. 사실 이러한 문제는 대부분의 Capistrano의 통제 밖의 일이지만, 평행성(parallelism), 배포 재시작과 같은 지원이 향상되어 모든 것이 이전보다 빨리졌고 실행을 빠르게 유지하는 것이 더 쉬워졌다.
* 응용성 : 항상 Capistrano는 시스템 공급(system provisioning)이 어려운 툴로 되어 있어서, 대개 서버들은 Chef, Puppet 등과 같은 것으로 더 잘 셋팅을 하고 있다. 이것에 대해서는 여전히 동의하고 있지만, Capistrano의 새로운 기능들은 이런 툴과의 통합에 신경을 많이 쏟았다.

###버전 3에서 누락된 사항


더 나아기기 전에 버전 3에서 아직 구현하지 것들을 짧게 나열하는 것은 의미가 있다.

* SSH 게이트웨이 지원 : SSH 게이트웨이 지원은 버전 3에서 아직 구현하지 않았다. 빠른 시일내에 구현되기를 바란다. 이에 대한 직접적인 필요성을 느끼지 못하기 때문에, 구현할 의도로 테스트해 볼 방법이 아직 없다.
* Mercurial, Subversion, CVS 지원 : 다른 것들과는 호환이 되지 않지만, 그동안 놀라울 정도로 깔끔하게 Git SCM을 구현할 수 있었기 때문에 이것들은 제외되어 왔다.  최소공통분모를 고집하는 것을 깨기를 원하기 때문에, 각자가 선택한 소스 관리 툴을 이용해서 빠르게 배포하는 것에 대한 모범 사례에 대해 기여하거나 전문성을 공유하는데 관심이 있는 사람을 적극적으로 찾고 있다.
* **HOSTFILER, ROLEFILTER** 등등 : 이것들은 환경 변수를 사용하는 것에 관해서 좋지 못한 디자인으로 고질적이라고 생각했기 때문에 없애 버렸다. 이것들은 CLI 상에서 `cap`에 넘겨 주는 플래그로, `Capistrano::Applciation` 루비 클래스에 지정할 수 있는 옵션으로 다시 돌아 올 것이다.
* SHELL : 쉘은 더 깔끔한 구현을 위해서 잠시 제거했다. 내부적으로 가지고 놀만한 것을 찾았지만 더 나은 readline 지원이 필요하고, 서버에 따라 작동이 제대로 되지 않을 때 대비할 수 있는 방법이 필요하다.
* Cold Deploy : `cap deploy:cold`는 정말 오래된 컴포넌트인데, (실행되고 있지 않은 워커를 시작하는) cold 상태로 배포하는 `script/spinner` 시대로 거슬러 올라 간다. (기존의 워커 풀을 재시작하는, 재미없죠!) warm 시스템 배포는 달랐다. 대체로 이런 것들은 사라졌고 이제 `deploy:cold`도 사라졌다. setup을 찾아서 호출하고 seed하며 기타 다른 Rake task를 과장없이 수행하는것이 모든 경우에서 안전하다. 그렇게 하는 것이 우리가 취하는 접근방식이어야 한다. 해당 서버에 수행하는 task는 같은 결과를 보여야 하며 두번 호출하더라고 그대로여야 한다.

###새로 추가된 기능


####Rake 통합

Capistrano를 Rake 응용 어플리케이션으로 구현했다.

Rake::Application을 서브클래싱해서 만들었다. (서브어플리케이션)

코드 몇 줄로 처음부터 구현할 수 있기 때문에, copy 전략을 없앴다.

Dependency Resolution 방식을 원칙으로 한다.

``` ruby
task :notify do
  this_release_tag = sh("git describe --abbrev=0 --tags")
  last_ten_commits = sh("git log #{this_release_tag}~10..#{this_release_tag}")
  Mail.deliver do
    to "team@example.com"
    subject "Releasing #{this_release_tag} Now!"
    body last_ten_commits
  end
end

namespace :deploy
  task default: :notify
end
```

마지막 코드 세줄에서, deploy:default 태스크는 notify 태스크에 의존하게 된다. 즉, notify 수행 후에 default 태스크를 수행한다.


####내장 Stage 지원

버전 2에서는 stage 지원을 내장하고 있지 않아서 capistrano-ext 젬을 사용해서 해결했다.

버전 3에서는 stage 지원을 내장하고 있어서 설치 단계에서 디폴트로 두개의 stage를 생성해 준다.

staging과 production

이 외에도 추가하고 싶은 경우에는 config/deply/____.rb 으로 파일을 추가하면 된다.

다른 방법으로는 STAGES 라는 환경 변수를 이용해서 설치시에 명령에 추가하면 된다. stage 이름은 콤마로 구분한다.

``` sh
$ cap install STAGES=staging,production,ci,qa
```

####Parallelism

이전 버전에도 parallel 옵션이 있었다. 그래서 서버 그룹별로 다른 task를 수행할 수 있도로 했다.

``` ruby
# Capistrano 2.0.x
task :restart do
  parallel do |session|
    session.when "in?(:app)", "/u/apps/social/script/restart-mongrel"
    session.when "in?(:web)", "/u/apps/social/script/restart-apache"
    session.else "echo nothing to do"
  end
end
```
뭔가 깔끔하지 못한 느낌이 있었지만, 버전 3에서는 동일한 내용을 아래와 같이 작성할 수 있다.

``` ruby
# Capistrano 3.0.x
task :restart do
  on :all, in: :parallel do |host|
    if host.roles.include?(:app)
      execute "/u/apps/social/script/restart-mongrel"
    elsif host.roles.include?(:web)
      execute "/u/apps/social/script/restart-web"
    else
      info sprintf("Nothing to do for %s with roles %s", host,
      host.properties.roles)
    end
  end
end
```

parallelism에 대한 다른 예

``` ruby
# Capistrano 3.0.x
on :all, in: :groups, max: 3, wait: 5 do
  # Take all servers, in groups of three which execute in parallel
  # wait five seconds between groups of servers.
  # This is perfect for rolling restarts
end

on :all, in: :sequence, wait: 15 do
  # This takes all servers, in sequence and waits 15 seconds between
  # each server, this might be perfect if you are afraid about
  # overloading a shared resource, or want to defer the asset compilation
  # over your cluster owing to worries about load
end

on :all, in: :parallel do
  # This will simply try and execute the commands contained within
  # the block in parallel on all servers. This might be perfect for kicking
  # off something like a Git checkout or similar.
end
```

####스트리밍 IO

이와 같은 IO 스트리밍 모델은 명령 실행 결과를 의미하며, 명령 자체와 모든 다른 임의의 결과물은 IO 모양을 한 인터페이스를 가지는 클래스에 객체로서 보내진다. 이 클래스는 이러한 것들을 가지고 해야할 일을 알고 있다.

* a progress formatter : 각 명령이 호출될 때 점으로 표시한다.
* a pretty formatter : 전체 명령, 표준 출력과 표준 에러로 표시는 결과, 최종 반환 상태를 표시한다.
* a HTML formatter
* other formatters : IRC room이나 email로 표시한다.

####호스트 정의 접근

parallelism을 유심히 봤다면 실행 블록 내에서 `host`를 사용한 것을 알 수 있다.
여러가지 이유로 버전 2에서는 불가능했던 것으로 블록이 한번만 실행되고 각 서버에 대해서 verbatim을 호출했다.
Capistrano로부터 호스트 목록을 불러와서 Chef Solo 등을 제어하는 것 처럼 role별로 작업을 수행하도록 할 수 없었다.

그러나, 버전 3에서는, `host` 객체를 사용할 수 있는데, 이것은 서버가 정의될 때 생성되는 객체이다. 예를 들어, 배포가 성공적으로 완료되었을 때, 각 서버로 덤프해서 보여 줄 last-deploy 메시지를 렌더링하기 위해서 이 `host` 객체를 ERB 템플릿으로 넘겨 줄 수 있다.
last-deploy 로그에는 배포 중에 Capistrano가 해당 서버에 대해서 알고 있는 모든 것을 포함한다.

> Capistrano 2 사용자들은 계속해서 발생하는 `cap deploy:cleanup` 문제에 익숙해 있다. 이 문제는 서버들이 각자 가지고 있는 이전 릴리스 목록에 차이가 있을 때 발생했다. 서버가 두 대 있는 시나리오를 생각해 보자. 한 대는 서비스 론칭이후에 지속적으로 메인 서버로 사용해 왔으며, 수 개월 또는 수 년에 걸쳐 배포된 수백개나 되는 모든 릴리스를 가지고 있다. 두번째 서버는 대략 한 달동안 클러스터로 묵여 있었는데 제대로 연결이 되지 못했다. 그래서 이전 릴리스 목록들이 조금 이상하게 보여서, 직접 릴리스 몇개를 삭제했는데, 어쨌던 10개 정도만 남았을 것이다.

> 이제 `cap deploy:cleanup` 명령을 호출한다고 가정해 보자. 호스트 properties 속성에 해당하는 첫번째 서버에서만 `capture()` 명령이 실행되어, 첫번째 서버는 대략 95개의 릴리스 목록을 반환했다. 다음으로 Capistrano 2가 두개의 서버에 대해서 `rm -rf release1..release95` 명령을 호출하면, Capistrano가 두 개의 서버로의 연결이 끊어질 것이기 때문에, 두번째 서버에서는 에러가 발생할 것이고 첫번째 서버는 정의되지 않는 상태에 있게 될 것이다.

이러한 `cleanup` 루틴은 이제 아래와 같이 더 훌륭하게 구현될 수 있다. (이것은 새로운 젬에서 실제로 구현한 내용이다)

``` ruby
# Capistrano 3.0.x
desc "Cleanup all old releases (keeps #{fetch(:releases_to_keep_on_cleanup)}
old releases"
task :cleanup do
  keep_releases     = fetch(:releases_to_keep_on_cleanup)
  releases          = capture(:ls, fetch(:releases_directory))
  release_to_delete = releases.sort_by { |r| rn.to_i }.slice(1..-(keep_releases + 1))
  releases_to_delete.each do |r|
    execute :rm, fetch(:releases_directory).join(r)
  end
end
```

몇가지 편리한 점은 두대의 서버는 독립적으로 이 작업을 수행할 것이고 이전 릴리스를 제거하는 작업을 마쳤을 때 `task :cleanup` 블록은 종료하게 될 것이다.

또한, Capistrano 3에서, 대부분의 경로 변수들을 `[Pathname]` 객체이기 때문에, `#basename`, `#expand_path`, `#join` 등과 같은 메소드에 반응을 한다.

경고 : `#expand_path` 메소드는 아마도 기대한 대로 되지 않을 것이다. 이 메소드는 원격 호스트가 아니라 로컬 머신에서 실행될 것이기 때문에, 경로가 원격 서버상에는 존재하나 로컬 머신상에서는 존재하지 않을 때, 에러를 반환하게 될 가능성이 있다.


####호스트 properties 속성

`host` 객체는 이제 task 블록에서 사용할 수 있기 때문에, 이 객체에 임의의 값을 저장하는 것이 가능해 진다.

`host.properties`를 보자. 이것은 단단한 OpenStruct 구제를 가지는데, 어플리케이션에서 중요한 특성들을 추가해서 저장하고자 할 때 사용할 수 있다.

사용 예는 아래와 같다.

``` ruby
h = SSHKit::Host.new 'example.com'
h.properties.roles ||= %i{wep app}
```

####더 다양하게 표현할 수 있는 명령 언어

Capistrano 2에서는 아래와 같은 명령을 찾아보기가 힘들지 않았다.

``` ruby
# Capistrano 2.0.x
task :precompile, :roles => lambda { assets_role }, :except => { :no_release => true } do
  run <<-CMD.compact
    cd -- #{latest_release} &&
    RAILS_ENV=#{rails_env.to_s.shellescape} #{asset_env} #{rake} assets:precompile
  CMD
end
```

동일한 task를 Capistrano 3에서는 아래와 같이 작성할 수 있다.

``` ruby
# Capistrano 3.0.x
task :precompile do
  on :sprockets_asset_host, reject: lambda { |h| h.properties.no_release } do
    within fetch(:latest_release_directory) do
      with rails_env: fetch(:rails_env) do
        execute :rake, 'assets:precompile'
      end
    end
  end
end
```

다른 예제와 비교해 볼 때, 이러한 형태는 다소 더 긴듯하지만, 훨씬 더 잘 표현할 수 있으며 쉘 이스케이핑의 악몽은 내부적으로 처리된다. 환경변수은 대문자로 사용하여 정확한 위치(이 예에서는 `cd`와 `rake` 호출 사이)에 적용한다.

다른 옵션으로는 `as :a_user` 를 포함한다.


####더 좋은 magic 변수 지원

Capistrano 2에서는, 예를 들어 `lastest_release_directory` 변수 경우, 이 변수를 호출할 때 `NoMethodError` 예외가 발생하며 어떤 마술 같은 일이 일어났다. 이 변수가 전역 네임스페이스에 정의되어 있지 않으면, 대신 `set()` 변수들의 목록을 찾아 보기 때문이다.

이러한 마술같은 일은 때로는 사람들이 magic 변수들이 사용되고 있다는 것을 인지하지 못했다. Capistrano 2의 magic 변수 시스템은 해당 변수가 이미 설정되어 있지 않은 경우 `fetch(:some_variable, 'with a default value')`를 호출하는 방법을 지원했지만, 널리 사용되지 못했고, 배후에서는 예외가 발생하고 예외가 해결된 상태임을 알지 못한채, 대개 사람들은 그저 `lastest_release_directory`와 같은 것을 사용하기만 하는 경우가 더 자주 있었다. 그리고, 변수 맵에서 `:lastest_release_directory`는 실제로 최초로 사용될 때 평가되었는 던 값이고 그 값이 스크립트 끝나는 시점까지 캐시되었던 것이다.

이제 시스템은 마술같은 부분을 100% 없애 버렸다. `set()`을 이용하여 변수를 설정할 경우, `fetch()`를 이용해서 해당 변수의 값을 가져올 수 있고, 해당 변수에 설정한 값이 `#call` 메소드를 가지고 있는 경우 사용될 때마다 현재의 문맥상에서 실행될 것이다. 이 때 값은 이 값은 특별히 캐싱이 필요하지 않는 이상 캐시되지 않을 것이다. 역시, 우리는 미세한 최적화 보다는 명확한 것을 더 좋아한다.

###SSHKit

Capistrano의 많은 새로운 기능들(로깅, 포맷팅, SSH, 연결 관리 및 풀링, 평행성(parallism), 배치작업 등)은, Capistrano 3 개발 과정에서 만들어진, 라이브러리에서 찾아 볼 수 있다.

SSHKit는 Net::SSH 보다는 높은 레벨이지만, 여전히 낮은 레벨의 툴킷으로, Capistrano의 역할(role), 환경(environment), 롤백과 다른 더 높은 레벨의 기능들이 없다.

SSHKit는 원격 머신에 단순히 연결해서 어떤 임의의 명령을 실행할 필요가 있을 때 이상적이다. 예를 들면,

``` ruby
# Rakefile (even without Capistrano loaded)
require 'SSHKit'
desc "Check the uptime of example.com"
task :uptime do |h|
  execute :uptime
end
```

SSHKit로 할 수 있는 것이 매우 많은데, 풍부한 [예제 목록](https://github.com/leehambley/SSHKit/blob/master/EXAMPLES.md)이 있다.  Capistrano 3 대부분에서, `on()` 블록 내에서 일어나는 모든 것은 SSHKit 로 발생하는 것이고, 이 라이브러리에 대한 문서를 참고하면 더 많은 정보를 얻을 수 있다.

<h3>명령(Command) 매핑</h3>

이것은 SSHKit의 또 다른 기능이다. 이것은 절차상의 모호함을 없애기 위해서 디자인 되었다. 명령들에 대한 소위 명령 맵이다.

아래와 같이 실행할 때

``` ruby
# Capistrano 2.0.x
execute "git clone ........ ......."
```

이 명령은 원격 서버로 변경없이 그대로 넘어 간다. 여기에는 사용자, 디렉토리, 환경변수와 같은 옵션들을 포함할 수 있다. 이것은 디자인 상의 문제이다. 이 기능은 필요할 때 [heredocs](https://en.wikipedia.org/wiki/Here_document) 형태로 명령을 작성할 수 있도록 고안되었다.

``` ruby
# Capistrano 3.0.x
execute <<-EOBLOCK
  # All of this block is interpreted as Bash script
  if ! [ -e /tmp/somefile ]
    then touch /tmp/somefile
    chmod 0644 /tmp/somefile
  end
EOBLOCK
```

경고 : SSHKit 다중라인 명령 이스케이핑(sanitizing) 로직은 라인피드를 제거하고 명령을 구분하기 위해서 각 라인 끝에 세미콜론(;)을 추가해 줄 것이다. 따라서 `then`과 다음 명령 사이에 개인문자가 없다는 것을 확인해야 한다.

Capistrano 3 에서 명령을 작성하는 방법은 별개의 메소드(복수개의 인수를 가지는, variadaric)를 이용해서 명령을 지정하는 것이다.

``` ruby
# Capistrano 3.0.x
execute :git, :clone, "........", "......."
```

또는 더 큰 예에서는,

``` ruby
# Capistrano 3.0.x
file = '/tmp/somefile'
unless test("-e #{file}")
  execute :touch, file
end
```

이런 식으로 명령 맵이 참조된다. 명령 맵은 모든 알 수 없는 명령들(여기서는 `git` 이 명령이고 나머지가 `git` 의 인수이다)이 `/user/bin/env ...` 으로 매핑되도록 한다.  즉, 이 명령은 `/usr/bin/env git clone ... ...`로 확장된다는 것을 의미한다. 전체 경로없이 `git`가 호출될 때 이러한 일이 발생한다. 이 때 `env`  프로그램은 어떤 `git`를 실행할 지를 결정하기 위해서 (아마도 간접적으로) 참조된다.

`rake`와 `rails` 명령은 자주 `bundle exec`를 앞에 붙여 주는 것이 좋은데 이 경우에는 아래와 같이 매핑할 수 있다.

``` ruby
SSHKit.config.command_map[:rake]  = "bundle exec rake"
SSHKit.config.command_map[:rails] = "bundle exec rails"
```

아래와 같이 매핑하는 곳에 `lambda`나 `Proc` 객체를 적용할 수도 있다.

``` ruby
SSHKit.config.command_map = Hash.new do |hash, key|
  if %i{rails rake bundle clockwork heroku}.include?(key.to_sym)
    hash[key] = "/usr/bin/env bundle exec #{key}"
  else
    hash[key] = "/usr/bin/env #{key}"
  end
end
```

위에서 언급한 두가지 옵션 사이에는, Capistrano 내장 task를 변경하지 않은 채, 해당 환경에서 명령들을 매핑하는 매우 강력한 옵션이 있어야 한다. 왜냐 하면, 경로가 다르거나 바이너리 파일이 다른 이름을 가지기 때문이다.

또한 예를 들어 `rbenv`  랩퍼(wrapper)와 같은 `shim` 실행명령들이 사용하는 환경들이 약간 남용될 수 있다.

``` ruby
SSHKit.config.command_map = Hash.new do |hash, key|
  if %i{rails rake bundle clockwork heroku}.include?(key.to_sym)
    hash[key] = "/usr/bin/env myproject_bundle exec myproject_#{key}"
  else
    hash[key] = "/usr/bin/env #{key}"
  end
end
```

위의 코드는 `rbenv wrapper default myproject` 와 같은 것을 실행했다고 가정한 것이고 이것은 인터액티브(interactive) 로긴 쉘이 필요없이 루비 환경을 정확하게 설정하는 랩퍼 바이너리(wrapper binaries)를 생성한다.


###테스트하기

Capistrano의 이전 버전의  테스트 슈트는 순수한 단위 테스트이고 다양한 경우의 문제를 다루지 못했다. 특히 실제 배포 코드인 `deploy.rb` 파일에 있는 어떤 것도 전혀 테스트하지 못했는데, Capistrano 자체의 DSL을 실행해야 하기 때문이었고, 다른 약간 이상한 디자인  문제로 실제 레시피를 테스트하는 것이 고통스러웠다.

테스트는 Capistrano 3의 초점이 되어왔다. 통합 테스트 슈트는 Vagrant를 이용해서 머신을 부팅한 후, portable 쉘스트립트를 이용하여 어떤 시나리오를 구성하고, 그 후에 시나리오에 대한 명령을 실행하여 전형적인 리눅스 시스템에 대한 공통된 구성을 배포한다. 이것을 실행 속도가 느리지만, 이전에 우리가 줄 수 있었던, 아무것도 깨지지 않았다는 보장을 더 강력하게 제공한다.

Capistrano 3는 또한 백엔드 실행을 교체할 수 있도록 지원한다. 이것은, 자신의 레시피를 테스트할 목적으로, 백엔드에서 프린터를 사용할 수 있고, 결과물이 기대했던 것과 일치하는 지를 입증할 수 있으며, 또는 호출이 제대로 되었는지 아니면 기대했던 대로 호출되지 않았는지를 입증할 수 있는 a stubbed backend(?)를 사용할 수 있다.

###재량에 따른 로깅

Capistrano는 `on()` 블록 내에서 `debug()`, `info()`, `warn()`, `error()`, `fatal()` 메소드를 사용할 수 있다. 이 메소드를 이용하면 기존의 로깅 인프라와 스트리밍 IO 포맷터를 이용해서 로그를 남길 수 있다.

``` ruby
# Capistrano 3.0.x
on hosts do |host|
  f = '/some/file'
  if test("[ -d #{f} ]")
    execute :touch, f
  else
    info "#{f} already exists on #{host}!"
  end
end
```


###업그레이드

업그레이드 관련 사항을 상세하게 알고 싶으면 [업그레이드 문서](http://capistranorb.com/documentation/upgrading/)를 보기 바란다.

간단한 버전은 **직접** 업그레이드하는 방법은 없다고 말하는 것이다. 버전 2와 3은 전혀 호환성이 없다.

이것은 부분적으로는 디자인 상의 문제이기도 한데, 이전의 DSL은, 올바른 것을 하는 것을 대부분 교묘하게 만들었던 곳에서, 정확하지 못했기 때문에, API를 이전 버전과의 호환성을 유지하도록 하는 것에 투자하는 것 보다는, 더 많은 기능과 더 좋은 신뢰성에 투자하는 것을 선택했다.

아래에 많은 기능들이 나열되어 있지만 중요한 것들은 내장 role의 새로운 이름이고, 디폴트로 Capistrano 3는 프랫폼을 인지하지 못한다는 것이다. 마이그레이션, asset pipeline 등과 같이 레일스 지원이 필요할 경우에는 해당 지원 파일들을 `require`해야 한다.

###기능들(Gotchas)

####Rake DSL은 추가적이다.

Capistrano 2에서는, 특정 task를 재정의하면 원래 구현된 내용을 대체해 버린다. 이것은 내장 task를 자신의 구현내용으로 대체하고자 하는 사람들이 사용했었다.


---

_**References:**_

1. [Capistrano Version 3 Release Announcement](http://capistranorb.com/2013/06/01/release-announcement.html)
