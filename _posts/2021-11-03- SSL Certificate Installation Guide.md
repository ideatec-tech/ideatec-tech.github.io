---
layout: post
title: SSL Certificate Installation Guide
featured-img: https.png
categories: ['SSL']
author: Dan
---

## SSL 인증서 설치하기 

### (Apache, Tomcat, Nginx, IIS7, WebtoB)
<br>

```
- SSL 인증서 설치를 하기전에 먼저 발급된 도메인 인증서 파일들이 필요합니다.
1. privateKey.key : 인증서 비밀키
2. 도메인.crt : 도메인 CA 인증서
3. AAACertificateServices.crt : Root CA 인증서
4. SectigoRSADomainValidationSecureServerCA.crt : Chain CA 인증서(1)
5. USERTrustRSAAAACA.crt : Chain CA 인증서(2)
- 추가로 위에 Chain CA 인증서 2개를 합친 Chain_RootCA_Bundle.crt 파일이 있을수 있습니다. 
```
<br>

## Apache 에서 SSL 인증서 설치하기.
<br>

```
- 먼저 Apache가 설치되어있는 Linux Server에서 Apache Config 파일(httpd.conf)을 수정합니다.
(기본적으로 conf 파일위치는 "아파치홈/conf"안에 httpd.conf로 존재합니다.)
```

![1](../image/hbshin/20211103/1.PNG)

```
- httpd.conf 파일에 들어와서 아래로 내리면 LoadModule이 여러개 있는데 (1)사진과 같이 
LoadModule ssl_module modules/mod_ssl.so 가 #으로 주석처리 되어있습니다. # 주석을 지워줍니다.

- 맨 아래 부분을 가보면 Include conf/extra/httpd-ssl.conf가 사진과 같이 잘 입력되어
있는지 확인하고, <IfModule ssl_module> ... </IfModule> 안에 내용과 함께 확인합니다.
```
```
- 다음은 Apache Config 파일(httpd-ssl.conf)파일을 수정합니다.
(파일 경로는 위에 "아파치홈/conf/extra"안에 httpd-ssl.conf로 존재합니다.)
```

![2](../image/hbshin/20211103/2.PNG)

```
- http-ssl.conf 파일에 들어오면 아래로 조금만 내려가보면 사진과 같이 
#SSLPassPhraseDialog builtin 이라고 되어있고 아래에 SSLPassPhraseDialog
"exec:/ssl/apache/ssl_pass.sh"라고 입력하면 됩니다.

- 다음은 bash 파일 제작입니다. 비밀번호 자동입력을 위해 exec:~ 으로 bash파일 제작
(위에 conf 파일 설정 경로상 Root 아래 ssl 디렉토리를 만들고 안에 
apache 디렉토리를 만들고 안에 vi를 통해 ssl_pass.sh 파일을 제작한뒤 bash를 입력합니다.)
```

![3](../image/hbshin/20211103/3.PNG)

```
- http-ssl.conf 파일에서 <VirtualHost _default_:443>을 찾고 아래 목록을 입력합니다.
1. DocumentRoot "http 기본설정과 동일한 디렉토리 경로" - 수정할필요없음.
2. ServerName "해당 서버의 도메인"
3. ServerAdmin "서버관리자 이메일"
4. ErrorLog "에러로그를 저장할 경로"
5. TransferLog "전송시 나타나는 에러를 저장할 경로"
6. SSLEngine on 으로 입력되어있는지 확인
```
![4](../image/hbshin/20211103/4.PNG)

```
- 마지막으로 http-ssl.conf 파일에서 사진과 같은 경로설정을 찾아서 입력합니다.
1. SSLCertificateFile "인증서 파일이 들어있는 경로 / 인증서파일 (ex : 도메인명.crt)"
2. SSLCertificateKeyFile "키 파일이 들어있는 경로 / 개인키파일 (ex : 도메인명.privateKey.key)
3. SSLCertificateChainFile "Chain파일이 들어있는경로 / 체인파일 (ex : Chain_RootCA_Bundle.crt")
- Chain 파일은 기본적으로 두개가 있을수도있고 Bundle 파일로 두개가 합쳐져 있을 수 있습니다.
```
![5](../image/hbshin/20211103/5.PNG)

```
- Apache 서버를 재기동하고 방화벽확인 및 443포트가 잘 올라가있는지 확인합니다. 
- 확인 후 브라우저에서 https://도메인 입력시 사진과 같이뜨면 SSL 인증서 설치 완료입니다.
```
<br>

## Tomcat 8.5에서 SSL 인증서 설치하기.
<br>

```
- tomcat 에서는 .crt 인증서 파일을 jks 인증서로 변환이 필요합니다.
또한 jks로 변환하기 위해서는 pem형식으로 변환이 필요합니다. 즉 crt > pem > jks 로 변환합니다.

- crt 파일 3개 (도메인.crt , Chain.crt , RootCA.crt)를 .txt로 확장자를 변경합니다.
```

![6](../image/hbshin/20211103/6.PNG)

```
- 그 다음 위 사진처럼 text파일로 열어서 안에 있는 KEY값을 전체복사하고 
빈 메모장에 3개를 한번에 합칩니다.
순서는 도메인 -> 체인 -> 루트 순서대로 복사붙여넣기해서 3개의 파일을 1개의 
빈메모장에 합쳐서  입력 후 이름.pem 확장자로 저장합니다. 

- BEGIN CERTIFICATE- ... -END CERTIFICATE- 이게 한 단락이며, 
총 도메인 1개 , 체인 2개 , 루트 1개가 나와야합니다.
```


![7](../image/hbshin/20211103/7.PNG)


```
- 이제 pem 형식의 파일과 privateKey.key 파일을 서버 톰캣에 
ssl 디렉토리를 만들고 그 안에 넣습니다.

- ssl 디렉토리 안에서 아래 내용을 입력합니다.

"openssl pkcs12 -inkey [privateKey.key파일명.key] -in [위에서 만든 pem 파일명.pem] 
-export -out [생성될 pfx파일명.pfx]"
(ex : openssl pkcs12 -inkey project1.ideatec.co.kr.privateKey.key 
-in cert.pem -export -out cert.pfx)

-입력 후 비밀번호 설정 및 확인을 하면 .pfx 파일이 생성됩니다.
```

![8](../image/hbshin/20211103/8.PNG)

```
- 이제 위와 같이 pfx 파일을 jks 형식으로 변환합니다. 입력 코드는 아래와 같습니다.

- keytool -importkeystore [생성한 pfx 이름] - srcstoretype pkcs12 -destkeystore 
[생성될 jks 이름] -deststoretype jks"

(ex)
"keytool -importkeystore -srckeystore cert.pfx -srcstoretype pkcs12 
-destkeystore project1.ideatec.co.kr.jks -deststoretype jks"
```

![9](../image/hbshin/20211103/9.PNG)

```
- 마지막으로 tomcat에 conf 디렉토리에 있는 server.xml 파일에 접속합니다.
- 아래로 내리다보면 Connector port 주석처리된 위에 사진과 같은 부분을 주석
해제하고 위와 같이 맞춰서 수정합니다.
- keystoreFile은 " jks파일이 있는 경로/jks파일 이름"
- keystorePassword 는 "jks파일 생성시 설정한 비밀번호"
```

![10](../image/hbshin/20211103/10.PNG)

```
- 톰캣 재기동 후 브라우저에서 https://"도메인" 을 통해 접속시 
자물쇠 모형과 https가 송출되면 완료입니다.
```
<br>

## NginX에서 SSL 인증서 설치하기.
<br>

```
- 먼저 nginx홈/conf/ 로 이동하여 nginx.conf 파일로 들어갑니다.
```
![11](../image/hbshin/20211103/11.PNG)

```
- listen 포트 : SSL 사용 포트 설정 (기본 포트는 443)
- ServerName : 해당 SSL 설치서버의 도메인
- ssl_certificate : 파일위치경로 / 도메인 CA 인증서 파일
- ssl_certificate_key : 파일위치경로 / 인증서 비밀키 파일

- 이 4가지를 확인해서 입력해주면 됩니다.
```

![12](../image/hbshin/20211103/12.PNG)


```
- 이제 nginx홈/sbin 위치에서 ./nginx 를 입력해서 재기동하고 브라우저에서 확인합니다.
```
<br>

## WebtoB에서 SSL 인증서 설치하기.
<br>

```
- WebtoB홈/conf 에 접속하여 http.m 설정파일에 접속합니다.
```
![13](../image/hbshin/20211103/13.PNG)

```
- 아래와 같이 설정을 추가합니다.

*URI

uri1         ........       vHostName = "아래 VHOST 의 NAME"


*VHOST

vhost_"이름"  HostName = "해당 서버의 도메인 (ex:project1.ideatec.co.kr)",
              DOCROOT = "[해당 호스트 루트 디렉토리],
              Port = "443"
              Logging = "log1",
              ErrorLog = "log2",
              SSLFLAG = Y,
              SSLName = "[SSL 설정 이름 (ex : ssl1)]

*SSL

ssl1     CertificateFile = "경로/도메인 CA 인증서파일" 
        - [ex:project1.ideatec.co.kr.crt]
         CertificateKeyFile = "경로/인증서 비밀키 파일" 
        - [ex:project1.ideatec.co.kr.privateKey.key]
```

![14](../image/hbshin/20211103/14.PNG)

```
- 저장 후 다시 conf 에 http.m이 있는 경로로 이동해서 wscfl -i http.m 입력
- 위와 같이 Successfully 뜨면 완료
- 그 후 wsdown
- :y
- wsboot 
- WebtoB wsdown , wsboot를 통해 재기동하고 그 후 브라우저에서 https://도메인주소 입력테스트
```
<br>

## IIS7에서 SSL 인증서 설치하기.
<br>

```
- 먼저 인증서 백업을 시작합니다.
```


![15](../image/hbshin/20211103/15.PNG)

```
- IIS7 인증서 관리접속을 위해 window검색창에 실행 검색 후 실행 창에서
 "mmc" 입력 후 콘솔창 실행
```

![16](../image/hbshin/20211103/16.PNG)

```
- MMC 콘솔 창에서 인증서 스냅인 추가 : 파일 > 스냅인 추가/제거 > 좌측에서 인증서 선택 후 
> 추가 버튼 클릭 > 컴퓨터 계정 선택 후 > 다음 버튼 클릭          
```
![17](../image/hbshin/20211103/17.PNG)

```
- MMC 콘솔 창에서 인증서 스냅인 추가 : 로컬 컴퓨터 확인 후 > 마침 클릭
우측(선택한 스냅인) 부분에 인증서 추가 여부 확인          
```

![18](../image/hbshin/20211103/18.PNG)

```
- 인증서 스냅인 추가 후 화면
```

![19](../image/hbshin/20211103/19.PNG)

```
- 인증서 > 개인용 > 인증서 선택 후 중앙 구역에 설치된 인증서 목록 중 백업할 인증서 우클릭 
> 내보내기 선택
```
![20](../image/hbshin/20211103/20.PNG)

```
- 인증서 내보내기 화면
```

![21](../image/hbshin/20211103/21.PNG)

```
- 암호를 설정하고, pfx 파일로 지정합니다.
```

![22](../image/hbshin/20211103/22.PNG)

```
- 내보내기 완료
```

```
- 이제 설치를 진행합니다.
```

![23](../image/hbshin/20211103/23.PNG)

```
- 인증서 스냅인을 추가합니다.
- MMC 콘솔 창에서 인증서 스냅인 추가 : 파일 > 스냅인 추가/제거 > 
좌측에서 인증서 선택 후 > 추가 버튼 클릭 > 컴퓨터 계정 선택 후 > 다음 버튼 클릭          
```

![24](../image/hbshin/20211103/24.PNG)

```
- MMC 콘솔 창에서 인증서 스냅인 추가 : 로컬 컴퓨터 확인 후 > 마침 클릭
우측(선택한 스냅인) 부분에 인증서 추가 여부 확인          
```
![25](../image/hbshin/20211103/25.PNG)

```
- 추가 후 화면
```

![26](../image/hbshin/20211103/26.PNG)

```
- 수신 받은 또는 보관한 인증서 파일 가져오기 (개인용 폴더에서 우클릭)
```

![27](../image/hbshin/20211103/27.PNG)


![28](../image/hbshin/20211103/28.PNG)

```
- 수신 받은 또는 보관환 인증서 파일 가져오기
```

![29](../image/hbshin/20211103/29.PNG)

```
- 중간 인증 기관 > 인증서 폴더에서 우클릭 > 모든 작업 > 가져오기 선택
```
![30](../image/hbshin/20211103/30.PNG)

```
- 인증서 가져오기 마법사 시작 > 중개 인증서 파일 선택 
(ChainCAn.crt, 중개인증서 수량 만큼 반복, 예] 중개인증서 2개 = 2회 반복)
```
![31](../image/hbshin/20211103/31.PNG)

```
- 가져오기 완료
```

![32](../image/hbshin/20211103/32.PNG)

```
- 중개 인증서 설치 확인 (인증서 제품별로 중개인증서 이름은 차이가 있을 수 있습니다.)
```

![33](../image/hbshin/20211103/33.PNG)

```
- 신뢰할 수 있는 루트 인증 기관 > 인증서 폴더에서 우클릭 > 모든 직업 > 가져오기 선택 
```
![34](../image/hbshin/20211103/34.PNG)

```
- 인증서 가져오기 마법사 시작
- 루트 인증서 파일 선택(RootCA.crt)
```

![35](../image/hbshin/20211103/35.PNG)

```
- 가져오기 완료
```

![36](../image/hbshin/20211103/36.PNG)

```
- 루트 인증서 설치 확인
```

![37](../image/hbshin/20211103/37.PNG)

```
- 설치 상태 확인
```

![38](../image/hbshin/20211103/38.PNG)

```
- 해당 사이트 SSL 인증서 바인딩

- IIS 관리자 실행 (관리자파일 : 시작 -> Windows 관리 도구 -> IIS(인터넷 정보 시스템) 관리자)
```

![39](../image/hbshin/20211103/39.PNG)

```
- SSL 인증서를 적용하려는 사이트 선택 후 우측 메뉴에서 바인딩 클릭 
```

![40](../image/hbshin/20211103/40.PNG)

```
- 사이트 바인딩 화면에서 추가 클릭 : https , 서버 Ip 주소 , 포트 ,SSL 인증서 선택 후 확인
```

![41](../image/hbshin/20211103/41.PNG)

```
- 사이트 재시작
```


![42](../image/hbshin/20211103/42.PNG)

```
- https:// 신청한 도메인 : 포트 로 접속하여 자물쇠 표시 및 https 통신 확인
```

<br>

## IBM SSL 인증서 설치하기.
<br>


![1](../image/hbshin/20211110/1.PNG)

```
- IBM istallation Manager 실행하고 다음 클릭
```

![2](../image/hbshin/20211110/2.PNG)

```
- 라이센스 계약의 조항에 동의함 체크 후 다음클릭 
```

![3](../image/hbshin/20211110/3.PNG)

```
- 패키지 설치할 경로 찾아서 등록하고 다음클릭
```

![4](../image/hbshin/20211110/4.PNG)

```
- 설치 클릭
```

![5](../image/hbshin/20211110/5.PNG)

```
- 설치 완료되면 Manager 다시시작 클릭
```
![6](../image/hbshin/20211110/6.PNG)

```
- 위와 같은 창이 뜨면 왼쪽 상단에 파일 클릭 후 환경설정을 클릭
```
![7](../image/hbshin/20211110/7.PNG)

```
- 저장소 추가 클릭해서 저장소를 추가합니다.
```
![8](../image/hbshin/20211110/8.PNG)

```
- 찾아보기 클릭
```
![10](../image/hbshin/20211110/10.PNG)

![9](../image/hbshin/20211110/9.PNG)


```
- IBM 설치시에 같이 포함되어있던 WAS_파일안에 repository.config 파일 선택
- 적용 > 확인 클릭
```

![12](../image/hbshin/20211110/12.PNG)

```
- 필요한 패키지는 IBM HTTP Server만 필요하기 때문에
서버만 체크 후 다음 클릭
```

![13](../image/hbshin/20211110/13.PNG)

![14](../image/hbshin/20211110/14.PNG)

```
- 다음 클릭
```
![15](../image/hbshin/20211110/15.PNG)

```
- 공유자원을 저장할 디렉토리 경로를 설정하고 다음 클릭
```
![16](../image/hbshin/20211110/16.PNG)

```
- HTTP Server 설치 디렉토리 경로를 지정합니다.
```
![17](../image/hbshin/20211110/17.PNG)

![18](../image/hbshin/20211110/18.PNG)

```
- 다음 클릭
```

![20](../image/hbshin/20211110/20.PNG)

```
- 설치 클릭해서 진행
```

![21](../image/hbshin/20211110/21.PNG)

```
- 설치 완료후 윈도우 서비스에 생성된것을 확인 후 실행
```

![22](../image/hbshin/20211110/22.PNG)

```
- 서비스 기동 후 hosts에 등록된 IP 확인하고 브라우저에서 테스트
```

![23](../image/hbshin/20211110/23.png)


```
- SSL 등록을 위해 시작 키 관리 유틸리티를 통해 키 변환작업을 합니다.
```

![24](../image/hbshin/20211110/24.PNG)

```
- 키 데이터베이스 정보를 불러와서 본인이 갖고 있는 파일을 열어줍니다.
```

![25](../image/hbshin/20211110/25.PNG)

```
- 파일 비밀번호를 입력합니다.
```

![26](../image/hbshin/20211110/19.PNG)

```
- 서명자 인증으로 변경하고 추가를 클릭합니다.
```
![26](../image/hbshin/20211110/26.PNG)

![27](../image/hbshin/20211110/27.PNG)


```
- 추가를 클릭하고 찾아보기를 클릭해서 저장된 root.crt파일 열기
```


![28](../image/hbshin/20211110/28.PNG)

```
- root.crt 파일 레이블 입력
- 동일한 방법으로 Chainca.crt파일도 진행
```

![29](../image/hbshin/20211110/29.PNG)

```
- 개인인증으로 다시 돌아옵니다.
```

![30](../image/hbshin/20211110/30.PNG)

![31](../image/hbshin/20211110/31.PNG)

![32](../image/hbshin/20211110/32.PNG)

```
- 위 첫번째 사진과 같이 개인인증안에 파일하나가 생깁니다.
- 오른쪽에 내보내기를 클릭합니다.
- 키 파일 유형은 CMS , 파일은 kdb파일입니다.
- 비밀번호를 설정하고 파일의 비밀번호를 숨김 체크를 하고 확인을 누릅니다.
- 위 세번째 사진과 같이 파일 3개가 설정했던 경로에 생성됩니다.
```

![33](../image/hbshin/20211110/33.PNG)

```
- 이제 생성된 파일을 HTTP Server에 넣습니다.
- 서버 재기동 후 실행하면 SSL이 적용되지 않습니다.
- error log를 살펴보면 메세지 하나가 발생합니다.
- IBM HTTP Server SSL0223E: SSL Handshake Failed, No certificate.
```

![34](../image/hbshin/20211110/34.PNG)

```
- error 해결방법은 cmd 창을 키고 본인의 HTTP Server파일경로까지 접속

- gskcapicmd -cert -setdefault -label clientcert -db 파일명.kdb 을
입력 후 hosts 파일에서 IP 확인하고 서비스 재기동 후 브라우저 테스트
```

<br>

## Oracle HTTP Server 에서 SSL 인증서 설치하기.
<br>

![1](../image/hbshin/20211112/1.PNG)

```
- ohs window 버전 다운받고 Dkis1 > install > win64 이동 후 setup
```

![2](../image/hbshin/20211112/2.PNG)

```
- 위와 같은 창이 뜨면 소프트웨어 갱신 설치를 진행합니다.
```

![3](../image/hbshin/20211112/3.PNG)

```
- 설치 및 구성 클릭후 다음 클릭
```

![4](../image/hbshin/20211112/4.PNG)

```
- 다음 클릭
```

![5](../image/hbshin/20211112/5.PNG)

```
- 오라클을 설치할 디렉토리 위치 지정 
```

![6](../image/hbshin/20211112/6.PNG)

```
- 다음 클릭
```

![7](../image/hbshin/20211112/7.PNG)

```
- 3개중에서 Oracle HTTP Server만 체크하고 나머지 해제 후 다음 클릭

- 설치 진행하고 완료
```

![8](../image/hbshin/20211112/8.PNG)

```
- 설치가 끝나면 cmd 창에서 오라클홈/instance1/bin 이동

- 이동 후 opmnctl.bat startall 서비스 실행
```

![9](../image/hbshin/20211112/9.PNG)

```
- 전자 지갑 만들고 비밀번호 생성
```
```
- C:\Middleware\oracle_common\bin 이 경로에 .p12 파일과 jks파일은 넣습니다.
- 이제 cmd 창에서 SSL 인증서 파일을 변환 하기위해서 orapki.bat 파일이 있는
C:\Middleware\oracle_common\bin 으로 이동하고 아래 키워드를 입력해서 변환

"wallet jks_to_pkcs12 -wallet C:\Middleware\Oracle_WT1 \instances\instance1\config\OHS\ohs1\keystores\default 
-pwd 패스워드 -keystore jks파일 -jkspwd 패스워드"

```
![10](../image/hbshin/20211112/10.PNG)

```
- 입력하면 변환된 파일이 
C:\Middleware\Oracle_WT1\instances\instance1\config\OHS\ohs1\keystores\default
이 경로에 생깁니다.

- 그 다음 서비스 재기동후 브라우저에서 확인
``