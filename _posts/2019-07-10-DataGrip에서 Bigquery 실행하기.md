---
title:  "DataGrip에서 Bigquery 실행하기"
subtitle: "Bigquery in DataGrip IDE"
search: true
categories:
  - SQL
tags: bigquery ,bigquery IDE, IDE, datagrip, IntelliJ
comments: true
---

![MAIN](https://www.dropbox.com/s/whlwhgjzxp12fyj/datagrip%280%20main%29?dl=1)


Mysql, Oracle 등 다양한 DBMS를 쓰면서, MysqlWorkBench나 SQLdeveloper 같은 IDE 적극적으로 활용하였다. 장인은 도구를 탓하지 않는다고 하지만, 좋은 툴이 업무 효율에 영향을 많이 주는 것은 부정할 수 없다. 빠르고 쉽게 데이터를 추출하는 니즈는 데이터를 다루는 사람이라면 누구나 공감을 할 것이다.

Bigquery는 아직 공식적으로 지원하는 IDE가 없어서 아쉬웠는데, 이런 아쉬움을 달래줄 꿀같은 IDE를 발견했다.
IntelliJ에서 Database IDE인 Datagrip으로 빅쿼리를 실행가능한 방법을 찾았고, 적용하는 과정을 공유해보려한다. 필자의 경우 현재도 업무에서 유용하게 사용하고 있는 중이다.

빅쿼리를 많이 쓰는 사람들에게 큰 도움이 되었으면 한다.
참고로 Datagrip은 *유료상품* 이다. 무료로 30일 사용가능하고, 가끔씩 하는 할인을 통해 구독을 하면 유용하게 사용이 가능하다. 단, 학생인 경우 무료로 사용가능하다!


우선 DataGrip의 장점은
1. 데이터 추출이 빠르고 간편하다.
- 웹의 경우, 데이터 추출에 제한이 많은 편이다.
16000행 제한수로, 그 이상이 될 시 구글드라이브 CSV다운을 받거나, 다른 테이블로 저장 후 다시 한번 추출을 해야하는 번거로움이 있는 반면, Datagrip으로 할시 행의 크기에 상관없이 클릭한번으로 데이터 추출이 가능하다.

2. 데이터 추출 후 정렬Sort, 검색Regrex 등이 매우 편하다.
- 데이터가 많아, 몇번의 페이지 이동으로 확인이 어려운 경우. OrderBy를 하거나, Where조건으로 또 다시 검색을 해야한다. 하지만 DataGrip으로 하면 이러한 문제들이 모두 해결된다.

3. 쿼리 저장 및 데이터 임시 저장
- 쿼리를 파일 자체로 저장이 가능하며, 쿼리들의 결과 데이터들을 고정핀을 걸어 Datagrip 종료시때까지 보관이 가능하다.

4. 커스텀한 테마 변경 가능
- IDE의 장점중 하나는 커스텀이지 아닐까 싶다.
웹형태의 빅쿼리는 흰배경만 제공하므로 어두운 계열의 배경을 좋아하는 개발자라면 Datagrip은 필수적인 선택이다.

그 외에도, 자동완성 기능, 다양한 데이터소스들을 한번에 관리하고 볼 수 있는 정말 많은 장점들이 있다.

- OS환경 : macOS Mojave(10.14.5)
- 빅쿼리 엑세스할 수 있는 GCP 계정필요. [IAM](https://console.cloud.google.com/iam-admin/iam) 에서 권한과 credential 확인하기! (예 : BigQuery -> BigQuery Data Editor / Viewer)

---

### 드라이버/IDE 설치하기

1. [Simbda JDBC 4.2 호환 드라이버](https://cloud.google.com/bigquery/partners/simba-drivers/?insvid=165c7bd5951c5482--1536654613358) 다운 (1.2.0.1000)
2. 최신 [DataGrip](https://www.jetbrains.com/datagrip/) 다운

### BigQuery 드라이버 설정하기

1. 압축 해제후 모든 `.jar`파일 `7`개를 Driver_file 에서 ‘+’를 누르고 'Custom JAR's로 추가한다. `com.simba.googlebigquery.jdbc42.Driver
`
2. 정상적으로 추가되면, class 부분, Dialect, URL_templates를  아래 사진처럼 설정 한다. 하단 URL_template를 설명하면, 접속하는 GCP의 정보를 입력받는 것이다. 아래는 템플릿만 작성 후, 실제 Data Source를 방금 추가한 Bigquery Drive로 생성하자.

![config1](https://www.dropbox.com/s/5ro2gpsu7i1zcsb/datagrip%281%29?dl=1)

`jdbc:bigquery://[Host]:[Port];ProjectId=[Project];OAuthType=[AuthValue];[Property1]=[Value1];[Property2]=[Value2];`

[HOST] = https://www.googleapis.com/bigquery/v2
[PORT] = 443
[ProjectId] = ‘프로젝트명'
[AuthValue] = 0
[Property] > 속성값. 필자는 여기서 Timeout만 추가했다.

다른 것 작성 필요없이, URL란에 아래 예시 처럼 입력하면 끝.

jdbc:bigquery://https://www.googleapis.com/bigquery/v2:443;ProjectId=PROJECTID;OAuthType=0;OAuthServiceAcctEmail==EMAIL;OAuthPvtKeyPath=KEYFILE;Timeout=10000

OAuthPvtKeyPath 란에는 console.cloud.google.com 에서 JSON으로 된Key를 다운받아

- `PROJECTID`는 Google 프로젝트 ID
- `OAuthType` = 0 (서비스 계정 승인용)

```
* OAuthType 참고
- OAuth 2를 사용하는 Google 사용자 계정 (OAuthType = 1)
- 사전 생성 된 액세스 및 새로 고침 토큰 사용 (OAuthType = 2)
- 환경에서 응용 프로그램 기본 자격 증명 사용 (OAuthType = 3)
```

- `EMAIL`은 서비스 계정과 연결된 이메일 주소
- `KEYPATH는` 키 파일의 절대경로 (ex: /Users/mike/Downloads/example.json)

'Test Connection (연결 테스트)'을 클릭하고 모든 것이 올바르게 설정되었다면 Datagrip 연결 성공

![config2](https://www.dropbox.com/s/o0qdup60esl46ny/datagrip%282%29?dl=1)

실제 콘솔창에서 돌려보았다.

![config3](https://www.dropbox.com/s/3xbs8bdra3bmprc/datagrip%284%29?dl=1)

잘 나옴!! 

----

+출처) [Setting up a BigQuery datasource in Jetbrains DataGrip](https://snowflake-analytics.com/blog/infrastructure/bigquery-datagrip-setup/) (Jetbrains DataGrip에서 BigQuery 데이터 소스 설정하기)


+에러)
[Simba][BigQueryJDBCDriver](100034) The job has timed out on the server. Try increasing the timeout value.
* 해당 에러가 나온다면, 서버의 timeout 시간을 늘리면 된다.
    * Timeout 설정 방법 링크
