# JDK Docker 이미지로 JRE Docker 이미지 만들기
## 목표
* 최대한 작은 사이즈의 JRE 이미지를 생성한다.
* 가능한 외부의 이미지를 사용하지 않는다.
* 하나의 Dockerfile로 생성하도록 한다.
## 필요 환경
* docker(podman도 가능)
## 순서
## 참고 URL
* [Java11, jlink and Docker](https://greut.medium.com/java11-jlink-and-docker-2fec885fb2d)
* [JRE를 쉽게 만들 수 있게 도와주는 온라인 툴](https://justinmahar.github.io/easyjre/)
* [Amazon Corretto JRE11 Dockerfile](https://github.com/corretto/corretto-docker/blob/d98afb2f45401891e62a0651499b929f8da823eb/11/jre/alpine/Dockerfile)
* [Apine Linux에서 glibc 사용하기](https://ssaru.github.io/2021/05/09/20210509-til_set_locale_in_alpine_linux/)