## AQE (Adaptive Query Execution)

### AQE가 왜 생겼을까? (Cost-Based Model의 한계점)

- Spark 2.0~ 는 Rule-Based로 Logical Plan을 최적화하는 방식과, Cost-Based로 Physical Plan을 최적화하는 2가지 방식으로 꾸준하게 개선되어 옴.
- Spark 2.xx까지는 Rule + Cost 기반으로 Optimize가 실행되었기에, 정해진 실행 계획에 따라 수행됨.
- Cost Based Model이 여러가지 한계점을 가지고 있음.
    - 데이터의 통계 정보가 오래되었거나, 최신 데이터와 맞지 않을 경우 부정확한 실행 계획을 생성할 수 있음.
    - 대규모 데이터셋에서 통계치를 정기적으로 갱신하는 작업은 Cost가 큰 작업임
    - UDF에서 Predicates(특정 조건)을 사용하는 경우, Spark가 정확한 통계를 낼 수 없음
    - 쿼리 힌트를 기반으로 최적화할 수 있는데, 데이터가 빠르게 변하면 최신 상태와 일치하지 않을 수 있음.

⇒ AQE가 나왔구나~!

### AQE란?

- SparkSQL에 도입된 최적화 엔진으로, 실행 중에 데이터의 런타임 통계를 기반으로 실행 계획을 동적으로 조정하여 쿼리 성능을 향상시키는 기능
- AQE는 Spark 3.xx에 도입되었으며 실행 시점에 데이터를 모니터링하고, 데이터의 특성에 따라 RunTime에 Logical Plan을 변경하는 작업을 수행함.
- Data Skew나 예측할 수 없는 크기의 데이터를 처리하는 상황에 유리함.
- Spark 3.2부터 Default Option임.

### Query Stage

- Shuffle Query Stage
    - 이 Stage는 Shuffle된 데이터를 Shuffle 파일로 실현함.
    - 주로 데이터가 재분배되어야 할 때 / 조인이나 집계 작업에서 많이 사용함.
- Broadcast Query Stage
    - 결과 데이터를 driver memory에 있는 배열로 실현
    - 작은 데이터셋을 broadcast하여 모든 executor에서 data를 local로 접근할 수 있도록 하여 네트워크 트래픽을 줄임

> Query Stage는 Physical Plan을 기준으로 생성되며, 여러 개의 Physical Plan 중 최적의 계획을 선택하여 실행함.

근데 실행 도중에 데이터 크기에 따라 최적화가 필요한 경우, 
**AQE가 실행되면 기존의 “고정된 계획”을 따르지 않고 Query Stage 단위로써 재평가한 후에, 필요시에 Physical Plan을 동적으로 변경한다.**
> 

### Parallelism on Reducer

- 병렬처리수준은 복잡한 쿼리 성능에 매우 중요하다.
- Reduce할 때 Job의 수가 너무 작으면, 하나의 Partition의 크기가 매우 커져 Executor 메모리가 OOM 될 수 있음.
    
    ⇒ 성능 저하 원인으로 이어짐
    
- Reduce할 때 Job의 수가 너무 많으면 오버헤드가 커져 일정 수준 이상으로부터는 오히려 성능이 감소함
- **CoalesceShufflePartitions**
    
    ![image.png](https://github.com/user-attachments/assets/a09c3b50-4c14-492c-9154-8da492016b16)
    
    - Spark가 Runtime에 데이터 크기에 따라 최적의 partition 수를 동적으로 조정
    - 인접한 작은 partition을 병합함으로써 작업이 이뤄짐.
    - executor가 수행하는 작업의 균형이 이뤄져 요청되는 I/O 작업 수가 감소함
    

### Join Strategy in Spark

- **ShuffleHashJoin**
    - Hash Table을 사용하여 Join을 수행하는 전략
    - 하나의 데이터셋에서 `Hash Table`을 만들고, 다른 데이터셋에서 추출한 Hash Table과 `Equi`(동등)한 record를 추출
    - 두 데이터셋을 모두 `Shuffling`하여, Join Key에 따라 데이터를 재분배 후 수행
- **SortMergeJoin**
    - Join Key를 기준으로 데이터셋을 merge하여 Join을 수행하는 전략
    - 두 데이터셋을 모두 `Shuffling` 해야 함.
- **BroadcastHashJoin**
    - 작은 크기의 데이터셋이 있을 때, 그 작은 데이터셋을 모든 노드에 `Broadcast`하여 Join을 수행하는 전략
    - 데이터를 `Shuffling`하지 않기 때문에 네트워크 비용이 절약됨.
- AQE에서는 SortMergeJoin을 BroadcastHashJoin으로 변경할 수 있음
    
    > 한쪽 데이터 셋의 크기가 작아지는 경우
    > 

### Shuffle이 발생하는 작업들

- GroupBy / GroupByKey
- reduceByKey / aggByKey
- Join
- Distinct
- Repartition / coalesce
- sortBy / sortByKey