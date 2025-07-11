

### W1M3 ###

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
