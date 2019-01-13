

![tyle](https://www.dropbox.com/s/wta60lspqd8hutc/tyle-SLO-3-1547387762.png?dl=1)
- 오늘 글은 유용하지만 잘 다뤄지지 않은 태블로 기능을 소개합니다.
- 데이터는 태블로 기본 데아터 'SuperStore'를 활용합니다. 태블로 환경은 2018.3 버전입니다.(주의:하위 단계에선 호환되지 않습니다.)
[`*` 로 시작하는 차원/측정값들이 이번 실습에 사용한 계산식입니다.] [실습 예제 파일 링크 다운](https://www.dropbox.com/s/fj6go5mr7uyp6kb/%5B%EA%B8%80%EB%98%90%5Dtimesize%20%ED%91%9C%ED%98%84.twb?dl=0)

## 날짜 블락 표현하기

날짜 데이터를 다루다보면 '시작날짜'와 '끝날짜'의 기간을 시각적으로 표현하고 싶을 때가 있습니다.
태블로에서는 자동으로 만들어주는 기능이 없지만 함수를 활용하여 이를 표현할 수 있습니다.
해당 문제에선 '주문날짜Order Date'와 '배송날짜Ship Date' 간의 시간 간격을 표현하였습니다.

![완성결과](https://www.dropbox.com/s/s0s0o8ugykyeyaw/tableau_1_result.png?dl=1)


1. 매개변수 생성하기
- `*START` 와 `*END` 매개 변수를 생성합니다.
![매개변수](https://www.dropbox.com/s/g2ifk8rkwfmewmb/tableau_2_variable.png?dl=1)


2. 함수식 설정
함수식 생성방법은 우측 [측정값/차원] 영역의 빈 부분을 우클릭후 [계산된 필드 만들기]를 누르시면 됩니다.
또, 함수식 결과에 따라 측정값(수치형)과 차원(문자형 혹은 논리형)으로 구분됩니다.

### 2-1. 측정값

'*Duration' : 시작과 끝 날짜를 계산하는 계산식
> ```avg(DATEDIFF( 'day' , [Order Date] , [Ship Date]))```

날짜가 아닌 분단위로된 시간단위가 필요하다면 아래 식으로 표현을 하는게 좀 더 범용적이다.

'*TIME_SIZE' :

>```avg(DATEDIFF( 'minute' , [Order Date] , [Ship Date])/60/24)```

![duration](https://www.dropbox.com/s/rv8rortthp5c8jl/tableau_duration.png?dl=1)

### 2-2. 차원

'*SET_START' : 기준이 되는 시작점
![차원_1](https://www.dropbox.com/s/ix702ywe2067itm/tableau_set_start.png?dl=1)

'*SET_END' : 기준이 되는 끝점
![차원_2](https://www.dropbox.com/s/xjjwg2ryaoy3lo5/tableau_set_end.png?dl=1)

'*TIME_TF' : 매개변수를 적용하는 True/False 조건식
![차원_3](https://www.dropbox.com/s/uy7uaxfgng4wkwb/tableau_time_tf.png?dl=1)



### 추가적 기능 
EDA를 해보면, 'ship mode'별 평균적으로 걸린 날짜가 나옵니다. 'First class'는 2일, 'Second Class' 4일 , 'Standard Class' 는 5일 이상 걸리는 것으로 나옵니다. 이중 5일 보다 오래 걸린 조건을 걸어서 혹시 오랜걸린 주문이 있다면 확인하는 작업을 해보았습니다.


'Over 5days?'
> '[DURATION] > 5'

![5days](https://www.dropbox.com/s/snf0239ftitc2qe/tableau_5days.png?dl=1)



4개월 동안 기간을 늘려 확인하면, 5일 이상 배송이 걸린 경우는 한달에 5~7건 정도로 자주 있지 않고 매우 적게 집계되었습니다. 지역별로 나누어서 보아도, 특별히 어느 지역에 편향되지 않다는 사실을 확인했습니다. 정상적인 배송 환경을 보여줍니다.

![5days_result](https://www.dropbox.com/s/drgd9wue9k0x09l/tableau_5days_result.png?dl=1)

다음 글에서도 다른 유용한 기능을 소개하도록 하겠습니다.