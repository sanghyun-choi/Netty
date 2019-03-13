# Netty-
Netty Data Flow 흐름 이해

Netty는 두개의 EventLoopGroup을 가지며 다음과 같은 실행 Flow를 가진다.(기본 셋팅으로 설정 시)
# NioEventLoopGroup - bossGroup
EventLoopGroup은 여러 EventLoop(Thread 1개, Thread로 봐도 무방)로 구성
(bossGroup의 경우 ServerSocketChannel로 들어오는 신규 Channel을 관리 하므로, 1개로 보통 설정해도 무방하다.)
connection으로 들어온 clientChannel을 childGroup으로 regist를 수행한다.(이 후, 데이터 송수신 역할은 childGroup이 실행한다.)
# NioEventLoopGroup - childGroup
childGroup은 클라이언트의 데이터 송,수신을 담당한다.
보통 cpu 코어수 * 2로 구성되나, 설정 가능하다.
여러 EventLoop(Loop당 Thread 1개)로 클라이언트 Channel은 생명 주기동안 하나의 Loop에만 등록된다.
# EventLoopGroup - Flow
클라이언트로 부터 데이터가 수신되면 해당 channel의 schedule이 시작된다.
이 때, 처음 connection을 맺을 때 childGroup의 등록된 EventLoop(Thread)의 스레드와 동일하면 scheduledTaskQueue.add하여 작업을 실행 할 수 있다.
(scheduledTaskQueue에 저장된 task는 taskQueue로 자동으로 이동된다. 실질적인 작업은 taskQueue에 있는 작업을 꺼내어 실행한다.)
만약, 처음 등록되지 않은 스레드로 부터 실행이 되었다면 scheduledTaskQueue.add하는 행위의 신규 Task를 생성한 후, 등록된 EventLoop의 스레드의 taskQueue로 전달한다.
(전달 행위는 startThread()를 통해 등록된 Thread(EventLoopGroup)를 실행하여 taskQueue.add 한다.)
<br/>
# handler 적용
.handler는 boosGroup에 대한 이벤트 핸들러 등록으로 connection 생성시에 실행된다.
.childHandler는 childGroup에 대한 이벤트 핸들러 등록으로 데이터 송수신시에 실행한다.
(이 떄, 채널에 대한 pipeline에 핸들러 이벤트들이 등록되어 실행된다. @Sharable은 모든 핸들러가 하나를 공유한다.)
<br/>
# write 이해
write 메소드는 실제 ChannelFuture를 반환하여 void형태를 반환하며 비동기 형태로 실행된다.
실제로는 FileDescriptor를 읽어 데이터 송수신을 이용하기때문에 read같은 블록킹 방식은 필요없다.
실제로 컴퓨터 OS는 NIO 방식을 취하도록 소켓을 File형태로 관리한다.(OIO는 그저 프로그래머를 편하게 해주는 방식이였을 뿐)
그러므로 양방향(Duplexing)이 가능하다.
<br/>
# Future와 Promise차이
Future : 다른 context를 실행 후 결과를 바로 리턴받아 해당 결과를 get할때까지 sink하여 내 위주로 내가 데이터 가져오는 것이 가능
Promise : 다른 context를 실행하는 게 아니라, 실제 결과를 돌려주는 타이밍을 본인이 결정(CallbackListener??)
<br/>
# clinent와 server
두개 모두 EventLoop기반으로 동작하므로 코드가 

전체 내용 참조(https://www.slideshare.net/JangHoon1/netty-92835335)
