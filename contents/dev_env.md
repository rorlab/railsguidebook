# 개발환경

통상적으로 웹프로젝트를 제작하여 일반인들에게 서비스하기 위해서는 `개발서버`(개발시 사용하는 서버)와 `운영서버`(실제 운영을 위한 서버)가 필요하다. 여기에 추가적으로 `테스트서버`([staging 서버](https://www.techopedia.com/definition/4205/staging-server): 운영환경과 동일하게 구축하여 실제 운영서버로 배포전에 발생할 수 있는 문제점을 최종적으로 점거하기 위한 서버)도 구축하여 운영할 수 있다. 

<img src="http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/rails_guideline/Matz_zpsb4305ed2.png">

레일스는 루비언어로 만들어졌기 때문에 모든 환경에서 루비 인터프리터가 설치되어 있어야 한다. 루비 언어를 만든 일본의 마츠의 이름이 기리기 위해서 원조 루비 인터프리터를 MRI(Matz' Ruby Interpreter)라고 하는데, C 언어로 작성되었다. 참고로 MRI외의 대표적인 루비 구현체로 JRuby와 Rubinius 등이 있는데, JRuby는 자바로 구현된 루비 구현체이고 Rubinius는 루비로 구현된 루비구현체이다.



* [소스관리자(GIT) 설치](git.html)
* 루비버전관리자의 설치 :
  * [rbenv](rbenv.html)
  * [rvm](rvm.html)
* [레일스 설치](rails/install.html)
* [코드 에디터의 선택](editors.html)
