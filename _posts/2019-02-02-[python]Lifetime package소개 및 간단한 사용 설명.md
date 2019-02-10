
![tyle](https://www.dropbox.com/s/5hsvwm67n5wppck/tyle-SLO-3-1549722095.png?dl=1)


- Lifetime package 소개 및 간단한 사용 설명.
- 2018 파이콘에서 하이퍼커넥트의 자동광고측정 사례를 보고 알게되었는데 실서비스에서도 적용을 하고 싶어서 공부를 해보았습니다.[(사례 링크)](https://www.pycon.kr/2018/program/83)

- 깃허브에 있는 소개글을 번역하고 직접 코드를 실행해보았습니다. https://github.com/CamDavidsonPilon/lifetimes
## Lifetime package 소개
Lifetimes 는 CLV를 계산할 수있는 Python 라이브러리다.

1. 사용자는 "살아있을 때(alive)" 상호 작용합니다.
2. 연구중인 사용자는 일정 기간 후에 "사망(die)" 할 수 있습니다.

"생존alive"과 "죽음die"는 생존 분석에서 사용되는 정의를 자유롭게 사용 하면 됩니다. 반복되는 개인이있을 때마다 Lifetimes를 사용하여 사용자의 행동을 이해할 수 있습니다.

## 활용 예시
1. 방문자가 귀하의 웹 사이트로 얼마나 자주 돌아갈 지 예측 (Alive = 방문, Die = 더이상 서비스에 접속하지 않을 때)
2. 환자가 병원으로 얼마나 자주 방문할지 예측 (Alive = 방문. Die = 환자가 새로운 도시로 이사했거나 사망했을때)
3. 사용 내역만 사용하여 앱에서 이탈 한 사용자를 예측 (Alive = 로그인, Die = 앱 삭제)
4. 고객의 반복 구매 예측. (Alive = 적극적으로 구매, Die = 제품에 무관심)


## 고객의 평생 가치 예측
P. Fader와 B. Hardie가 강조한 것처럼 고객 평생 가치 (CLV)에 대한 이해와 행동은 비즈니스 영업 노력에서 가장 중요한 부분입니다. 해당 내용을 자세히 설명한 강의가 있습니다.[(강의 영상 링크)](https://www.youtube.com/watch?v=guj2gVEEx4s)


## 설치
> pip install lifetimes

## 빠른 시작
아래의 예 cdnow_customers.csv는 datasets/디렉토리 에있는 것을 사용하고 있습니다.

> from lifetimes.datasets import load_cdnow_summary
data = load_cdnow_summary(index_col=[0])
print(data.head())

모든 모델에 대해 다음 명칭이 사용됩니다.

- `frequency` : 고객이 반복 구매 한 횟수를 나타냅니다. 즉, 총 구매 횟수보다 1 적은 금액입니다. 이것은 실제로 약간 잘못되었습니다. 고객이 구매 한 기간의 수입니다. 따라서 일을 단위로 사용하면 고객이 구매 한 일 수가 계산됩니다.
- `T` : 위의 데이터 세트에서 매주 선택한 시간 단위로 고객의 연령을 나타냅니다. 이것은 고객의 첫 구매와 연구 기간의 끝 사이의 기간과 동일합니다. (**해당 부분에서 age가 연령인지 확실치 않아서 좀 더 확인을 해봐야함!**)
- `recency` : 가장 최근에 구매 한 고객의 나이를 나타냅니다. 이는 고객의 첫 구매와 최신 구매 간의 기간과 동일합니다. (따라서 1회만 구매 한 경우 최신성은 0입니다.)


>        frequency  recency      T
    ID                           
    1           2    30.43  38.86
    2           1     1.71  38.86
    3           0     0.00  38.86
    4           0     0.00  38.86
    5           0     0.00  38.86



### 모델을 아웃 데이터에 맞추기
먼저 BG / NBD 모델을 사용하겠습니다 . 이 문서에서 살펴볼 다른 모델이 있지만 가장 간단합니다.
(참고) [BG/NBD모델 페이지](http://mktg.uni-svishtov.bg/ivm/resources/Counting_Your_Customers.pdf) (Beta Geometric Negative Binomial Distribution) model


    from lifetimes.datasets import load_cdnow_summary
    from lifetimes import BetaGeoFitter
    data = load_cdnow_summary(index_col=[0])
    print(data.head())

    # similar API to scikit-learn and lifelines.
    bgf = BetaGeoFitter()
    bgf.fit(data['frequency'], data['recency'], data['T'])
    print(bgf)
    ```
    <lifetimes.BetaGeoFitter: fitted with 2357 customers, a: 0.79, alpha: 4.41, r: 0.24, b: 2.43>
    ```
피팅 후, 여러 좋은 메소드와 속성을 피터 객체에 첨부합니다.

작은 표본 크기의 경우, 매개 변수가 어쩔 수없이 커질 수 있으므로 12배의 페널티를 추가하여 매개 변수의 크기를 제어 할 수 있습니다. `penalizer_coef` 모델의 초기화에서 양수 설정으로 구현됩니다. 일반적으로 0.001에서 0.1 정도의 penalizer가 효과적입니다.

<!-- 2357명의 고객 데이터를 통해 -->


## 빈도 / 날짜순 그래프 시각화
고객이 3주 동안 매일 구매했고, 몇 달 동안 고객으로부터 소식을 받지 못했습니다. 고객이 아직도 "살아있다"는 것은 무엇입니까? 꽤 작습니다. 반면에 한 분기에 한 번 구매하고 지난 분기에 구매 한 고객은 아직 살아있을 가능성이 큽니다. 이 관계를 시각적으로 표시 할 수 있습니다. `빈도 / 최근 값 행렬` 은 다음 기간에 인위적인 고객이 기대하는 거래 수를 계산합니다. 최근 거래가 성사 된 기간과 빈도(반복 거래 횟수)를 보여줍니다.)

    from lifetimes.plotting import plot_frequency_recency_matrix

    plot_frequency_recency_matrix(bgf)


![matrix](https://www.dropbox.com/s/9bebumra62ejk8f/Rw8PGcq.png?dl=1)

고객이 25 번을 구매하고 가장 최근 구매가 35주 (개인이 35주인 경우) 였을 때 가장 적합한 고객 (오른쪽 하단)임을 알 수 있습니다. 가장 차가운 고객은 오른쪽 상단에있는 고객입니다. 그들은 많이 빨리 구입했으며, 몇 주 동안 제품을 보지 못했습니다.

주위에는 아름다운 "꼬리"(좌측 상단부터 내려오는)가 있습니다 (5,25). 드물게 구입하는 고객을 대표하지만 최근에 다시 유입되었고 재구매할 가능성이 높습니다. - 아쉽지만 해당고객이 이탈하거나 재방문 사이에 있는지 확실하지 않습니다..

살펴볼 또 다른 흥미로운 매트릭스는 여전히 살아있을 확률입니다 .

    from lifetimes.plotting import plot_probability_alive_matrix

    plot_probability_alive_matrix(bgf)


![matrix2](https://www.dropbox.com/s/5ypazr27d81zauf/di6MTic.png?dl=1)

## 고객에게 랭킹 먹이기(Best to Worst)
다시 돌아가서 "다음에 구매할 때 가장 높은 예상 구매액"을 기준으로 낮은 순위로 매겨 봅시다. 모델은 내역을 사용하여 다음 기간에 고객의 예상 구매를 예측하는 메소드를 반환합니다.

    t = 1
    data['predicted_purchases'] = bgf.conditional_expected_number_of_purchases_up_to_time(t, data['frequency'], data['recency'], data['T'])
    data.sort_values(by='predicted_purchases').tail(5)

    """
           x    t_x      T  predicted_purchases
    ID
    509   18  35.14  35.86             0.424877
    841   19  34.00  34.14             0.474738
    1981  17  28.43  28.86             0.486526
    157   29  37.71  38.00             0.662396
    1516  26  30.86  31.00             0.710623
    """

26건의 구매를 한 고객이 다음에도 재구매할 것임을 알 수 있습니다.

## 모델 적합성 평가
우리는 예측할 수 있으며 우리는 고객의 행동을 시각화 할 수 있지만 모델이 맞습니까? 모델의 정확성을 평가하는 몇 가지 방법이 있습니다. 첫 번째는 적합 모델의 매개 변수로 시뮬레이션 된 데이터와 인공 데이터를 비교하는 것입니다.

    from lifetimes.plotting import plot_period_transactions
    plot_period_transactions(bgf)

![plot](https://www.dropbox.com/s/8ctty96j013pa9y/qlE4LDU.png?dl=1)

실제 데이터와 시뮬레이션 데이터가 잘 일치하는 것을 볼 수 있습니다. 이것은 모델이 끔찍하지 않다는 것을 증명합니다.

## 트랜잭션 데이터 집합을 사용하는 예
대부분의 경우, 보유하고있는 데이터 집합은 트랜잭션 수준에 있습니다. Lifetimes에는 트랜잭션 데이터 (구매 당 한 행)를 요약 데이터 (빈도, 최근 성 및 연령 데이터 세트)로 변환하는 유틸리티 함수가 있습니다.


    from lifetimes.datasets import load_transaction_data
    from lifetimes.utils import summary_data_from_transaction_data

    transaction_data = load_transaction_data()
    print(transaction_data.head())
    """
                  date  id
    0  2014-03-08 00:00:00   0
    1  2014-05-21 00:00:00   1
    2  2014-03-14 00:00:00   2
    3  2014-04-09 00:00:00   2
    4  2014-05-21 00:00:00   2
    """

    summary = summary_data_from_transaction_data(transaction_data, 'id', 'date', observation_period_end='2014-12-31')

    print(summary.head())
    """
    frequency  recency      T
    id
    0         0.0      0.0  298.0
    1         0.0      0.0  224.0
    2         6.0    142.0  292.0
    3         0.0      0.0  147.0
    4         2.0      9.0  183.0
    """

    bgf.fit(summary['frequency'], summary['recency'], summary['T'])
    # <lifetimes.BetaGeoFitter: fitted with 5000 subjects, a: 1.85, alpha: 1.86, b: 3.18, r: 0.16>


## 더 많은 모델 피팅
트랜잭션 데이터를 사용하여 데이터 세트를 교정 기간 데이터 세트 및 보류 데이터 세트로 분할 할 수 있습니다. 이것은 우리 모델이 아직 보지 못한 데이터에 대해 어떻게 작동하는지 테스트하고자 할 때 중요합니다 (표준 기계 학습 문헌의 교차 검증을 생각해보십시오). Lifetimes에는 다음과 같이 데이터 세트를 분할하는 함수가 있습니다.


    from lifetimes.utils import calibration_and_holdout_data

    summary_cal_holdout = calibration_and_holdout_data(transaction_data, 'id', 'date',
                                            calibration_period_end='2014-09-01',
                                            observation_period_end='2014-12-31' )   
    print(summary_cal_holdout.head())
    """
        frequency_cal  recency_cal  T_cal  frequency_holdout  duration_holdout
    id
    0             0.0          0.0  177.0                0.0               121
    1             0.0          0.0  103.0                0.0               121
    2             6.0        142.0  171.0                0.0               121
    3             0.0          0.0   26.0                0.0               121
    4             2.0          9.0   62.0                0.0               121
    """

이 데이터셋을 사용하여 `_cal`열을 맞추고 열을 테스트 할 수 있는`_holdout`이 있습니다.

![plot2](https://www.dropbox.com/s/7zy2q29l7uns6jn/LdSEYUwl.png?dl=1)

    from lifetimes.plotting import plot_calibration_purchases_vs_holdout_purchases

    bgf.fit(summary_cal_holdout['frequency_cal'], summary_cal_holdout['recency_cal'], summary_cal_holdout['T_cal'])
    plot_calibration_purchases_vs_holdout_purchases(bgf, summary_cal_holdout)


## 고객 예측
고객 기록에 대한 기본 사항으로, 향후 개인 구매가 어떻게 될지 예측할 수 있습니다.

    t = 10 #predict purchases in 10 periods
    individual = summary.iloc[20]
    # The below function is an alias to `bfg.conditional_expected_number_of_purchases_up_to_time`
    bgf.predict(t, individual['frequency'], individual['recency'], individual['T'])

    # 0.0576511



## 고객의 재방문 확률 시계열
고객 거래 내역을 감안할 때, 훈련된 모델에 따라 생존 가능성에 대한 시계열 확률을 계산할 수 있습니다. 예 :

    from lifetimes.plotting import plot_history_alive

    id = 35
    days_since_birth = 200
    sp_trans = transaction_data.loc[transaction_data['id'] == id]
    plot_history_alive(bgf, days_since_birth, sp_trans, 'date')


![plot3](https://www.dropbox.com/s/eknbkj6hkx5f9s1/y45tum4.png?dl=1)

## 감마 - 감마 모델을 사용하여 고객의 평생 가치를 추정
이 기간 동안 각 거래의 경제적 가치를 고려하지 않았으며 주로 거래의 발생에 초점을 맞추었습니다. 이것을 추정하기 위해 감마 - 감마 서브 모델을 사용할 수 있습니다. 하지만 먼저 거래별 경제적 가치 (profits or revenues)가 포함 된 거래 데이터로 요약 데이터를 작성해야합니다.

    from lifetimes.datasets import load_cdnow_summary_data_with_monetary_value

    summary_with_money_value = load_cdnow_summary_data_with_monetary_value()
    summary_with_money_value.head()
    returning_customers_summary = summary_with_money_value[summary_with_money_value['frequency']>0]

    print(returning_customers_summary.head())
    """
                 frequency  recency      T  monetary_value
    customer_id
    1                    2    30.43  38.86           22.35
    2                    1     1.71  38.86           11.77
    6                    7    29.43  38.86           73.74
    7                    1     5.00  38.86           11.77
    9                    2    35.71  38.86           25.55
    """
## 감마 - 감마 모델과 독립 가정
우리의 사용자 기반에 대한 CLV를 추정하기 위해 사용할 모델을 감마 - 감마 모델이라고 부릅니다.이 모델은 중요한 가정에 의존합니다. 감마 - 감마 모델은 실제로 금전적 가치와 구매 빈도간에 아무런 관계가 없다고 가정합니다. 실제로 우리는이 모델을 사용하기 위해 두 벡터 간의 피어슨 상관 관계가 0에 가까운 지 여부를 확인해야합니다.

    returning_customers_summary[['monetary_value', 'frequency']].corr()
    """
                    monetary_value  frequency
    monetary_value        1.000000   0.113884
    frequency             0.113884   1.000000
    """
이 시점에서 우리는 감마 - 감마 모델을 훈련 할 수 있고 고객의 조건부 평균 예상 평생 가치를 예측할 수 있습니다.

    from lifetimes import GammaGammaFitter

    ggf = GammaGammaFitter(penalizer_coef = 0)
    ggf.fit(returning_customers_summary['frequency'],
            returning_customers_summary['monetary_value'])
    print(ggf)
    """
    <lifetimes.GammaGammaFitter: fitted with 946 subjects, p: 6.25, q: 3.74, v: 15.45>
    """
이제 평균 거래 금액을 계산할 수 있습니다.

    print(ggf.conditional_expected_average_profit(
            summary_with_money_value['frequency'],
            summary_with_money_value['monetary_value']
        ).head(10))
    """
    customer_id
    1     24.658619
    2     18.911489
    3     35.170981
    4     35.170981
    5     35.170981
    6     71.462843
    7     18.911489
    8     35.170981
    9     27.282408
    10    35.170981
    dtype: float64
    """

    print("Expected conditional average profit: %s, Average profit: %s" % (
        ggf.conditional_expected_average_profit(
            summary_with_money_value['frequency'],
            summary_with_money_value['monetary_value']
        ).mean(),
        summary_with_money_value[summary_with_money_value['frequency']>0]['monetary_value'].mean()
    ))
    """
    Expected conditional average profit: 35.2529588256, Average profit: 35.078551797
    """
자본 비용을 조정하는 DCF 방법 (https://en.wikipedia.org/wiki/Discounted_cash_flow)을 사용하여 총 CLV를 계산하는 동안 :

    # refit the BG model to the summary_with_money_value dataset
    bgf.fit(summary_with_money_value['frequency'], summary_with_money_value['recency'], summary_with_money_value['T'])

    print(ggf.customer_lifetime_value(
        bgf, #the model to use to predict the number of future transactions
        summary_with_money_value['frequency'],
        summary_with_money_value['recency'],
        summary_with_money_value['T'],
        summary_with_money_value['monetary_value'],
        time=12, # months
        discount_rate=0.01 # monthly discount rate ~ 12.7% annually
    ).head(10))
    """
    customer_id
    1      140.096211
    2       18.943467
    3       38.180574
    4       38.180574
    5       38.180574
    6     1003.868107
    7       28.109683
    8       38.180574
    9      167.418216
    10      38.180574
    Name: clv, dtype: float64
    """

## 마무리
정리하면 크게 2가지를 알아야 본 패키지를 제대로 사용할 수 있습니다.
1. BG/NBD 개념도
2. frequency & recency & T 개념

해당 글에서 소개하지 못한 모델들도 확인해보면 좋을 것 같습니다.
- **'BetaGeoFitter'**
- 'ParetoNBDFitter'
- **'GammaGammaFitter'**
- 'ModifiedBetaGeoFitter'
- 'BetaGeoBetaBinomFitter'

쉽지 않을 것 같지만 다음엔 실서비스에도 적용한 결과를 보려합니다.


참고 : [Lifetime:Python에서 고객 평생 가치 측정](https://dataorigami.net/blogs/napkin-folding/18868411-lifetimes-measuring-customer-lifetime-value-in-python)
