# Postgres (or PostgreSQL) 설치하기

: 개발장비에 PostgreSQL 서버를 설치하는 방법을 소개한다. 

### 맥 OSX에 설치하기

두가지 방법으로 설치할 수 있다.

1. homebrew를 이용
2. Postgre.app을 이용

---

### 1. homebrew을 이용하여 설치하기

```bash
$ brew update
$ brew install postgresql
```


자세한 내용은 [여기](http://www.moncefbelyamani.com/how-to-install-postgresql-on-a-mac-with-homebrew-and-lunchy/)를 참고하기 바란다.


### 2. Postgre.app을 이용하여 설치하기

맥에서는 [Postgre.app](http://postgresapp.com)을 설치하면 바로 PostgreSQL에 접속할 수도 있다.

Download and Install PostgreSQL database from Postgres.app which provides PostgresSQL in a single package to easily get started with Max OS X. After the installation, open Postgres located under Applications to start PostgreSQL database running. Find out the PostgreSQL bin path and append to ~/.bashrc for accessing commands through the shell.

```bash
$ echo 'PATH="/Applications/Postgres.app/Contents/Versions/9.3/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc
```


### 레일스 프로젝트를 위한 준비


* 레일스 프로젝트에서 사용한 PostgreSQL 유저를 생성한다.

  ```bash
$ createuser -P -d -e sampleuser
Enter password for new role:
Enter it again:
CREATE ROLE sampleuser PASSWORD 'md5afd8d364af0c8efa11183c3454f56c52' NOSUPERUSER CREATEDB NOCREATEROLE INHERIT LOGIN;
```


* `config/database.yml`를 아래와 같이 수정한다.

  ```ruby
  production:
    adapter: postgresql
    encoding: unicode
    database: <database-name>   # 원하는 대로 DB 명을 정한다.
  ```






