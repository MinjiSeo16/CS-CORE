## ❓JVM의 구조에 대해 설명해보세요.
<img width="400" height="255" alt="image" src="https://github.com/user-attachments/assets/9d37f4e7-4f91-4ad7-bf46-543a40ba42e9" />

- **Class Loader** : .class파일(바이트코드)를 JVM 메모리로 적재합니다.
- **Execution Engine** : 바이트코드를 기계어로 변환 후 실행합니다.
- **Runtime Data Areas** : 프로그램 실행 시 사용하는 메모리 공간입니다.
   - Stack : 스레드마다 하나씩 생성, 메서드 호출
   - PC Register : 현재 실행 중인 명령 주소 저장
   - Native Method Stack : 네이티브 코드(C/C++) 실행 시 사용
   - Heap : new로 생성된 객체와 배열, 모든 스레드 공유, GC 대상
   - Method Area : 모든 스레드 공유, 클래스 정보, static 변수, 상수 저장
</br>

## ❓JDK와 JRE, JVM의 차이에 대해 설명해보세요.
<img width="337" height="159" alt="image" src="https://github.com/user-attachments/assets/dfe1c398-edfa-4d9e-999e-a15e0b2689dd" />

- **JDK** : 자바 통합 개발 환경 (JRE, JVM 모두 포함)
- **JRE** : 자바 실행 환경 (JVM + 실행에 필요한 라이브러리와 파일 포함)
- **JVM** : 자바 바이트코드를 OS에 맞게 기계어로 해석해주는 가상머신

**➕ JVM을 사용함으로써 얻을 수 있는 장점과 단점에 대해 설명해 주세요.** </br>
장점은 플랫폼 독립성입니다. 자바 프로그램은 한 번 작성하면 JVM만 설치되어 있다면 어떤 운영체제에서도 실행할 수 있는 Write Once, Run Anywhere 특징을 가집니다.
또한 가비지 컬렉션을 통해 메모리 영역을 자동으로 관리합니다.
단점은 JVM 위에서 코드가 실행되기 때문에 OS에 맞춘 네이티브 프로그램보다 실행 속도가 늘릴 수 있고
다중 상속이나 타입에 엄격하고 제약이 많습니다.

**➕ JVM과 내부에서 실행되고 있는 프로그램은 부모 프로세스 - 자식 프로세스 관계를 갖고 있다고 봐도 무방한가요?** </br>
프로세스는 OS로부터 자원을 할당받는 독립적인 실행 단위입니다. 
JVM은 하나의 프로세스 내에서 여러개의 스레드를 생성하여 자바 프로그램을 실행하기 때문에 부모-자식 프로세스 관계가 아닙니다.

*OS 하나당 JVM 여러개 띄울 수 있음 / JVM = 프로세스 / 하나의 프로세스 JVM 내부에서 여러 스레드 실행*

</br>

## ❓GC (Garbage Collection) 에 대해서 설명해보세요.
자바 메모리 관리 방법 중 하나로 JVM의 Heap영역에서 동적으로 할당했던 메모리 중 
필요없게 된 메모리 객체를 모아 주기적으로 제거하는 프로세스를 말합니다.
- **stop the world** -> JVM 애플리케이션을 멈추고, GC를 실행하는 스레드를 제외한 모든 스레드 작업을 중단 후
- **mark and sweep** -> 사용하지 않는 메모리를 제거하고
- 작업이 재개됩니다.
  
**➕ Minor GC와 Major GC** </br>
먼저 객체는 기본적으로 Young영역에 생성되는데, 이 영역은 Eden과 Servivor로 나뉘어져 있습니다.
Eden이 가득 차면 **Minor GC**가 발생해 사용하지 않는 객체를 제거하고 남은 객체만 Survivor로 옮겨집니다.
이렇게 여러번 Minor GC를 거쳐 살아남은 객체는 Old영역으로 옮겨지고, 장기간 사용되는 객체들이 쌓이게 됩니다. 
Old영역이 가득차면 **Major GC**가 발생합니다. 
Major GC는 전체 Old영역을 검사하고 압축까지 수행하기 때문에 시간이 오래 걸리고 Stop-the-word 시간이 길어져 성능에 영향을 줄 수 있습니다. 
그래서 Minor GC는 자주 일어나지만 가볍고, Major GC는 드물지만 무겁다는 차이가 있습니다.
