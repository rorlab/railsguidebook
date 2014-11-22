# Capistrano 2.x 에서 업그레이드하기


(1) Gemfile을 업데이트한다.

``` ruby
gem 'capistrano', '~> 3.0', require: false, group: :development
```

레일스를 배포할 경우에는, `capistrano-rails`와 `capistrano-bundler` 젬이 필요하다. Capistrano 3.x 에서는 레일스와 Bundler 통합이 제거되었다.

``` ruby
group :development do
  gem 'capistrano-rails',   '~> 1.1', require: false
  gem 'capistrano-bundler', '~> 1.1', require: false
end
```

rvm, rbenv, chruby 중에 선호하는 루비버전관리자에 대한 지원을 추가한다.

``` ruby
group :development do
  gem 'capistrano-rvm',   '~> 0.1', require: false
  gem 'capistrano-rbenv', '~> 2.0', require: false
  gem 'capistrano-chruby', github: 'capistrano/chruby', require: false
end
```

(2) 처음부터 프로젝트 `capify`할 것을 권한다. 이전 버전의 정의들은 별도의 디렉토리로 옮긴다.

``` sh
$ mkdir old_cap
$ mv Capfile old_cap
$ mv config/deploy.rb old_cap
$ mv config/deploy/ old_cap # --> only for multistage setups
```

이제 `capify` 한다.

``` sh
$ cap install
```

(3) Capistrano 3.x 는 디폴트로 multistage를 지원한다. 따라서 `capify`후 `config/deploy/production.rb`와 `config/deploy/staging.rb` 파일이 디폴트로 생성될 것이다. 하나의 stage만이 필요한 경우에는, 이 파일들을 제거하고 `config/deploy.rb` 파일에 stage(예, production)와 servers를 선언한다.

(4) `config/deploy/production.rb` 와 `config/deploy/staging.rb` 파일을 업데이트해서 관련 데이터를 지정한다. 이전 설정(old_cap/deploy/)로부터 더 많은 stages를 추가할 수도 있다.

(5) 게이트웨이 서버 셋팅이 `set :gateway, "www.capify.org"` 로 지정되어 있었다면, 아래와 같이 업그레이드해야 한다.

``` ruby
require 'net/ssh/proxy/command'
set :ssh_options, proxy: Net::SSH::Proxy::Command.new('ssh mygateway.com -W %h:%p')
```

또는 서버별로 `ssh_options` 을 지정할 수 있다.

(6) 이제 이전 `deploy.rb` 파일(물론 `Capfile`도 함께, 그러나 Capistrano 2.x 에서는 이 파일을 변경하지 않았었다.)을 리팩토링할 필요가 있다. 파라미터들(`set :deploy_to, "/home/deploy/#{application}"` or `set :keep_releases, 4` 등)을 `config/deploy.rb` 파일로, task들은 `Capfile`로 옮긴다.

중요 : `repository` 옵션은 `repo_url`로, `default_environment` 옵션은 `default_env`로 이름이 변경되었다.

`use_sudo` 와 `normalize_asset_timestamps` 파라미터는 더 이상 필요없다는 것을 주목하자.

(7) 이전에 `deploy_to` 옵션을 사용하지 않고 `/u/apps/your_app_name`으로 배포했다면, 한가지 더 변경해야 한다. 이제 디폴트 배포 경로가 `/var/www/app_name`로 변경되었기 때문에 업그레이드 후 config 가 깨질 것이다. 아래와 같이 커스텀 `deploy_to` 옵션을 선언해 주면 해결된다.

``` ruby
set :deploy_to, "/u/apps/#{fetch(:application)}"
```

그러나, `/u/apps` 는 앱을 저장하기에 최선의 장소는 아니므로 나중에 변경할 것을 권장한다.

(8) Capfile을 수정해서 필요한 addon들(rbenv/rvm, bundler, rails)을 추가한다.

``` ruby
require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rbenv'
```

(9) 이제 새로 작성한 config로 배포해 보자. 본 가이드에 빠진 내용을 발견하면 기여해 주기 바란다.

###일반적인 추천사항


####ENV 변수를 작성하지 말고 DSL을 사용하라

``` ruby
run <<-CMD.compact
  cd -- #{latest_release} &&
  RAILS_ENV=#{rails_env.to_s.shellescape} #{asset_env} #{rake} assets:precompile
CMD
```
대신에

``` ruby
on roles :all do
  within fetch(:latest_release_directory) do
    with rails_env: fetch(:rails_env) do
      execute :rake, 'assets:precompile'
    end
  end
end
```

와 같이 사용하는 것이 더 좋다.

주의 : DSL이 인식하기 위해서는 `on` 블록으로 감싸줘야 하는데, 이 때 `within` 블록이 필요하다.

호출 당 하나의 `with` 블록을 가질 수 있다. 하나 이상의 env 설정이 필요한 경우는 아래와 같이 `with` 블록을 작성한다(하나의 맵으로 넘겨 준다).

``` ruby
on roles :all do
  within fetch(:latest_release_directory) do
    with rails_env: fetch(:rails_env), rails_relative_url_root: '/home' do
      execute :rake, 'assets:precompile', env: {rails_env: fetch(:rails_env), rails_relative_url_root: ''}
    end
  end
end
```

---

_**References:**_

1. [Upgrading from v2.x.x](http://capistranorb.com/documentation/upgrading/)
