## W1M2 팀 활동 요구 사항

**이해하기 어려웠던 내용**

1.  `{}` Represents any escaped character: Oracle에서만 제공하는 연산 방식
    - `%` 과 같은 문자를 그대로 사용하기 위해 Escape문자를 사용하는데 그 문자를 사용자 지정해줄 수 있음
2. SQL like & SQL wildcards의 차이점이 무엇인지?
    - `~ WHERE ~ LIKE '패턴';` 패턴에 wildcards가 들어가기 때문에 LIKE은 반드시 wildcards랑 같이 써야함
3. 서브 쿼리가 복잡해지면 그 구조를 잘 파악하기 어려웠음.
    - 서브쿼리를 실행하여 하나의 값이나 하나의 컬럼을 반환하여 메인쿼리의 컬럼 값과 비교 연산할 수 있다는 것을 알게 됨.
    - 쿼리 실행의 우선순위를 잘 따라가면서 천천히 생각해보고, 각 단계별로 끊어서 중간 산출물을 계속해서 파악해봄
4. 쿼리문의 실행 순서가 헷갈렸음
    - FROM > WHERE > GROUP BY > HAVING > SELECT > ORDER BY > LIMIT 순서로 진행

**이해는 했지만, 추가적으로 궁금했던 내용**

1. betweeen을 다를 표현으로 바꿀 수 있을까?
    
    ```sql
    col BETWEEN a AND b  ≡  col >= a AND col <= b
    ```
    
2. Sqlite는 Date Type을 따로 가지고 있지 않은가? 
    
    가지고 있지 않음.
    
    내부적으로  `TEXT` (`YYYY-MM-DD`), `REAL` (Julian), `INTEGER` (Unix timestamp)로 저장.
    
3. MySQL과 같은 다른 DBMS에서 제공하는 Date Type은 String Type과 어떤 점이 다를까?
    - 형식이 보장됨
    - 자동 정렬/비교가 정확하게 지원됨
    - 시간대(TIMEZONE) 등도 내부적으로 관리 가능
4. 아래 질의는 문자열 덧셈인가? 만약에 열 자료 타입이 문자열이 아니라면 오류가 발생하는가?
    
    ```sql
    SELECT CustomerName, Address + ', ' + PostalCode + ' ' + City + ', ' + Country AS Address
    FROM Customers;
    ```
    
    문자열이 아닌 열과 `+`를 쓰면 산술 연산이 발생하거나 오류 발생.
    
5. Having는 Group By와 의존 관계인가? 독립적으로 사용할 수는 없는가?
    
    HAVING은 GROUP BY 이후 집계 결과에 조건을 거는 절
    
     독립적으로 사용할 수는 있으나 WHERE로 대체 가능하기에 의미가 없음
    
6. In과 EXIST의 차이가 무엇인지? 각각 어떨 때 사용하는지?
    
    
    | 항목 | `IN` | `EXISTS` |
    | --- | --- | --- |
    | 의미 | 특정 값이 서브쿼리 결과 중 하나인지 확인 | 서브쿼리에 행이 존재하는지만 확인 |
    | 성능 | 소규모 데이터에 유리 | 대규모 서브쿼리에 유리 |
    | 서브쿼리 평가 | 한 번에 평가 | 외부값 기준 반복 평가  |
7. 아래 질의에서는 한 값이 여러 값을 가질 수 없기 때문에 서브 쿼리의 결과 값이 1개 초과일 경우 무조건 False 반환 
    
    ```sql
    SELECT ProductName
    FROM Products
    WHERE ProductID = ALL (
        SELECT ProductID
        FROM OrderDetails
        WHERE Quantity = 10
    );
    ```
    
8. `ANY`와 `SOME`이 같은 건지? → 같다.
9. `ANY`, `ALL` 연산자는 Sqlite에서 제공하지 않는데, 해당 기능을 다른 표현으로 대체하여 사용할 수 있을까?
    - `ANY` 연산자 정의인 “어떤 서브쿼리 값 중 하나라도 조건을 만족한다면 TRUE를 반환한다”를 대체하기 위해서 IN 을 사용하여 대체 가능
    - `ALL` 연산자도 `NOT EXISTS`를 사용하고, 서브쿼리에서 연산자를 반대로 사용하여, 해당 서브쿼리에 해당하는 값이 없다면 메인쿼리의 반환값으로 나올 수 있게 사용 가능


## W1M3 팀 활동 요구 사항

- wikipeida 페이지가 아닌, IMF 홈페이지에서 직접 데이터를 가져오는 방법은 없을까요? 어떻게 하면 될까요?
    - IMF 공식 홈페이지는 IMF Data 포털에 존재하는 World Economic Outlook 데이터베이스에서 나라별 경제적 수치 데이터를 제공한다. ([https://data.imf.org/en/Data-Explorer?datasetUrn=IMF.RES:WEO(6.0.0)](https://data.imf.org/en/Data-Explorer?datasetUrn=IMF.RES:WEO(6.0.0)))
    - 해당 데이터베이스에서 Indicator를 “Gross domestic product (GDP), Current prices, US dollar”로 설정하여 나라의 연도별 GDP 값을 csv 등의 형식으로 다운로드 할 수 있다.
    - 자동화된 코드로 주기적으로 데이터를 가져오기 위해서는
        - 데이터 업데이트 감지 로직을 구현하고
        - 새로운 데이터를 셀레니움 등의 동적 웹 크롤링 프레임워크를 활용하여 데이터를 추출하고 저장할 수 있다.
- 만약 데이터가 갱신되면 과거의 데이터는 어떻게 되어야 할까요? 과거의 데이터를 조회하는 게 필요하다면 ETL 프로세스를 어떻게 변경해야 할까요?
    - IMF의 GDP 데이터는 정기적으로 갱신되며, 갱신된 데이터는 기존 값을 덮어쓸 가능성이 있다. 시계열 추적과 같이 필요에 따라서는 이전 데이터를 보존하면서 새로운 데이터를 병행하여 저장하는 ETL 프로세스를 구성해야한다.
    - Type 2 Slowly Changing Dimensionns( SCD Type 2 ) : 레코드의 변경 이력을 완전히 보존하면서도 특정 시점의 유효한 데이터를 쉽게 조회할 수 있도록 한다.
        - 완전한 이력 보존 : 모든 데이터 변경 이력을 정확하게 추적할 수 있다.
        - 특정 시점 조회 용이 : start_date, end_date가 있어 특정 시점의 데이터를 쉽게 조회할 수 있다.
    - 테이블 스키마 변경: 기존의 테이블 스키마 (country, gdp) ⇒ 새로운 스키마 (id, country, gdp, year, snapshot_date)로 구성한다. 동일 국가, 연도에 데이터가 여러번 갱신될 수 있기 때문에 ETL 수행 일자로 snapshot_date를 추가한다.
        - 데이터 이력 추적 : 데이터가 시간이 지남에 따라 어떻게 변했는지 추적할 수 있게 한다.
        - 특정 시점의 데이터 조회 : 과거 특정 시점의 데이터를 "그때 그 모습 그대로" 조회할 수 있게 하기 때문에 시계열 분석에 필수적이다.
