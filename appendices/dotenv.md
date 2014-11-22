# 개발 환경변수를 관리하는 dotenv 젬

https://github.com/bkeepers/dotenv

`dotenv` 젬은 개발모드에서 `.env` 파일로부터 `ENV`로 환경변수를 로드하기 위한 것이다. 따라서 운영(배포)모드에서는 사용할 때는 별도의 `dotenv-deployment` 젬을 사용하면 된다.

사실 다수의 프로젝트를 실행하는 경우 개발 머신이나 CI 서버에서 환경변수를 사용하는 것이 반드시 실용적인 것은 아니다. 이런 경우 `dotenv`를 사용하면 개발모드가 시작될 때 `.env` 파일을 `ENV`로 변수들을 로드하게 된다.

### 설치

`Gemfile`에 아래와 같이 추가하고,

```ruby
gem 'dotenv-rails', groups: [:development, :test]
```
설치한다.

```bash
$ bundle install
```

### 사용법

프로젝트의 루트에 `.env` 파일를 만든 후 아래와 같이 어플리케이션 설정을 위한 값들을 추가한다.

```
S3_BUCKET=YOURS3BUCKET
SECRET_KEY=YOURSECRETKEYGOESHERE
```

또는 yaml 포맷도 지원하여 아래와 같이 작성해도 된다.

```
S3_BUCKET: yamlstyleforyours3bucket
SECRET_KEY: thisisalsoanokaysecret
```

이제 어플리케이션이 로드될 때마다 아래와 같이 이 변수값을 `ENV`에서 사용할 수 있게 된다.

```ruby
config.fog_directory  = ENV['S3_BUCKET']
```

### .env 파일의 커밋

개발모드에서 사용하는 환경설정 값만을 `.env` 파일에 저정하고 저장소로 커밋하도록 권한다. 즉, 개발 환경에서 사용하는 모든 보안사항은 다른 배포환경에서 사용하는 것과 다르다는 것을 확인해야 한다. 같은 프로젝트에 참여하는 다른 개발자들도 동일한 개발 환경을 별도의 설정없이 바로 사용할 수 있게 되는 것이다.

> **Caution** 그러나 주의할 것은 공개 저장소로 푸시해서는 안 된다는 것에 유의하기 바란다. 즉, `github`의 공개 저장소에 커밋할 경우 `.env` 파일내의 환경변수에 포함된 보안데이터가 공개되기 때문에 반드시 `private` 저장소로만 푸시해서 사용해야 한다.
