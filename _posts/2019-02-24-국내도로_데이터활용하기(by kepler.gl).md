

![tyle](https://www.dropbox.com/s/6j7a59v2b7nnxhs/tyle-SLO-2-1550997715.png?dl=1)


회사에서 사용중인 태블로BI는 지리데이터 분석(geospatial analysis)에 편리하지 못하다는 점이 있다. 그 이유 중 하나가 layer개념이 없어서인데 태블로에서 다른 성격의 지리정보값을 표현하려면 같은 위경도 컬럼으로 묶어야 한다. 이를 해결하기 위해 오픈소스 kepler.gl을 사용해보려 한다,

이번 글로 얻어갈 수 있는 정보는
1. TM좌표계를 위경도 좌표계로 변환하는 방법 (by QGIS)
2. Kepler.gl의 사용법

이다.

## 1. TM좌표계를 위경도 좌표계로 변환하는 방법

[QGIS 3.X 다운로드](https://www.qgis.org/ko/site/forusers/download.html)

[표준노드/링크 데이터셋 다운로드](https://nodelink.its.go.kr/data/data01_view.aspx?Fileno=124)

데이터셋 다운로드 후, `MOCT_NODE.shp`파일을 QGIS에서 실행한다.(추가로 관련 데이터셋 스키마는 해당 사용법에 상세하게 적혀있다)

먼저 해당 데이터셋에서 수정해야할 부분은 아래 두가지다.

    첫째, 인코딩(EUC-KR)
    둘째, 좌표계변환(TM to lat,lng GIS)

국내에서 사용하는 표준 지리데이터는 EUC-KR로 구성되어있는데 기본 인코딩은 UTF-8로 되어있다. 그대로 불러오면 한글이 깨지는 문제 발생하므로 아래 과정을 거쳐 해결하면 된다.

1. 해당 레이어 우클릭 [속성]-[데이터 소스 인코딩] 부분을 EUC-KR로 하기
![qgis_ecu_kr](https://www.dropbox.com/s/xlc91mewloa7s88/qgis_EUC_KR.png?dl=1)

2. [레이어]-[다른 이름으로 저장] 에서 인코딩을 'UTF-8' 로 선택후 내보내기

새로운 shp파일로 저장되면서 다시 실행을 하면 인코딩 변환이 완료되어있다.

다음 둘째, 좌표계변환은 조금 더 까다로운데 차근차근히 하면 된다. TM좌표계 설명은 해당 링크에서 확인하면 된다.(https://linuxism.ustd.ip.or.kr/833)


1. [레이어]-[레이어 좌표계 설정] 에서 경위도 좌표계를 확인한다.
![qgis1](https://www.dropbox.com/s/0uklmejmah8cqn5/qgis_1.png?dl=1)

2. 변환할 레이어 우클릭-[내보내기]-[객체를 다른이름으로저장]. 레이어 좌표계를 `WGS84 EPSG:4326`로 지정한다.
![qgis2](https://www.dropbox.com/s/98jm04ydlbwrfio/qgis_lat_lng1.png?dl=1)

3. 적용 후, 레이어 우클릭-[속성 테이블 열기]를 행렬표가 나오는데 1행만 복사한다.
![qgis3](https://www.dropbox.com/s/1d87c8so6506f25/qgis_lat_lng2.png?dl=1)

4. 구글 스프레드 시트를 연 후 붙여넣기 했을때, wkt_geom컬럼이 올바르게 나온다면 올바르게 변환 성공이다.
![qgis4](https://www.dropbox.com/s/9obah2rciu8ar0e/qgis_lat_lng3.png?dl=1)

5. 이제 [속성 테이블]에서 전체를 복붙해서 사용하면 된다.

여기서 반환하는 결과값은 WKT이며 이는 해당 링크에서 확인하면 된다. [WKT WIKIPEDIA](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)

NODE는 `Point()` LINK는 `MultiLineString()` 으로 구성되어있다.

ex.
  MultiLineString ((127.05705761225334527 37.56494538498876068, 127.05695711197159881 37.56467361470648569, 127.05679418939058678 37.56428345284011527, 127.0567619708897098 37.56424031782528772))

![wkt_geom](https://www.dropbox.com/s/6t67eizb2ntzkws/qgis_wkt.png?dl=1)


## 2. Kepler.gl의 사용법
http://kepler.gl/demo

kepler.gl은 우버에서 만든 지리분석 시각화툴이다. 별도의 설명이 필요없이 매우 쉬운 UI와 데이터 업로드시 자동으로 지리데이터를 매핑해주기 떄문에 분석시 매우 빠르고 유용하게 사용할 수 있다. QGIS에서 본 작업을 해도 되지만 초보자에게 QGIS는 적합하지 않다고 생각하며, 빠른 지리 분석을 하기 어렵다.

위에서 변환된 csv 데이터를 업로드하면 아래 사진처럼 자동으로 매칭해준다. 혹시나 인식을 못하게 되면, 수동으로 [ADD LAYER]-[Polygon] `geom` 데이터를 선택하면 된다.
![kepler](https://www.dropbox.com/s/lhv7iizt0hagvqp/kepler_2.png?dl=1)
노란점은 NODE, 하늘색실선은 LINK다.

좀더 확대해서 보면 실제 하늘색실선은 상향/하향 도로 두개의 실선으로 되어있고, 노란점은 링크들의 시작과 끝의 구분점이다.


여기서 인터렉션 기능을 사용하여서 마우스오버시 해당 정보를 표현할 수 있으며,
![kepler_2](https://www.dropbox.com/s/zkit7nyszqbblg4/kepler_4.png?dl=1)


필터기능을 통해 특정 지역의 지리정보만 확인할 수 있다.
![kepler_3](https://www.dropbox.com/s/2klsyhra9vfpxf9/kepler_5.png?dl=1)

그 외에도, [맵박스](https://www.mapbox.com/)와 연동하면 좀 더 다채로운 지도데이터를 표현할 수 있다.
![kepler_3](https://www.dropbox.com/s/lgtia0uvyv9u3ay/kepler_6.png?dl=1)


이상으로 국내 도로데이터를 인코딩, 위경도 좌표계 변환, kepler.gl 사용법을 알아보았다. 좀 더 데이터를 분석하기 위해서 QGIS나 [Bigquery GIS](https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions?hl=ko)를 사용하길 추천한다.

## 추가
서울 지역 샘플 데이터를 업로드 했습니다.
[QGIS 변환 데이터 링크](https://docs.google.com/spreadsheets/d/1_hlD50W-VTIGTF4NojV92LXj0I-fyB96B9Lt7C4vqwA/edit?usp=sharing)
