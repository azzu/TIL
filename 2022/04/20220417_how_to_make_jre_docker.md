# JDK Docker 이미지로 JRE Docker 이미지 만들기
## 목표
* 최대한 작은 사이즈의 JRE 이미지를 생성한다.
* 가능한 외부의 이미지를 사용하지 않는다.
* 하나의 Dockerfile로 생성하도록 한다.
## 필요 환경
* JDK
* docker(podman도 가능)
## JRE Docker 컨테이너 만들기
Springboot와 같은 Java를 사용하는 환경에서 Docker 컨테이너를 통해 배포/운영하게 될 경우,   
[OpenJDK](https://hub.docker.com/_/openjdk)와 같은 JDK를 Base Image로 Docker 컨테이너를 생성하는 경우가 많은데,   
JDK를 Base로 생성하게 되면, 최종 생성되는 컨테이너의 사이즈가 매우 커지게 된다(300MB!).   
![Docker Hub JDK-11 Base Image](/resources/static/img/docker_hub-openjdk-11.png)   
```javac```와 같은 JDK로 컴파일된 최종 결과물(Springboot의 경우 *.jar)을 실행할때는 JRE만 있어도 충분하며,   
JRE를 Base로 컨테이너를 생성하면 컨테이너 사이즈도 작아져 리소스 사용시에 효율적이다.   

하지만, JDK Docker 컨테이너 중에는 JRE 컨테이너가 없는 경우도 있는데, 이 경우에는 JDK에 포함되어있는 ```jlink```를 이용하여   
JRE 컨테이너를 만들어 줄 수 있다.   
```jlink```명령어에 대한 사용법은 [여기](https://docs.oracle.com/javase/9/tools/jlink.htm)를 참고하면 되고, 아래의 2가지 방법으로 원하는 모듈을 가져 올 수 있다.
### 1. 명령어를 통한 JRE 모듈리스트 보기
```java --list-modules```를 통해 모듈 리스트를 보고 *java.** 로 시작하는 모듈들을 ```jlink```에 실행시 지정하면 해당 모듈들로 jre를 생상해 준다.
```shell
]$ java --list-modules
java.base@11.0.11
java.compiler@11.0.11
java.datatransfer@11.0.11
java.desktop@11.0.11
java.instrument@11.0.11
java.logging@11.0.11
java.management@11.0.11
java.management.rmi@11.0.11
java.naming@11.0.11
java.net.http@11.0.11
java.prefs@11.0.11
java.rmi@11.0.11
java.scripting@11.0.11
java.se@11.0.11
java.security.jgss@11.0.11
java.security.sasl@11.0.11
java.smartcardio@11.0.11
java.sql@11.0.11
java.sql.rowset@11.0.11
java.transaction.xa@11.0.11
java.xml@11.0.11
java.xml.crypto@11.0.11
jdk.accessibility@11.0.11
jdk.aot@11.0.11
jdk.attach@11.0.11
jdk.charsets@11.0.11
jdk.compiler@11.0.11
jdk.crypto.cryptoki@11.0.11
jdk.crypto.ec@11.0.11
jdk.dynalink@11.0.11
jdk.editpad@11.0.11
jdk.hotspot.agent@11.0.11
jdk.httpserver@11.0.11
jdk.internal.ed@11.0.11
jdk.internal.jvmstat@11.0.11
jdk.internal.le@11.0.11
jdk.internal.opt@11.0.11
jdk.internal.vm.ci@11.0.11
jdk.internal.vm.compiler@11.0.11
jdk.internal.vm.compiler.management@11.0.11
jdk.jartool@11.0.11
jdk.javadoc@11.0.11
jdk.jcmd@11.0.11
jdk.jconsole@11.0.11
jdk.jdeps@11.0.11
jdk.jdi@11.0.11
jdk.jdwp.agent@11.0.11
jdk.jfr@11.0.11
jdk.jlink@11.0.11
jdk.jshell@11.0.11
jdk.jsobject@11.0.11
jdk.jstatd@11.0.11
jdk.localedata@11.0.11
jdk.management@11.0.11
jdk.management.agent@11.0.11
jdk.management.jfr@11.0.11
jdk.naming.dns@11.0.11
jdk.naming.ldap@11.0.11
jdk.naming.rmi@11.0.11
jdk.net@11.0.11
jdk.pack@11.0.11
jdk.rmic@11.0.11
jdk.scripting.nashorn@11.0.11
jdk.scripting.nashorn.shell@11.0.11
jdk.sctp@11.0.11
jdk.security.auth@11.0.11
jdk.security.jgss@11.0.11
jdk.unsupported@11.0.11
jdk.unsupported.desktop@11.0.11
jdk.xml.dom@11.0.11
jdk.zipfs@11.0.11
```

### 2. 온라인 툴 이용하는 방법
https://justinmahar.github.io/easyjre/ 를 이용하여 ```jlink```명령어와 옵션들을 생성한다.   

#### jlink 명령어
```shell
]$ $JAVA_HOME/bin/jlink --output jre-11 --compress=2 --no-header-files --no-man-pages \
    --strip-debug --release-info $JAVA_HOME/release --module-path ../jmods \
    --add-modules java.base,java.datatransfer,java.desktop,java.instrument,java.logging, \
    java.management,java.management.rmi,java.naming,java.prefs,java.rmi,java.security.sasl, \
    java.xml,jdk.internal.vm.ci,jdk.jfr,jdk.management,jdk.management.jfr,jdk.management.agent, \
    jdk.net,jdk.sctp,jdk.unsupported,jdk.naming.rmi,java.compiler,jdk.aot,jdk.internal.vm.compiler, \
    jdk.internal.vm.compiler.management,java.se,java.net.http,java.scripting,java.security.jgss, \
    java.smartcardio,java.sql,java.sql.rowset,java.transaction.xa,java.xml.crypto,jdk.accessibility, \
    jdk.charsets,jdk.crypto.cryptoki,jdk.crypto.ec,jdk.dynalink,jdk.httpserver,jdk.jsobject,jdk.localedata, \
    jdk.naming.dns,jdk.scripting.nashorn,jdk.security.auth,jdk.security.jgss,jdk.xml.dom,jdk.zipfs, \
    jdk.jdwp.agent,jdk.pack,jdk.scripting.nashorn.shell,jdk.jcmd,jdk.jfr
```

여기까지의 내용을 바탕으로 Dockerfile을 작성해 보자.   
용량을 줄이기 위해 Multi Stage방법을 이용하여 JRE Docker 이미지를 생성하였다.
```dockerfile
FROM amazoncorretto:11 AS jre-build

RUN $JAVA_HOME/bin/jlink --output jre-11 --compress=2 --no-header-files --no-man-pages \
    --strip-debug --release-info $JAVA_HOME/release --module-path ../jmods \
    --add-modules java.base,java.datatransfer,java.desktop,java.instrument,java.logging, \
    java.management,java.management.rmi,java.naming,java.prefs,java.rmi,java.security.sasl, \
    java.xml,jdk.internal.vm.ci,jdk.jfr,jdk.management,jdk.management.jfr,jdk.management.agent, \
    jdk.net,jdk.sctp,jdk.unsupported,jdk.naming.rmi,java.compiler,jdk.aot,jdk.internal.vm.compiler, \
    jdk.internal.vm.compiler.management,java.se,java.net.http,java.scripting,java.security.jgss, \
    java.smartcardio,java.sql,java.sql.rowset,java.transaction.xa,java.xml.crypto,jdk.accessibility, \
    jdk.charsets,jdk.crypto.cryptoki,jdk.crypto.ec,jdk.dynalink,jdk.httpserver,jdk.jsobject,jdk.localedata, \
    jdk.naming.dns,jdk.scripting.nashorn,jdk.security.auth,jdk.security.jgss,jdk.xml.dom,jdk.zipfs, \
    jdk.jdwp.agent,jdk.pack,jdk.scripting.nashorn.shell,jdk.jcmd,jdk.jfr

FROM alpine:3.15

ENV LANG=C.UTF-8 LC_ALL=en_US.UTF-8
ENV JAVA_HOME=/usr/lib/jvm/default-jvm
ENV PATH="$JAVA_HOME/bin:${PATH}"
COPY --from=jre-build jre-11 $JAVA_HOME
```

## 하지만...java가 실행되지 않는다!!!!
해당 Dockerfile로 이미지를 생성하고 해당 이미지를 base로 docker build를 해서 실행하면 ```java```파일을 찾을 수 없다면서 실행되지 않는다.   
왜일까? 이유는, alpine linux가 초경량을 지향하면서 내부에 *```glibc```모듈 대신에 ```musl```을 사용하기 때문이다.*   
고로 alpine linux base 이미지에 ```glibc```를 설치해 준다.
Apine Linux용 ```glibc```는 [sgerrand github](https://github.com/sgerrand/alpine-pkg-glibc)에서 찾아서 설치할 수 있다.   
설치 후 ```ln```을 통해 라이브러리 링크를 걸어주고, ```glibc-bin```, ```glibc-i18n```까지 설치하고, ```localedef```로 locale설정까지 해주면 된다.
```shell
/usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8
```

## 최종 Dockerfile
```dockerfile
FROM amazoncorretto:11 AS jre-build

RUN $JAVA_HOME/bin/jlink --output jre-11 --compress=2 --no-header-files --no-man-pages \
    --strip-debug --release-info $JAVA_HOME/release --module-path ../jmods \
    --add-modules java.base,java.datatransfer,java.desktop,java.instrument,java.logging, \
    java.management,java.management.rmi,java.naming,java.prefs,java.rmi,java.security.sasl, \
    java.xml,jdk.internal.vm.ci,jdk.jfr,jdk.management,jdk.management.jfr,jdk.management.agent, \
    jdk.net,jdk.sctp,jdk.unsupported,jdk.naming.rmi,java.compiler,jdk.aot,jdk.internal.vm.compiler, \
    jdk.internal.vm.compiler.management,java.se,java.net.http,java.scripting,java.security.jgss, \
    java.smartcardio,java.sql,java.sql.rowset,java.transaction.xa,java.xml.crypto,jdk.accessibility, \
    jdk.charsets,jdk.crypto.cryptoki,jdk.crypto.ec,jdk.dynalink,jdk.httpserver,jdk.jsobject,jdk.localedata, \
    jdk.naming.dns,jdk.scripting.nashorn,jdk.security.auth,jdk.security.jgss,jdk.xml.dom,jdk.zipfs, \
    jdk.jdwp.agent,jdk.pack,jdk.scripting.nashorn.shell,jdk.jcmd,jdk.jfr

FROM alpine:3.15

ENV LANG=C.UTF-8 LC_ALL=en_US.UTF-8
ENV JAVA_HOME=/usr/lib/jvm/default-jvm
ENV PATH="$JAVA_HOME/bin:${PATH}"

RUN wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub && \
  wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-2.34-r0.apk && \
  apk update && apk add gzip glibc-2.34-r0.apk && \
  ln -sf /usr/glibc-compat/lib/ld-linux-x86-64.so.2 /usr/glibc-compat/lib/ld-linux-x86-64.so && \
  ln -sf /usr/glibc-compat/lib/libBrokenLocale.so.1 /usr/glibc-compat/lib/libBrokenLocale.so && \
  ln -sf /usr/glibc-compat/lib/libanl.so.1 /usr/glibc-compat/lib/libanl.so && \
  ln -sf /usr/glibc-compat/lib/libc_malloc_debug.so.0 /usr/glibc-compat/lib/libc_malloc_debug.so && \
  ln -sf /usr/glibc-compat/lib/libcrypt.so.1 /usr/glibc-compat/lib/libcrypt.so && \
  ln -sf /usr/glibc-compat/lib/libdl.so.2 /usr/glibc-compat/lib/libdl.so && \
  ln -sf /usr/glibc-compat/lib/libmvec.so.1 /usr/glibc-compat/lib/libmvec.so && \
  ln -sf /usr/glibc-compat/lib/libnsl.so.1 /usr/glibc-compat/lib/libnsl.so && \
  ln -sf /usr/glibc-compat/lib/libnss_compat.so.2 /usr/glibc-compat/lib/libnss_compat.so && \
  ln -sf /usr/glibc-compat/lib/libnss_db.so.2 /usr/glibc-compat/lib/libnss_db.so && \
  ln -sf /usr/glibc-compat/lib/libnss_dns.so.2 /usr/glibc-compat/lib/libnss_dns.so && \
  ln -sf /usr/glibc-compat/lib/libnss_files.so.2 /usr/glibc-compat/lib/libnss_files.so && \
  ln -sf /usr/glibc-compat/lib/libnss_hesiod.so.2 /usr/glibc-compat/lib/libnss_hesiod.so && \
  ln -sf /usr/glibc-compat/lib/libpthread.so.0 /usr/glibc-compat/lib/libpthread.so && \
  ln -sf /usr/glibc-compat/lib/libresolv.so.2 /usr/glibc-compat/lib/libresolv.so && \
  ln -sf /usr/glibc-compat/lib/librt.so.1 /usr/glibc-compat/lib/librt.so && \
  ln -sf /usr/glibc-compat/lib/libthread_db.so.1 /usr/glibc-compat/lib/libthread_db.so && \
  ln -sf /usr/glibc-compat/lib/libutil.so.1 /usr/glibc-compat/lib/libutil.so && \
  rm glibc-2.34-r0.apk && \
  wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-bin-2.34-r0.apk && \
  apk add glibc-bin-2.34-r0.apk && \
  rm glibc-bin-2.34-r0.apk && \
  wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-i18n-2.34-r0.apk && \
  apk add glibc-i18n-2.34-r0.apk && \
  rm glibc-i18n-2.34-r0.apk /etc/apk/keys/sgerrand.rsa.pub && \
  gunzip --keep /usr/glibc-compat/share/i18n/charmaps/UTF-8.gz && \
  /usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8 && \
  rm /usr/glibc-compat/share/i18n/charmaps/UTF-8 /var/cache/apk/* && \
  apk del glibc-bin glibc-i18n gzip

COPY --from=jre-build jre-11 $JAVA_HOME
```

## 참고 URL
* [Alpine Linux용 glibc - sgerrand](https://github.com/sgerrand/alpine-pkg-glibc)
* [Java11, jlink and Docker](https://greut.medium.com/java11-jlink-and-docker-2fec885fb2d)
* [JRE를 쉽게 만들 수 있게 도와주는 온라인 툴](https://justinmahar.github.io/easyjre/)
* [Amazon Corretto JRE11 Dockerfile](https://github.com/corretto/corretto-docker/blob/d98afb2f45401891e62a0651499b929f8da823eb/11/jre/alpine/Dockerfile)
* [Apine Linux에서 glibc 사용하기](https://ssaru.github.io/2021/05/09/20210509-til_set_locale_in_alpine_linux/)