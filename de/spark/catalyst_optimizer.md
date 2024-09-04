## Catalyst Optimizer

### Catalyst optimizer가 무엇일까?

![image.png](https://github.com/user-attachments/assets/0d43daf0-428a-49b8-85b4-771fa3b6c03d)

- Spark SQL에서 핵심적인 역할을 하는 Rule-Based 엔진임.
- 사용자가 입력한 `Logical Plan`을 최적화된 `Physical Plan`으로 변환하여 쿼리 실행을 효율적으로 만드는 작업을 수행함.
- 사용자 입장에서는 어떻게 최적화가 수행되는지는 모르지만, Spark Context가 알아서 가장 효율적인 방식으로 최적화를 수행함.

### 수행 방법

- **분석 (Analysis)**
    - Unresolved Logical Plan을 생성함.
    - SparkSQL엔진이 쿼리를 위한 `AST`를 생성.
- **논리적 계획 생성 (Logical Plan Generation)**
    - 위 생성한 AST를 Logical Plan으로 변환하는 작업 수행
    - 구체적인 실행 방법보다는 **무엇을 할 것인가**에 초점을 두어 수행
- **논리적 계획 최적화 (Logical Plan Optimization)**
    - 표준적인 최적화 방법에 따라 Rule-base로 최적화를 수행함
    - **Constant Folding**
        
        > 상수 표현식을 미리 계산해서 표현식을 단순화한다.
        > 
    - **Predicate Pushdown:**
        
        > 필터 조건을 데이터 소스와 가능한 가깝게 배치시켜 데이터의 처리 양을 줄인다.
        > 
    - **Join Reordering**
        
        > 조인의 순서를 최적화하여, 중간 결과의 크기를 최소화하고 쿼리 속도를 올린다.
        > 
    - **Projection Pruning**
        
        > 쿼리에 불필요한 열을 제거하여 I/O에 발생하는 cost를 줄인다.
        > 
- **물리적 계획 생성 (Physical Plan Generation)**
    - Logical Opmization 결과를 Physical로 변환하는 작업 수행
    - 실제로 어떻게 수행할지에 초점을 두며, 구체적인 데이터 처리 방식을 결정함.
- **물리적 계획 최적화 (Physical Plan Optimization)**
    - 여러 Physical plan을 비교하여 가장 cost가 적은 작업을 선택하는 방식
    - 여기서 `cost model`을 활용하여 CPU, I/O와 같은 필요한 자원량을 추정함.
    - **Data Size (데이터 크기)**
        
        > 데이터 크기 (레코드 수 와 파티션 크기)에 영향을 받음.
        > 
    - **Operation Type (연산 유형)**
        
        > 데이터 셔플링이 네트워크 간 I/O를 발생시키기에, Cost가 큰 연산임
        > 
    - **Data Distribution (데이터 분포)**
        
        > 데이터 스큐 현상이 있으면, 일부 노드에 작업 과부하가 걸릴 수 있음.
        > 
    - **Join Stratege (조인 전략)**
        
        > 사용되는 조인 전략에 따라 비용이 크게 달라짐.
        Broadcast / Sort-Merge …
        > 
    - **Number of Stages (스테이지 수)**
        
        > Spark에서 Stage는 논리적 단계로 나눠지는데, 스테이지가 많아질수록 작업 스케쥴링과 실행에서 오버헤드가 많이 발생함.
        > 
- **코드 생성 (Code Generation)**
    - 선택된 Physical Plan은 Java Bytecode로 컴파일됨.
    - 해당 Bytecode는 Spark의 Runtime 엔진에 의해 실행됨.

### Whole-Stage Code Generation

- 가장 Optimize한 Physical Plan을 **`단일 Java 함수`**로 변환하여 성능을 향상시키는 작업
- CPU 명령어 수, 데이터 이동, 함수 호출 오버헤드 등을 최소화시켜줌.
    
    

### DPP (Dynamic Partition Pruning)

- SparkSQL에서 성능 최적화를 위해 사용되는 기술
- **Predicate Pushdown + Broadcast Hash Join**
- 특정 필터 조건에 따라 필요한 파티션만 동적으로 선택하여 처리하는 방식

### DPP 동작 방식

- **Small Table Filtering and Hash Table Generation**
    - 작은 테이블이 쿼리되고 필터링됨.
    - 필터 조건에 맞는 데이터에 대한 **해시 테이블**이 생성됨
- **Hash Table Broadcast**
    - 생성된 해시 테이블을 기반으로 broadcast 변수를 활용하여 클러스터의 모든 Executor에 broadcast를 수행
    - 각 Executor는 Local에서 필터 조건을 적용할 수 있게 됨.
- **Dynamic Filter 적용**
    - 실행 시점에서 Spark의 Physical Plan이 조정되어, 큰 테이블에 동적 필터를 적용함.
    - 이 동적 필터는 작은 테이블에서 적용된 필터 조건을 바탕으로 내부적으로 생성된 서브쿼리로, 팩트 테이블의 스캔 시 불필요한 파티션을 건너뛰게 함.
- **Partition Pruning**
    - 큰 테이블을 스캔할 때, 조건에 맞지 않는 파티션은 아예 스캔하지 않음
    - 이로 인해 처리해야 할 데이터 양이 크게 줄어들고, 쿼리 성능이 크게 향상됨.
    

### DPP 예시

- 판매 기록(Sale)을 저장하는 테이블과 날짜 정보(Date)를 저장하는 테이블이 존재함.
- Date 테이블에 접근해서 2024.01에 해당하는 날짜만 필터링하고, 결과가 해시 테이블로 저장
- 이 해시 테이블이 Broadcast되어 모든 Executor가 필터 조건을 local에서 사용할 수 있음
- 큰 테이블인 Sale을 스캔할 때, 동적 필터를 활용해서 2024.01에 해당하는 partition만 읽음
    
    **⇒ 데이터 처리량을 감소시킬 수 있으며, 성능 및 리소스를 최적화할 수 있음.**
    

### DPP 제한 사항

- 큰 테이블이 Join Key column 중에 하나를 기준으로 partitioning 되어있어야 함.
- 작은 테이블이 Hash Table이기 때문에, DPP는 기본적으로 Equi-Join에만 작동함
- 스타 스키마 모델에서 최적의 성능을 발휘
    
    > 스타 스키마는 **하나의 큰 팩트 테이블이 여러 작은 차원 테이블과 조인되는 형태**로, 이 구조가 별 모양처럼 보여서 "스타 스키마"라고 불립니다.
    >