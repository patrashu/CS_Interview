## Spark 기본 개념

### What is Spark?

- Apache Spark는 대규모 데이터 처리 및 분석을 위해 설계된 오픈 소스 분산 컴퓨팅 시스템이다.
- Spark는 메모리 내 컴퓨팅 기능(In-Memory, Cache)을 활용하여 빠른 데이터 처리 속도를 제공하며, 대규모 데이터셋에 대한 복잡한 쿼리와 분석을 효율적으로 수행할 수 있다
- MapReduce를 진행할 때는 한 번의 작업이 수행될 때마다 HDFS에 File I/O를 진행해야하며, 대규모 데이터 처리에는 적합하지만 실시간 데이터 처리에는 한계점이 존재함.
    
    **⇒ Hadoop MapReduce Layer인 application layer의 대체자로써, spark를 사용함.**
    

### Pros / Cons

- In-Memory내에서 데이터 처리를 수행하여 디스크 기반의 MapReduce보다 빠름.
- Java, Scala, Python 등 여러 프로그래밍 언어를 지원함.
- 여러 모듈을 제공함으로써 다양한 데이터 처리 작업을 지원함. (Pandas, Streaming, Graphx …)
- HDFS에 저장하지 않고 메모리에 임시로 저장하여 활용하기 때문에 속도는 빠르지만, 어플리케이션의 규모에 따라 많은 메모리를 요구하기 때문에 MapReduce 작업보다 상대적으로 비쌈.
    
    ![스크린샷 2024-07-22 오후 5.03.34.png](https://github.com/user-attachments/assets/5a0c09ec-24ad-4163-a88b-2ca512e8f95e)
    

### Main Component(Module) in Spark

- Spark Core
    - Spark의 핵심 컴퓨팅 엔진(실행 엔진)으로서, 작업 스케줄링/메모리 관리/장애 복구 등을 담당함.
    - RDD(Resilient Distributed Datasets)의 생성 및 변환을 위한 API를 제공함.
    - Spark SQL, Streaming, GraphX 등의 상위 라이브러리를 지원함.
- SparkSQL
    - 구조화된 데이터를 사용하는 작업을 위한 Spark Module.
    - SQL이나 DataFrame API(pyspark.pandas)를 통해 구조화된 데이터를 쿼리할 수 있음.
- Spark Streaming
    - 실시간 데이터 스트리밍을 처리하기 위한 Spark Module.
    - Kafka, HDFS, 소켓 등에서 실시간으로 데이터를 수집하고 처리할 수 있음.
    - 데이터를 짧은 시간으로 분할하여 배치 처리함으로써 실시간 데이터 처리 파이프라임을 구현함.
- MLlib
    - 기계 학습 라이브러리로써, 다양한 기계 학습 알고리즘을 제공함.
    - 분류, 회귀, 클러스터링, 협업 필터링 등의 알고리즘을 포함.
- GraphX
    - 그래프와 그래프 병렬 계산을 위한 Spark API로서, 유연성이 뛰어남.
    - 그래프 데이터를 RDD로 표현할 수 있으며, 그래프 연산자와 알고리즘을 제공하여 그래프 데이터를 분석할 수 있음.
- Spark Driver
    - 사용자 애플리케이션의 메인 컨트롤러 역할을 함.
    - 사용자 프로그램이 실행되는 주체이며, AM 역할을 하는 Spark Context를 가지고 있음.
    - **Spark Core를 사용하여 RDD를 관리하고, 작업을 분산처리 하는 역할을 담당함.**
    - Driver가 Executor를 생성하는 역할을 함. (AM → 타른 datanode에게 일 시키는거랑 비슷)

### Cluster Manager Types

- Standalone
    - Spark 전용 클러스터 매니저로, 별도의 클러스터 매니저 없이 Spark 자체에서 관리함.
    - 간단하게 설정할 수 있어서 테스트 / 소규모 클러스터에 적합함.
    - Master가 클러스터 resource 관리 / job scheduling, Worker는 실제 작업 수행
- YARN
    - Hadoop의 resource 관리 / job scheduling을 담당하는 클러스터 매니저임.
    - 다양한 프레임워크를 지원하기 때문에 여러 작업을 동시에 진행할 수 있음.
    - Standalone보다 대규모 클러스터 환경에서도 효율적으로 작동할 수 있음.
- Mesos
    - 다양한 애플리케이션과 서비스 리소스를 관리할 수 있음.
- Kunernetes
    - 다양한 클라우드 환경과 On-Premise 환경에서 작동함.
    - 배포, 확장, 로드 밸런싱, 롤백 등을 자동화함으로써 운영 효율성을 높임.

### Deploy Modes

- Local Mode
    - **Spark는 기본적으로 JVM 위에서 작동함.**
    - 모든 Spark 컴포넌트가 단일 JVM 프로세스 내에서 실행하는 방식.
    - 디버깅에는 유용하지만, 단일 머신에서 실행되기 때문에 성능이 제한됨.
    - 작동 방식
        - Client가 Master JVM에게 `spark submit`을 보냄.
        - Master JVM이 Worker JVM에게 Driver를 실행하라고 요청.
        - 내부 노드 중 하나를 골라 Driver JVM으로 설정.
        - Driver JVM이 작업노드 Executor로 설정하고 실행하여 작업을 수행함.
            
            (Driver JVM과 Executor JVM이 서로 통신하며 작업 수행)
            
- Client Mode
    
    ![Untitled](https://github.com/user-attachments/assets/5bac489d-5f8f-4d64-b883-65093f44e9d1)
    
    - Spark Driver가 Client 머신에서 실행되고, Executor는 클러스터 내 노드에서 실행됨.
    - Client 내 Driver가 생기고, 클러스터 내 AM과 네트워크 통신하며 작업을 수행함.
    - 통신이 끊기면, 전체 어플리케이션이 중단되는 단점이 존재함.
    - 작동 방식
        - Client가 `spark submit`이 실행되면 Driver가 실행됨.
        - Driver가 클러스터 내 임의의 노드를 AM으로 설정
        - 클러스터 내 AM이 다른 노드를 Executor로 지정하여 실제 작업을 수행함.
        - **여기서 Driver가 AM에게 작업을 요청할 때, 네트워크 통신을 통해 요청 진행**
- Cluster Mode
    
    ![Untitled](https://github.com/user-attachments/assets/6d254cfe-f97c-45f8-8a0a-40cbfe14f628)
    

- Spark Driver가 클러스터 환경에서 실행되고, Executor는 클러스터 내 또 다른 노드에서 실행됨.
- 전체 어플리케이션이 클러스터 내에서 작동하기 때문에, Client보다 네트워크 의존성이 적음.
- 작동 방식
    - Client가 spark submit을 실행하면, 클러스터 매니저에게 전달됨.
    - 클러스터 매니저가 임의의 노드 중 하나를 선택하여 Driver 프로그램 실행
    - Driver는 클러스터 매니저와 통신하며 Executor를 요청함.
    - 할당된 Executor 노드는 실제 작업을 수행함.
    - Driver와 Executor간 통신을 진행하며 결과를 Driver에게 보고함.