## Airflow

### 정의

- 에어비앤비에서 만들었으며, 업데이트가 매우 빠름
- 스케쥴링 도구라 조금 무거울수도 있지만 거의 모든 기능을 제공하고 확장성이 좋음
- **코드로 작성된 데이터 파이프라인 흐름을 스케쥴링하고 모니터링하는 목적으로 사용됨.**

### 장점

- 파이썬 기반으로 스케쥴링 및 파이프라인 작성 가능
- 스케쥴링 및 파이프라인 목록을 볼 수 있는 웹 UI를 제공함
- BranchOperator라는 개념을 사용하면 특정 조건에 따라 작업을 분기할 수 있음
- 과거 특정 일시부터 현재시점까지 반복하여 DAG에 정의한 작업을 수행할 수 있음.
    
    굳이 귀찮게 shell script를 돌려서 데이터를 추출하고, 처리할 필요가 없어짐
    

### 기본 아키텍쳐

![Untitled](https://github.com/user-attachments/assets/7cb66d7f-c7a3-4caa-9013-818ba6c1a28f)

- 우리가 DAG 파일을 업로드하면, airflow scheduling이 DAG 프로세스를 확인해서 Metadata.db에 Serialize된 DAG정보를 저장한 후 작업 시간이 다 된 DAG를 Scheduling Queue에 올려 실행하는 방식

### 핵심 개념

- DAG
    - 방향 비순환 그래프의 약자로, Airflow에서 작업을 정의하는 방법임.
    - 작업의 흐름과 순서를 정의함.
    - **사실상 개발자가 작성하는 부분이 해당 영역임**
- Operator
    - Airflow 작업 유형을 나타내줌.
    - Bash / Python / SQL 등 다양한 Operator가 존재
- Scheduler
    - **Airflow에서 가장 중요한 컴포넌트임**
    - AirFlow의 핵심 구성 요소 중 하나로, DAG를 보며 현재 실행해야 하는지 스케쥴을 확인.
    - DAG 디렉토리 내에 있는 .py파일을 파싱하여 DB에 저장
        
        > Metadata.db에 .py로 정의된 DAG 파일을 정리
        ⇒ DAG Serialization 방법이라고 한다.
    - Executor를 통해 스케쥴링 기간이 된 DAG을 실행

### DAG 작성

- 기본적으로 DAG는 `Crontab`이라는 문법을 기준으로 작성함.
- Cron 표현식 사용 / * * * * * <command>
- **minute, hour, day of month, month, day of week 순서임**
- 모두 *을 쓰면 매분, 0 **** ⇒ 매시간, 00 *** ⇒ 매일, …. 표현식이 많이 존재함
- 단점
    - 재실행 및 알림이 불가능함 (에러 있을 때 몇 번더 실행해보고 안되면 알림을 준다던지, …)
    - 과거 실행이력 및 실행 로그를 보기 어려움
    - 복잡한 파이프라인을 만들기가 어려움
- **airflow Scheduling 짤 때, UTC기준으로 수집되기 때문에, 명심할 것**
datetime(2024, 8, 1) ⇒ 2024/08/01 00:00:00 (UTC) ⇒ seoul은 여기서 9 더해줘야 함.
기본적으로 시간은 Pedulum Type이라고 함.

### DAG 예시

```python
default_args = {
    'owner': 'airflow',
    'depends_on_past': False, # whether or not to run the task if the previous task failed
    'start_date': datetime(2024, 9, 3), # start date
    'end_date': datetime(2024, 9, 4), # end date
    'retries': 2, # retry the task
    'retry_delay': timedelta(minutes=1) # retry delay
}

dag = DAG(
    dag_id='naver_cafe_crawl', # dag_id
    default_args=default_args, # default_args
    description='naver_cafe_crawling every day', # description
    tags=['crawling'], # tags
    schedule_interval='0 0 * * *',  # every day at 00:00
    max_active_runs=1, # max_active_runs on parallel
    template_searchpath=['/opt/airflow/data'],
    on_failure_callback=task_fail_slack_alert,
    on_success_callback=task_succ_slack_alert,
)
```

- depends on past
    - 이전 일자에 실행했던 작업이 실패할 경우 작업을 종료해야하는지 결정하는 옵션
    - 특정 일시에 데이터가 없으면 안되는 상황이라면, True로 주어야 함.
- catchup (backfill)
    - 과거에 실패했던 작업을 재실행해야하는지에 대한 옵션
    - Start date를 2022.01.01로 설정하고 현재 시점에서 catchup=False를 두면 실행 시점을 기준으로만 데이터를 수집함.
- retries
    - Failover 몇번까지 수행할 지 (Default가 5번)
    - Lambda도 2번은 수행하는 것으로 알고 있음.
- tags
    - Airflow Web UI에서 DAG를 쉽게 찾을 수 있도록 Tagging하는 것
- Schedule interval
    - CRON 문법을 활용하여 언제 주기적으로 DAG를 실행할 지
    - AWS EventTrigger와 굉장히 비슷하다
- max_active_run
    - 한 번에 몇 개까지 병렬적으로 처리할지 결정하는 옵션
    - Airflow는 32개의 Task 병렬 처리를 Default로 가짐
    - 네이버 크롤링에 16 Task 병렬처리를 수행하면, 보호조치에 들어감.
- templete_searchpath
    - Airflow 내에서 특정 파일을 사용하고자 할 때, 경로를 잡아주는 역할을 수행
    - 해당 경로 내에 xxx.py를 탐색할 수 있도록 만들어줌
- on_fail/success_callback
    - 실패/성공 시에 callback function 호출이 가능함.

### Backfill

- 이전에 실패한 작업에 대하여(ex- 특정 커뮤니티의 특정 시간대 작업을 실패했다) 스케쥴링이 없는 시점에(큐가 널널한 시점에) 자동으로 재실행하도록 하는 기능
- DAG의 default_option을 추가할 때 ‘**catchup**’을 **true**로 지정해주면 적용된다.
- 파이프라인의 무결성을 유지하는데 도움될듯싶다.

### DAG 작성 시 주의할 점

원자성

- 하나의 Python_callable에 많은 작업을 정의하게 될 경우, 어디서 문제가 발생하는지 추적이 힘듬
- 특정 Case에서 실패한 작업임에도 불구하고 성공한 것처럼 보이는 경우가 있음
- 하나의 Python_Callable은 하나의 기능을 수행하도록 정의하는 것이 바람직함

멱등성

- 동일한 입력으로 계속 반복적으로 실행해도, 동일한 결과 값이 나와야 함.
- 과거의 특정 시점에서 실행하더라도, 현재 시점에 영향이 없어야 함.

### Jinja Templete

- Airflow에서 Default로 저장해둔 값
- {{~~~}} 형태로 call하여 값을 받아와 사용할 수 있음.
- 실행 시간(ds), 사용자 입력 parameter(params), 현재 Task 객체(task_instance), … 등을 활용 가능

### PythonOperator

- python_callable 인수에 Python Function을 넣어주면 실행된다.
- 어떤 함수더라도 Python 언어고, Callable하면, 호출이 가능하다.
- **kwargs
    - Keyword Arguments ⇒ Dictionary 형태로 key:value 값을 추출
    - Jinja Templete을 통해 얻을 수 있었던 정보들을 kwargs[’ds’] 이런 형태로 추출할 수 있음.

### Task간 데이터를 전달하는 방법

- XCOM
    - Airflow metastore를 활용하여 task간 결과를 쓰고 읽음.
    - MetaStore에 선택 가능한 개체(Pickable)를 저장.
    - Pickle형태라 Write는 느린데, Read가 빠름
    - 작은 데이터셋을 주고 받을 때 많이 활용함
    - Metadata.db에 저장되는데, 작업이 끝난 후에 자동으로 삭제되는 것이 아니기 때문에 주기적으로 지워주는 작업을 수행해야 함.
- DISK에 영구 저장
    - PostgreSQL나 MySQL 등 외부 DB에 결과 저장
    - 훨씬 안전하고 대용량의 데이터셋도 저장하여 관리할 수 있음.