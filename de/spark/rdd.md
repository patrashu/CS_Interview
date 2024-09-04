## RDD (Resilient Distributed Data)

### RDD란 무엇인가?

- Spark에서 처리하는 기본 데이터 구조.
- 직독직해하면, “탄력적인 분산 데이터셋”임.
- 다수의 서버(클러스터)에 걸쳐 분산 형태로 저장된 요소들의 집합을 의미하며, 기본적으로 병렬 처리가 가능하고, 장애가 발생했을 때 스스로 복구될 수 있는 내성을 가지고 있음.

### RDD의 특징

- **데이터 추상화:**
    - 실제로는 여러 클러스터에 흩어져 있지만, 하나처럼 사용할 수 있음.
- **탄력/불변:**
    - 네트워크 장애가 발생했을 때, 스스로 복구할 수 있는 기능을 가지고 있음.
    - DAG 형태로 되어있기 때문에, C에서 장애가 발생하면 B, A로 되돌아가 재작업을 수행
- **Lazy Evaluation**
    - RDD는 연산이 필요할 때 까지(Action이 호출될 때 까지) 연산을 수행하지 않고, DAG 형태로 (RDD Lineage) 관리함.
    - Action (count, save …) method가 호출될 때, 비로소 DAG를 기반으로 실행이 됨.

### RDD는 왜 사용할까?

- 가장 기본이 되는 데이터 구조이기 때문에, Low Level에서 작업을 수행할 수 있다.
- DataFrame과 비교한다면 DataFrame은 고수준의 API를 제공하고, Catalyst optimizer 등으로 최적화가 잘 되어있지만, RDD는 그렇지 않다.
- 텍스트나, binary file들과 같이 비정형 데이터들을 다루기에는 RDD가 효율적일 수 있다.
    
    ⇒ 상황에 맞는 방법을 선택하면 문제가 발생하지 않는다.
    

### Transform

- Transform은 기본적으로 새로운 RDD를 생성한다.
- map / filter / flatMap 등 method를 호출해서 사용한다고 해도, 즉시 수행되지 않는다.
- Action이 호출되어야 비로소 Spark Context가 Job Scheduling을 시작하고, RDD에서 수행되는 여러 작업들을 Task 단위로 분할하여 Executor에게 전달한다.

### Action

- Action은 변환(Transformation)과 달리 새로운 RDD를 생성하지 않고, 계산의 최종 결과를 반환하거나 부수 효과(side effect)를 발생시킨다.
- `count`, `collect`, `saveAsTextFile` 등의 메서드를 호출하면 Spark는 DAG(Directed Acyclic Graph)를 기반으로 Job을 스케줄링하여 작업을 수행하고, 최종 결과를 제공합니다.
- Action이 호출되면, 이전에 RDD 계보(lineage)로 정의된 DAG가 실제로 실행됩니다. 이 과정에서 Spark는 DAG를 태스크로 분할하여, 각 태스크를 Executor에서 실행하게 합니다.

### Narrow / Wide Transformation

- **Narrow Transformation**
    - 각 입력 partition이 하나의 출력 patition을 생성
    - map, filter, flatMap이 대표적인 Narrow Transformation의 method임
    - 데이터가 동일한 노드에서 처리되기 때문에, Shuffle이 일어나지 않음
- **Wide Transformation**
    - 각 입력 partition이 여러 개의 출력 partition에 영향을 미치는 경우
    - groupBy, reduceByKey, Join 등이 여기에 속함
    - 데이터가 여러 파티션에 걸쳐 분산되기 때문에, Shuffle이 일어남.

### Spark Job 생성 방식

- **Driver 생성 및 리소스 요청**
    
    > Spark 애플리케이션이 시작되면, 가장 먼저 **Driver** 프로그램이 생성됩니다.
    그 후, Driver는 **클러스터 매니저**(예: `YARN`, `Mesos`, `Kubernetes` 등)와 통신하여 애플리케이션이 실행될 클러스터 리소스를 요청합니다. 클러스터 매니저는 작업 실행에 필요한 리소스를 할당합니다. (CPU, 메모리, Executor 수 등)
    > 
- **Spark Context 생성**
    
    > Driver는 클러스터와의 연결을 관리하고, 애플리케이션에서 필요한 환경을 설정하는 **SparkContext**를 생성합니다. 사용자 코드에서 변환(Transformation) 연산(예: `map`, `filter` 등)을 호출하면, 이러한 연산은 RDD로 표현됩니다. 이때, RDD는 실제로 데이터를 처리하지 않고, 변환 연산이 어떻게 수행될 지를 나타내는 논리적 실행 계획, 즉 **계보(lineage)** 정보를 유지합니다.
    > 
- **Action 호출**
    
    > **Action**이 호출되면, Spark는 이때까지 생성된 RDD 계보(lineage)를 기반으로 **DAG (Directed Acyclic Graph)**를 생성합니다. 이 DAG는 작업이 어떻게 실행될지에 대한 실행 계획을 표현합니다.
    
    Spark는 DAG를 분석하여 하나 이상의 **Job**으로 분할합니다. 각 Job은 여러 개의 **Stage**로 나뉘며, Stage는 여러 **Task**로 세분화됩니다. 이 Task들은 클러스터의 각 워커 노드에서 병렬로 실행됩니다.
    > 
- **실행**
    
    > 생성된 Job은 Spark의 스케줄러에 의해 실행됩니다. 스케줄러는 Job을 `Stage`로 나누고, 각 Stage를 `여러 Task`로 분할하여 Executor에게 할당합니다. 각 `Executor`는 할당된 Task를 실행하며, 이 과정에서 데이터가 `파티션` 단위로 처리됩니다. 필요한 경우, 셔플(shuffling)을 통해 데이터가 클러스터의 다른 노드로 이동하여 재분배됩니다.
    >