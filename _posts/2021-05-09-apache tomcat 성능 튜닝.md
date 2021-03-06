---
layout: post
title: tomcat 성능 튜닝
featured-img: tomcattunning.jpg
categories: ['linux', 'tomcat']
author: oscar
---

# Apache tomcat 튜닝

## 1. Server.xml 튜닝

1) Listener 설정
```
<Listener className="org.apache.catalina.security.SecurityListener" checkedOsUsers="root" />
```
– tomcat이 기동할 때, root 사용자이면 기동을 하지 못하게 하는 옵션이다.<br>
– application server를 root 권한으로 띄웠다가 다음번에 다시 실행하려고 하면 permission 에러 <br>
– root 권한으로 서버가 실행되었기 때문에, 각종 config 파일이나 log 파일들의 permission이 모두 root로 바뀌어 버리기 때문에, 일반 계정으로 다시 재 기동하려고 시도하면, config 파일이나 log file들의 permission 이 바뀌어서 파일을 읽어나 쓰는데 실패하게 되고 결국 서버 기동이 불가능<br><br>  
  
  
2) Connector 설정
```
protocol="org.apache.coyote.http11.Http11Protocol"
```
– Tomcat은 네트워크 통신하는 부분에 대해서 3가지 정도의 옵션을 제공 (BIO,NIO,APR)<br>
– NIO는 Java의 NIO 라이브러리를 사용하는 모듈이고, APR은 Apache Web Server의 io module을 사용<br>
– C라이브러리를 JNI 인터페이스를 통해서 로딩해서 사용하는데, 속도는 APR이 가장 빠른것으로 알려져 있지만, JNI를 사용하는 특성상, JNI 코드 쪽에서 문제가 생기면, 자바 프로세스 자체가 core dump를 내면서 죽어 버리기 때문에 안정성 측면에서는 BIO나 NIO보다 낮다.<br>
– BIO는 일반적인 Java Socket IO 모듈을 사용하는데, 이론적으로 보면 APR > NIO > BIO 순으로 성능이 빠르지만, 실제 테스트 해보면 OS 설정이나 자바 버전에 따라서 차이가 있다. (Default는 BIO)<br><br>
  
```
acceptCount="10"
```
– 이 옵션은 request Queue의 길이를 정의한다.<br>
– HTTP request가 들어왔을때, idle thread가 없으면 queue에서 idle thread가 생길때 까지 요청을 대기하는 queue의 길이<br>
– 보통 queue에 메세지가 쌓였다는 것은 해당 톰캣 인스턴스에 처리할 수 있는 쓰레드가 없다는 이야기이고, 모든 쓰레드를 사용해도 요청을 처리를 못한다는 것은 이미 장애 상태일 가능성이 높다.<br>
– 그래서 큐의 길이를 길게 주는 것 보다는, 짧게 줘서, 요청을 처리할 수 없는 상황이면 빨리 에러 코드를 클라이언트에게 보내서 에러처리를 하도록 하는 것이 좋다.<br>
– Queue의 길이가 길면, 대기 하는 시간이 길어지기 때문에 장애 상황에서도 계속 응답을 대기를 하다가 다른 장애로 전파 되는 경우가 있다.<br>
– 순간적인 과부하 상황에 대비 하기 위해서 큐의 길이를 0 보다는 10내외 정도로 짧게 주는 것이 좋다.<br><br>

```
enableLookups="false"
```
– 톰캣에서 실행되는 Servlet/JSP 코드 중에서 들어오는 http request에 대한 ip를 조회 하는 명령등이 있을 경우, 톰캣은 yahoo.com과 같은 DNS 이름을 IP주소로 바뀌기 위해서 DNS 서버에 look up 요청을 보낸다.<br>
– 이것이 http request 처리 중에 일어나는데, 다른 서버로 DNS 쿼리를 보낸다는 소리이다.<br>
– 그만큼의 서버간의 round trip 시간이 발생하는데, 이 옵션을 false로 해놓으면 dns lookup 없이 그냥 dns 명을 리턴하기 때문에, round trip 발생을 막을 수 있다.<br><br>

```
compression="off"
```
– HTTP message body를 gzip 형태로 압축해서 리턴한다.<br>
– 업무 시나리오가 이미지나 파일을 response 하는 경우에는  compression을 적용함으로써 네트워크 대역폭을 절약하는 효과가 있겠지만, 이 업무 시스템의 가정은, JSON 기반의 REST 통신이기 때문에, 굳이 compression을 사용할 필요가 없으며, compression에 사용되는 CPU를 차라리 비지니스 로직 처리에 사용하는 것이 더 효율적<br><br>

```
maxConnection="8192"
```
– 하나의 톰캣인스턴스가 유지할 수 있는 Connection의 수를 정의 한다.<br>
– 이 때 주의해야 할 점은 이 수는 현재 연결되어 있는 실제 Connection의 수가 아니라 현재 사용중인 socket fd (file descriptor)의 수 이다.<br>
– TCP Connection은 특성상 Connection 이 끊난 후에도 바로 socket이 close 되는 것이 아니라 FIN 신호를 보내고, TIME_WAIT 시간을 거쳐서 connection을 정리한다.<br>
– 실제 톰캣 인스턴스가 100개의 Connection 으로 부터 들어오는 요청을 동시 처리할 수 있다하더라도, 요청을 처리하고 socket이 close 되면 TIME_WAIT에 머물러 있는 Connection 수가 많기 때문에, 단시간내에 많은 요청을 처리하게 되면 이 TIME_WAIT가 사용하는 fd 수 때문에, maxConnection이 모자를 수 있다. 그래서 maxConnection은 넉넉하게 주는 것이 좋다.<br>
– 이외에도 HTTP 1.1 Keep Alive를 사용하게 되면 요청을 처리 하지 않는 Connection도 계속 유지 되기 때문에, 요청 처리 수 보다, 실제 연결되어 있는 Connection 수가 높게 된다.<br>
– 그리고, process당 열 수 있는 fd수는 ulimit -f 를 통해서 설정이 된다. maxConnection을 8192로 주더라도, ulimit -f에서 fd 수를 적게 해놓으면 소용이 없기 때문에 반드시 ulimit -f 로 최대 물리 Connection 수를 설정해놔야 한다.<br><br>
  
```
maxKeepAliveRequest="1"
```
– HTTP 1.1 Keep Alive Connection을 사용할 때, 최대 유지할 Connection 수를 결정하는 옵션이다.<br>
– REST 방식으로 Connectionless 형태로 서비스를 진행한다면, Kepp Alive를 사용하지 않기 위해서 값을 1로 준다.<br>
– 만약에 KeepAlive를 사용할 예정이면, maxConnection과 같이 ulimit에서 fd수를 충분히 지정해줘야 한다.<br><br>

```
maxThread="100"
```
– 톰캣내의 쓰레드 수를 결정 하는 옵션이다.<br>
– 쓰레드수는 실제 Active User 수를 뜻한다.<br> 
– 즉 순간 처리 가능한 Transaction 수를 의미<br>
– 일반적으로 100 내외가 가장 적절하고, 트렌젝션의 무게에 따라 50~500 개 정도로 설정하는 게 일반적이다. 이 값은 성능 테스트를 통해서 튜닝을 하면서 조정해 나가는 것이 좋다.<br><br>
  
```
tcpNoDelay="true"
```
– TCP 프로토콜은 기본적으로 패킷을 보낼때 바로 보내지 않는다. 작은 패킷들을 모아서 버퍼 사이즈가 다 차면 모아서 보내는 로직을 사용한다.<br>
– 버퍼가 4K라고 가정할때, 보내고자 하는 패킷이 1K이면 3K가 찰 때 까지 기다리기 때문에, 바로바로 전송이 되지 않고 대기가 발생한다.<br>
– tcpNoDelay 옵션을 사용하면, 버퍼가 차기전에라도 바로 전송이 되기 때문에, 전송 속도가 빨라진다.<br>
– 반대로, 작은 패킷을 여러번 보내기 때문에 전체적인 네트워크 트래픽은 증가한다.<br>
– 예전에야 대역폭이 낮아서 한꺼번에 보내는 방식이 선호되었지만 요즘은 망 속도가 워낙 좋아서 tcpNoDelay를 사용해도 대역폭에 대한 문제가 그리 크지 않다.<br><br>


3) Tomcat Lib 세팅<br>
– 자바 애플리케이션에서 사용하는 라이브러리에 대한 메모리 사용률을 줄이는 방법<br>
– 일반적으로 배포를 할때 사용되는 라이브러리(jar)를 *.war 패키지 내의 WEB-INF/jar 디렉토리에 넣어서 배포 하는 것이 일반적이다.<br>
– 보통 하나의 war를 하나의 톰캣에 배포할 때는 큰 문제가 없는데, 하나의 톰캣에 여러개의 war 파일을 동시 배포 하게 되면, 같은 라이브러리가 각각 다른 클래스 로더로 배포가 되기 때문에, 메모리 효율성이 떨어진다.<br>
– 그래서 이런 경우는 ${TOMCAT_HOME}/lib 디렉토리에 배포를 하고 war 파일에서 빼면 모든 war가 공통 적으로 같은 라이브러리를 사용하기 때문에 메모리 사용이 효율적이고, 또한 시스템 운영 관점에서도 개발팀이 잘못된 jar 버전을 패키징해서 배포하였다 하더라도, lib 디렉토리의 라이브러리가 우선 적용되기 때문에, 관리가 편하다.<br>
– 반대로 war의 경우, war만 운영중에 재배포를 하면 반영이 가능하지만, lib 디렉토리의 jar 파일들은 반드시 톰캣 인스턴스를 재기동해야 반영되기 때문에, 이 부분은 주의해야 한다.<br><br>


4) JVM 튜닝<br>
– Java Virtual Machine 튜닝은 java 기반 애플리케이션에서는 거의 필수 요소<br><br>
- server<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 제일 먼저 해야할일은 JVM 모드를 server 모드로 전환하는 것이다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– JVM 내의 hotspot 컴파일러도 클라이언트 애플리케이션이나 서버 애플리케이션이냐 에 따라서 최적화 되는 방법이 다르다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 그리고 메모리 배치 역시 클라이언트 애플리케이션(MS 워드와같은)의 경우 버튼이나 메뉴는 한번 메모리에 로드 되면, 애플리케이션이 끝날 때 까지 메모리에 잔존하기 때문에 Old 영역이 커야 하지만, 서버의 경우 request를 받아서 처리하고 응답을 주고 빠져서 소멸되는 객체들이 대부분이기 때문에, New 영역이 커야 한다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 이런 서버나 클라이언트냐에 대한 최적화 옵션이 이 옵션 하나로 상당 부분 자동으로 적용되기 때문에, 반드시 적용<br><br>
- 메모리 옵션<br>
&nbsp;&nbsp;&nbsp;&nbsp;– JVM 튜닝의 대부분의 메모리 튜닝이고 그중에서도 JVM 메모리 튜닝은 매우 중요<br>
&nbsp;&nbsp;&nbsp;&nbsp;– Full GC 시간을 줄이는 것이 관건인데, 큰 요구 사항만 없다면, 전체 Heap Size는 1G 정도가 적당<br>
&nbsp;&nbsp;&nbsp;&nbsp;– New대 Old의 비율은 서버 애플리케이션의 경우 1:2 비율이 가장 적절<br>
&nbsp;&nbsp;&nbsp;&nbsp;– PermSize는 class가 로딩되는 공간인데, 배포하고자 하는 애플리케이션이 아주 크지 않다면 128m 정도면 적당하다. (보통 256m를 넘지 않는다. 256m가 넘는다면 몬가 애플린케이션 배포나 패키징에 문제가 있다고 봐야 한다.)<br>
&nbsp;&nbsp;&nbsp;&nbsp;– heap size는 JVM에서 자동으로 늘리거나 줄일 수 가 있다. 그래서 -Xms와 -Xmx로 최소,최대 heap size를 정할 수 있는데, Server 시스템의 경우 항상 최대 사용 메모리로 잡아 놓는 것이 좋다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 메모리가 늘어난다는 것은 부하가 늘어난다는 것이고, 부하가 늘어날때 메모리를 늘리는 작업 자체가 새로운 부하가 될 수 있기 때문에, 같은 값을 사용하는 것이 좋다.
    ```
    -Xmx1024m –Xms1024m -XX:MaxNewSize=384m -XX:MaxPermSize=128m
    ```
&nbsp;&nbsp;&nbsp;&nbsp;– 이렇게 하면 전체 메모리 사용량은 heap 1024m (이중에서 new가 384m) 그리고 perm이 128m 가 되고, JVM 자체가 사용하는 메모리가 보통 300~500m 내외가 되서 java process가 사용하는 메모리 량은 대략 1024+128+300~500 = 대략 1.5G 정도가 된다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 32 bit JVM의 경우 process가 사용할 수 있는 공간은 4G가 되는데, 이중 2G는 시스템(OS)이 사용하고 2G가 사용자가 사용할 수 있다. 그래서 위의 설정을 사용하면 32bit JVM에서도 잘 동작한다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 64 bit JVM의 경우 더 큰 메모리 영역을 사용할 수 있는데, 일반적으로 2G를 안 넘는 것이 좋다.(최대 3G), 2G가 넘어서면 Full GC 시간이 많이 걸리기 시작하기 때문에, 그다지 권장하지 않는다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 시스템의 가용 메모리가 많다면 Heap을 넉넉히 잡는 것보다는 톰캣 인스턴스를 여러개 띄워서 클러스터링이나 로드밸런서로 묶는 방법을 권장<br><br>
- OutOfMemory<br>
&nbsp;&nbsp;&nbsp;&nbsp;–자바 애플리케이션에서 주로 문제가 되는 것중 하나가 Out Of Memory 에러<br>
&nbsp;&nbsp;&nbsp;&nbsp;– JVM이 메모리를 자동으로 관리해줌에도 불구하고, 이런 문제가 발생하는 원인은 사용이 끝낸 객체를 release 하지 않는 경우<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 예를 들어 static 변수를 통해서 대규모 array나 hashmap을 reference 하고 있으면, GC가 되지 않고 계속 메모리를 점유해서 결과적으로 Out Of Memory 에러를 만들어낸다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;–Out Of Memory 에러를 추적하기 위해서는 그 순간의 메모리 레이아웃인 Heap Dump가 필요한데, 이 옵션을 적용해놓으면, Out Of Memory가 나올때, 순간적으로 Heap Dump를 떠서 파일로 저장해놓기 때문에, 장애 발생시 추적이 용이하다.<br>
    ```
    -XX:-HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./java_pid<pid>.hprof
    ```<br><br>
- GC옵션<br>
&nbsp;&nbsp;&nbsp;&nbsp;– Memory 옵션 만큼이나 중요한 옵션인데, Parallel GC + Concurrent GC는 요즘은 거의 공식처럼 사용<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 이때 Parallel GC에 대한 Thread 수를 정해야 하는데, 이 Thread수는 전체 CPU Core수 보다 적어야 하고, 2~4개 정도가 적당<br>
    ```
    -XX:ParallelGCThreads=2 -XX:-UseConcMarkSweepGC
    ```<br><br>
- GC로그옵션<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 서버와 JVM이 건강한지 메모리상 문제는 없는지 GC 상황은 어떻게 디는지를 추적하려면 GC 로그는 되도록 자세하게 추출할 필요가 있다. GC로그를 상세하게 걸어도 성능 저하는 거의 없다.
    ```
    -XX:-PrintGC -XX:-PrintGCDetails -XX:-PrintGCTimeStamps -XX:-TraceClassUnloading -XX:-TraceClassLoading
    ```
&nbsp;&nbsp;&nbsp;&nbsp;– 마지막에 적용된 TraceClassLoading은 클래스가 로딩되는 순간에 로그를 남겨준다.<br>
&nbsp;&nbsp;&nbsp;&nbsp;– 일반적으로는 사용하지 않아도 되나, OutOfMemory 에러 발생시 Object가 아니라 class에서 발생하는 경우는 Heap dump로는 분석이 불가능 하기 때문에, Out Of Memory 에러시 같이 사용하면 좋다.









