# async-practice : 비동기 프로그래밍 구현해보기


### 💡 비동기 프로그래밍 개념
- Async 한 통신
- 실시간성 응답을 필요로 하지 않는 상황에서 사용
    - ex) notification, email 전송(회원가입 후 축하합니다.), push 알림
- main thread가 task를 처리하는 게 아니라 sub thread에게 비동기로 task를 위임하는 행위라고 말할 수 있다.
- 그러면 여기서 main thread는 알아서 생성되고 자동적으로 처리할텐데 sub thread는 어떻게 생성하고 어떻게 관리를 해야할까?
    - spring에서 비동기 프로그래밍을 하기 위해선 ThreadPool을 정의할 필요가 있다.
    - 비동기는 Main Thread가 아닌 Sub Thread에서 작업이 진행한다.
    - java에서는 threadPool을 생성하여 async 작업을 처리한다.

### 💡 ThreadPool 설정 시 다양한 옵션들 (https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ThreadPoolExecutor.html)
- CorePoolSize : 최소 몇개의 threadPool을 갖고 있을 것이냐?
    - ex : 이를 5개로 설정하면 사용하든 안하든 5개 thread는 무조건 resource를 정의하고 있다.
- MaxPoolSize : 최대 thread를 몇 개까지 할당할것인가? (무한정 thread를 할당할 수 없다.)
- WorkQueue : 어떤 요청이 들어오면 모든 thread들이 바로 진행할 수 없다. workQueue에 요청이 들어온 많은 작업들을 담아놓는다. 현재 작업하고있는 것이 끝나면 다음 작업할 task를 가져온다.
    - 💡 **ex** **)** CorePoolSize가 3일때 request가 4개로 들어왔을 떄 여기서 바로 4번째 스레드를 생성하는 것이 아니라 WorkQueue에 담게 됩니다. WorkQueue 사이즈만큼 들어오는 요청들을 계속해서 담고, 그 WorkQueue의 사이즈만큼 요청이 쌓이면 그 다음 maxPoolSize만큼 thread를 생성하게 된다. **(순서중요)**
- KeepAliveTime : 최소와 최대의 thread 개수를 왔다갔다 할텐데 CorePoolSize 보다 더 많은 thread를 할당하게 되면 언젠간 반환하게 되는데 반환하는 조건으로 KeepAliveTime을 설정한다. 내가 지정한 시간만큼 thread를 사용하지 않으면 반환한다.
- unit : KeepAliveTime이 시간인지 분인지 초인지를 설정하는 단위

### 💡 ThreadPool 생성시 주의해야 할 부분
- CorePoolSize 값을 너무 크게 설정할 경우 side effect 고려해보기
    - 그 수 만큼은 자원을 점유하고 있기 때문에 너무 큰값을 설정하게 되면 이 threadPool은 잘 사용되지 않고, 혹 사용이 되더라도 그만큼의 Thread가 필요없는데도 불구하고 많은 스레드를 점유하고 있을 수 있기 때문에 적절하게 설정해야 한다.
    - 기본적으로 정의가 되어있는 ThreadPool이 있을 것이다. 그 값을 참고하여 설정을 하는 것이 일반적이다.
- IllegalArgumentException, NullPointException 의 발생을 주의해야한다. (아래의 조건 중 하나라도 성립하면 exception이 발생한다.)
    
    **IllegalArgumentException - if one of ter following holds**
    
    - corePoolSize < 0 : 메인 스레드가 스레드풀의 테스크를 위임하는데 스레드풀에 스레드가 없는 것이다.
    - keepAliveTime < 0 : 코어풀사이즈에서 스레드가 더 필요해서 새로운 스레드를 생성을 하려는데, 생성을 하자마자 바로 죽어야 된다는 것을 의미한다.
    - maximumPollSize ≤ 0 : 맥시멈풀사이즈가 0보다 작으면 코어풀사이즈가 0보다 작을 수 밖에 없다는 것을 의미한다. 이 스레드풀에는 단 1개의 스레드가 없기 때문에 exception 이 발생한다.
    - maximumPoolSize < corePoolSize : 미니멈이 맥시멈보다 큰 값을 가지는 것은 말이 안되기 때문에 익셉션 발생
    
    **NullPointerException - if workQueue is null**
    
    - workQueue가 null일경우에는 워크큐.push, put 이런 명령어를 사용할텐데 정의되어있지 않는 큐에다가 값을 넣게 되면 NPE가 발생한다.
    
### 💡 비동기 프로그래밍 주의사항
- 스프링 컨테이너에 등록 된 빈이 아니면 비동기가 되지 않는다. (@Async를 사용한 인스턴스를 빈을 주입하는 것이 아닌 인스턴스를 직접 생성한다. ex) new XXX();
- 이미 빈을 가져온 상태에서 내부메소드로 직접 접근하려고 하면 비동기가 되지 않는다.(해당 빈을 프록시 객체로 랩핑할 수 없기 때문이다.)
- Async 프로그래밍을 할 때에는 반드시 빈을 주입받아야 한다.
- 비동기 프로그래밍할 때에는 테스트 코드를 작성하거나 반드시 스레드 이름을 찍어보고 확인한다.
- @Async를 사용할 땐 public 으로 사용해야된다. (private를 사용하게 되면 컴파일 에러가 발생한다.)
