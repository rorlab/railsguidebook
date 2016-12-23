# AWS S3 사용하기

`AWS S3` : ***A***mazone ***W***eb ***S***ervices ***S***imple ***S***torage ***S***ervices

레일스 어플리케이션을 허로쿠로 배포할 때의 문제점은 허로쿠의 [ephemeral filesystem](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem) 때문이다. `ephemeral`이란 단어는 단명의, 단 하루뿐인이란 의미를 가지는데, 미루어 짐작할 수 있듯이 업로드된 파일이 일정시간 지나면 자동으로 삭제된다.

이에 대한 해결책으로 `AWS S3`를 이용하면 허로쿠로 배포된 레일스 어플리케이션에서 파일 업로드 후 지속적으로 이미지를 볼 수 있게 된다.

`S3`의 모든 파일은, 디렉토리와 매우 흡사한 상위레벨의 컨테이너로 동작하는, `bucket`이라는 곳에 저장된다. `S3`로 파일을 보내면 하나의 `bucket`에 속하게 되는데, `bucket` 명은 전체 아마존 시스템에서 유일해야 한다.

`Access Key ID`와 `Secret Access Key`가 있어야 `S3 API`로 접근할 수 있다. 여기서 access 키는 `S3` 사용자 계정이고 secret 키는 비밀번호에 해당하기 때문에 다른 사람에게 노출되지 않도록 주의해야 한다.

### S3 셋업

`S3`를 사용하기 위해서는 `AWS Credentials`와 파일 저장을 위한 `bucket`을 확보해야 한다. 각각에 대해서는 아래를 참조하기 바란다.

#### Credentials

자세한 내용은 [`여기`](https://devcenter.heroku.com/articles/s3#credentials)를 참고하기 바란다.

#### Bucket

자세한 내용은 [`여기`](https://devcenter.heroku.com/articles/s3#bucket)를 참고하기 바란다.

#### Bucket Naming

자세한 내용은 [`여기`](https://devcenter.heroku.com/articles/s3#naming-buckets)를 참고하기 바란다.

### Carrierwave 젬 사용시 S3 셋업 방법

레일스에서는 `S3` 저장을 위한  `aws-sdk` 라이브러리를 사용할 수 있는데 [`carrierwave-aws`](https://github.com/sorentwo/carrierwave-aws)라는 `carrierwave`용  젬이 있다. 따라서 `Gemfile`에서 이미 등록한 `carrierave` 대신에 `carrierwave-aws` 젬을 추가하면 된다. 왜냐하면, 이 젬을 설치할 때 젬 의존성에 따라 `carrierwave`와 `aws-skd` 젬이 자동으로 설치되기 때문이다.

```ruby
gem 'carrierwave-aws'
```

위와 같이 `Gemfile`에 젬을 추가하고

```bash
$ bundle install
```

설치한다.

#### 단순 비교 테이블 [07/17/2013]

[`fog`](https://github.com/carrierwaveuploader/carrierwave#using-rackspace-cloud-files) 젬을 이용하여 `AWS S3` 서비스를 사용할 수 있지만 아래와 같은 장점이 있어 `aws-sdk` 젬이 더 선호된다.

| Library | Disk Space | Lines of Code | Boot Time | Runtime Deps | Develop Deps |
| ------- | ---------- | ------------- | --------- | ------------ | ------------ |
| fog     | 28.0M      | 133469        | 0.693     | 9            | 11           |
| aws-sdk | 5.4M       |  90290        | 0.098     | 3            | 8            |


#### 사용법

`config/initializers/carrierwave.rb` 파일에 아래와 같이 추가한다.

```ruby
CarrierWave.configure do |config|
  config.storage    = :aws
  config.aws_bucket = ENV['S3_BUCKET_NAME']
  config.aws_acl    = :public_read
  config.asset_host = 'http://example.com'
  config.aws_authenticated_url_expiration = 60 * 60 * 24 * 365

  config.aws_credentials = {
    access_key_id:     ENV['AWS_ACCESS_KEY_ID'],
    secret_access_key: ENV['AWS_SECRET_ACCESS_KEY']
  }
end if Rails.env == 'production'
```

`config.asset_host`값을 `AWS S3`의 해당 `bucket`의 `Endpoint` 주소로 변경한다. (예, `https://s3-ap-northeast-1.amazonaws.com/<bucket-name>`)

> ####Caution::주의
> 현재 `AWS S3 bucket`의 `Endpoint` 주소의 링크의 오류가 있으므로 위의 예를 복사해서 `bucket-name` 만 변경해서 사용하도록 한다.

위에서 사용한 'S3_BUCKET_NAME', 'AWS_ACCESS_KEY_ID', 'AWS_SECRET_ACCESS_KEY'을 시스템 환경변수로 등록하기 위해서 `~/.bash_profile` 또는 `~/.zshrc` 파일을 열어서 아래와 같이 추가한다.

```
export S3_BUCKET_NAME=<user-bucket-name>
export AWS_ACCESS_KEY_ID=<user-access-key>
export AWS_SECRET_ACCESS_KEY=<user-secret-key>
```

허로쿠로 배포시에는 아래와 같이 추가 작업을 해 준다.

```bash
$ heroku config:set S3_BUCKET_NAME=<user-bucket-name>
$ heroku config:set AWS_ACCESS_KEY_ID=<user-access-key>
$ heroku config:set AWS_SECRET_ACCESS_KEY=<user-secret-key>
```

그리고 파일 업로드 클래스 파일에 아래와 같이 `storage` 옵션 조건을 추가한다.

```ruby
if Rails.env == 'production'
  storage :aws
else
  storage :file
end
```

> **Info** `paperclip` 젬을 사용할 때는 [paperclip-s3](https://devcenter.heroku.com/articles/paperclip-s3)를 참고하기 바란다.

---

> **Note** `fog` 젬 사용법은 [`여기`](https://gist.github.com/cblunt/1303386)를 참고하면 도움이 된다.

