### Hadoop

- 하둡이란 대규모 데이터 세트를 분산 저장하고 처리하는 오픈 소스 소프트웨어 프레임워크임.
- 빅데이터 처리에 특 화되어있으며, 페타바이트 단위의 데이터를 처리할 수 있다는 확장성(Scale-Up / Scale-Down)을 가지고 있음.
- 하둡은 기본적으로 4가지 구성요소로 이뤄지며, Hadoop Common, HDFS, Hadoop Yarn, Hadoop MapReduce로 나뉨

### Hadoop Architecture

![스크린샷 2024-07-15 오전 11.02.00.png](https://github.com/user-attachments/assets/1d6a591f-033f-41ea-bb25-0f3f1c0ce52a)

- 기본적으로 하둡은 `Master Node` / `Slave Node`로 나누어 관리하며, 해당 구조를 통해 작섭과 데이터를 분산처리하고 관리하는 역할을 진행함.
- Master Node
    - 마스터 노드는 시스템 전체를 관리하고 조정하는 역할을 진행.
    - NameNode와 ResourceManager가 마스터 노드에서 작동함
- SlaveNode
    - 슬레이브 노드는 실제 데이터 저장과 처리 작업을 수행함.
    - DataNode와 NodeManager가 슬레이브 노드에서 작동함.

### HDFS(Hadoop Distributed File System)이란?

- HDFS는 하둡 시스템에서 대용량 데이터를 저장하기 위해 설계된 분산 파일 시스템이며, HDFS는 크게 `NameNode`와 `DataNode`로 구성되어 있음.
- NameNode는 파일 시스템의 메타데이터(파일 이름, 디렉터리 구조, 레플리카 수 등)를 관리함.
- DataNode는 실제 데이터를 저장하는 노드이며 데이터를 저장 단위인 ‘**블록**’ 단위로 나누어 저장함. 또한, 해당 데이터를 다른 DataNode에 여러 개의 복제본(replica)을 저장해두어 데이터 손실을 방지함.
- **그럼 하둡에선 HDFS를 왜 채택했고, 왜 사용할까? (목표)**
    - Hardware failure
        
        > Commodity Hardware를 기준으로 작은(저성능의) 컴퓨터를 엄청 많이 사용하여 실패 시에는 다른 Hardware로 교체하여 failure를 처리함.
        > 
    - Streaming Data Access
        
        > 하둡에서 실행되는 애플리케이션은 범용 파일시스템에서 실행되는 애플리케이션이 대상이 아니라, 대용량의 데이터를 일괄 처리하기 위해 설계 되었음. (**높은 처리량**에 중점을 두고 개발됨)
        > 
    - Large Dataset
        
        > 기본적으로 대용량의 데이터를 저장하고, 처리하는 것에 목표를 두어 설계 되었음.
        > 
    - Simple Coherency Model
        
        > 한 번 작성한 파일에 대해서는 수정이 존재하지 않는다. (한 번의 쓰기 + 여러번의 읽기)
        데이터 일관성 문제를 단순화하고, 데이터 처리량을 향상시킬 수 있는 핵심임.
        > 
- **HDFS에서 NameNode와 DataNode 사이의 통신 시스템**
    
    ![스크린샷 2024-07-15 오후 3.13.31.png](https://github.com/user-attachments/assets/008af4da-b89b-4e84-97c9-b26e0c8c2f32)
    
    - `Heartbeat`
        - DataNode가 NameNode에 주기적으로 상태를 보고하는 매커니즘.
        - 3초마다 한 번씩 DataNode의 상태가 NameNode로 전달됨.
        - 일정 기간동안 Heartbeat를 받지 못하면 DataNode의 실패(시스템 다운) 상태로 판단함
    - `Replication`
        - 데이터의 신뢰성과 가용성을 보장하기 위해 HDFS에서 데이터 블록을 여러개 복사하여 저장하는 매커니즘임.
        - 기본적으로 3개의 복사본으로 저장되며, NameNode는 복사본 블록을 어디에 저장할지 결정하고, DataNode에게 명령을 보냄
        - 주기적으로 복제 상태를 확인하고, 부족한 복사본에 대하여 자동으로 복구함.
    - `Balancing`
        - 로드 밸런싱은 클러스터 내에서 데이터 블록을 고르게 분산시켜 성능을 최적화하는 매커니즘임.
        - DataNode간 데이터 언밸런싱이 감지되었을 경우, `balancer`라는 유틸리티를 통해 데이터 블록을 균형있게 재배치함.
    - 실제로 사용자가 NameNode에게 사용하고자 하는 데이터를 요청하면, NameNode는 사용자 요청에 따라 필요한 데이터가 담겨있는 블록의 메타데이터를 제공하며, 이를 활용하여 DataNode에 직접 접근하여 Read/Write를 진행함.

### Yarn (Yet Another Resource Negotiator)이란?

- Yarn은 하둡 클러스터의 리소스 관리를 담당하며, 작업 스케쥴링과 클러스터 리소스의 효율적 사용을 도와주는 역할을 진행함. Yarn은 크게 `ResourceManager`와 `NodeManager`로 구성됨.
- ResourceManager는 클러스터의 전체 리소스를 관리하는 역할을 담당함. (자원 할당 조정, 실행 관리)
- NodeManager는 각 노드에서 실행되는 리소스 관리 데몬임. (리소스 사용 모니터링 + 컨테이너에서 애플리케이션 실행 담당)

### Yarn Architecture

![Untitled](https://github.com/user-attachments/assets/a991f401-efa5-4d1b-a9aa-77e4969878ab)

- Yarn은 HDFS, MapReduce와 같이 Master / Slave 구조로 구성됨.
- `RM(Resource Manager)`
    - 리소스 매니저는 마스터 서버에 존재하며, 슬레이브 노드들의 상태를 모니터링하고, 자원 사용량을 관리하는 글로벌 스케쥴러임.
    - 적절하게 자원을 분배하고, 리소스 사용 상태를 모니터링함.
- `AM(Application Manager)`
    - RM에게 전달된 Job들의 유효성을 체크하고, 자원 할당을 위해 스케쥴러에게 넘겨줌
    - 여기서 스케쥴러는 말 그대로 자원 할당과 관련이 있을 뿐, 모니터링/추적을 지원하지 않으며 실패한 Container 작업에 대한 재시작 권한이 존재하지 않음.
- `NM(Node Manager)`
    - 노드 매니저는 슬레이브 서버에 해당하며, 각 슬레이브에서 하나의 데몬이 실행됨.
    - 컨테이너 단위로 애플리케이션을 실행하고, 각 상태를 스케쥴링함.
- `컨테이너(Container)`
    - NM이 어플리케이션 실행과 상태를 스케쥴링한다면, 컨테이너는 실제로 작업을 수행하는 객체임.
    - 하나의 서버에 여러개의 컨테이너가 실행될 수 있음. (여러 컨테이너를 NM이 관리함)
- `AM(Application Master)`
    - 애플리케이션 매니저는 하나의 애플리케이션을 관리하는 마스터 서버임.
    - ResourceManager가 신호를 보내면 슬레이브 노드 중 하나를 선택하고, 해당 노드에 속해있는 컨테이너 중 하나를 골라 AM으로 지정하는 과정이 수행됨.
    - 애플리케이션에 필요한 리소스를 스케쥴링하고, NM에게 컨테이너 실행을 요청함.

### Yarn 작동 순서

- 클라이언트가 ResourceManager에게 애플리케이션 실행 (Job) 요청
- ResourceManager가 NodeManager에게 Application Master 생성 요청
- 특정 Container에 Application Master가 할당되고, ResourceManager에게 Job Resource 요청
- ResourceManager는 작업을 수행할 때 필요한 컨테이너를 할당하고, Application Master에게 통보
- Application Master는 각 NodeManager에게 Container 할당을 요청
- NodeManager가 컨테이너를 시작하고, 작업을 시작함.
- Application Master가 컨테이너 작업들의 상태를 모니터링
- 작업 완료 후 Application Master에게 보고하면, 완료 결과를 Resource Manager에게 전달
- 리소스 해제 및 다른 애플리케이션 실행

### MapReduce란?

- 맵리듀스는 분산 데이터 처리 모델로, 대규모 데이터셋을 효율적으로 처리하기 위해 사용함.
- Map은 입력 데이터를 Key-Value 쌍으로 Mapping하고, 분산된 여러 노드에서 병렬로 처리함.
- Reduce는 Map 단계에서 생성된 중간 결과를 Merge/Sort하고 정리하여 최종 결과를 생성함.