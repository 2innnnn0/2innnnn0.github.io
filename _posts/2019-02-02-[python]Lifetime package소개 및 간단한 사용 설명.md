
![tyle](https://www.dropbox.com/s/5hsvwm67n5wppck/tyle-SLO-3-1549722095.png?dl=1)

- Lifetime package 소개 및 간단한 사용 설명.

![아티클 Lifetime:Python에서 고객 평생 가치 측정](https://dataorigami.net/blogs/napkin-folding/18868411-lifetimes-measuring-customer-lifetime-value-in-python)



## Lifetime package 소개
Lifetimes 는 CLV를 계산할 수있는 Python 라이브러리입니다.

좀 더 일반적으로 Lifetime은 몇 가지 가정을 토대로 미래의 사용법을 이해하고 예측하는 데 사용할 수 있습니다.

연구중인 개체는 임의의 시간이 지나면 죽을 수 있습니다.
엔티티는 살아있을 때 상호 작용합니다.
Lifetimes는이 개체가 아직 살아 있는지 추정 하고 기존 기록을 기반으로 상호 작용할 것인지를 예측하는 데 사용할 수 있습니다.
너무 추상적 인 경우 다음 상황을 고려하십시오.

방문자가 귀하의 웹 사이트로 얼마나 자주 돌아갈 지 예측합니다.
환자가 병원으로 얼마나 자주 돌아갈지를 이해합니다.
사용 기록 만 사용하여 사망 한 사람 예측.
정말로, "고객"은 생존 분석에서 "출생"과 "사망"과 비슷한 매우 일반적인 용어입니다.
반복되는 개인이있을 때마다 Lifetimes를 사용하여 행동을 이해할 수 있습니다.


## 설치

> pip install lifetimes

## 빠른 시작
아래의 예 cdnow_customers.csv는 datasets/디렉토리 에있는 것을 사용하고 있습니다 .

> from lifetimes.datasets import load_cdnow_summary
data = load_cdnow_summary(index_col=[0])
print(data.head())


x고객이 반복 구매 한 횟수 (또는`frequency`)를 나타냅니다. T고객의 연령을 나타냅니다. t_x고객이 가장 최근에 구매 한 항목 (또는 `recency`)을 표시합니다.

>        frequency  recency      T
    ID                           
    1           2    30.43  38.86
    2           1     1.71  38.86
    3           0     0.00  38.86
    4           0     0.00  38.86
    5           0     0.00  38.86



### 모델을 아웃 데이터에 맞추기
먼저 BG / NBD 모델을 사용하겠습니다.
![BG / NBD 모델](http://mktg.uni-svishtov.bg/ivm/resources/Counting_Your_Customers.pdf)


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

피팅 후, 우리는 많은 좋은 메소드와 속성을 피터 객체에 첨부합니다.

## 빈도 / 날짜순 그래프 시각화
고려해보십시오 : 고객이 3 주 동안 매일 매일 당신에게서 샀고, 우리는 몇 달 동안 그들로부터 소식을받지 못했습니다. 그들이 아직도 "살아있다"는 기회는 무엇입니까? 꽤 작습니다.

 반면에 역사적으로 한 분기에 한 번 구매하고 지난 분기에 구매 한 고객은 아직 살아있을 가능성이 큽니다. 이 관계를 시각적으로 표시 할 수 있습니다. `빈도 / 최근 값 행렬` 은 다음 기간에 인위적인 고객이 기대하는 거래 수를 계산합니다. 최근 거래가 성사 된 기간과 빈도(반복 거래 횟수)를 보여줍니다.)

    from lifetimes.plotting import plot_frequency_recency_matrix

    plot_frequency_recency_matrix(bgf)

고객이 귀하에게서 25 번을 구매하고 가장 최근 구매가 35 주 (개인이 35 주인 경우) 였을 때 가장 적합한 고객 (오른쪽 하단)임을 알 수 있습니다. 가장 차가운 고객은 오른쪽 상단에있는 고객입니다. 그들은 많이 빨리 구입했으며, 몇 주 만에 제품을 보지 못했습니다.

주위에는 아름다운 "꼬리"가 있습니다 (5,25). 즉 자주 구입하는 고객을 대표하는, 그러나 우리는 최근에 그 사람이나 그 여자를 본 적이, 그래서 그들은 수도 를 다시 구입 - 우리는 그들이 죽은하거나 구매 사이에 있는지 확실하지 않다.

살펴볼 또 다른 흥미로운 매트릭스는 여전히 살아있을 확률입니다 .

    from lifetimes.plotting import plot_probability_alive_matrix

    plot_probability_alive_matrix(bgf)



## 고객을 최우선 순위에서 최악으로 평가
고객에게 돌아가서 "다음 기간에 가장 높은 예상 구매액"에서 가장 낮은 것으로 순위를 매겨 봅시다. 모델은 내역을 사용하여 다음 기간에 고객의 예상 구매를 예측하는 메소드를 노출합니다.

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

26 건의 구매를 한 고객이 매우 최근에 우리에게서 구매 한 것이 다음 기간에 다시 구매할 것임을 알 수 있습니다.

## 모델 적합성 평가
우리는 예측할 수 있으며 우리는 고객의 행동을 시각화 할 수 있지만 모델이 맞습니까? 모델의 정확성을 평가하는 몇 가지 방법이 있습니다. 첫 번째는 적합 모델의 매개 변수로 시뮬레이션 된 데이터와 인공 데이터를 비교하는 것입니다.

    from lifetimes.plotting import plot_period_transactions
    plot_period_transactions(bgf)

실제 데이터와 시뮬레이션 데이터가 잘 일치하는 것을 볼 수 있습니다. 이것은 우리 모델이 끔찍하지 않다는 것을 증명합니다.

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

이 데이터 세트를 사용하여 `_cal`열을 맞추고 열을 테스트 할 수 `_holdout`있습니다.


    from lifetimes.plotting import plot_calibration_purchases_vs_holdout_purchases

    bgf.fit(summary_cal_holdout['frequency_cal'], summary_cal_holdout['recency_cal'], summary_cal_holdout['T_cal'])
    plot_calibration_purchases_vs_holdout_purchases(bgf, summary_cal_holdout)


## 고객 예측
고객 기록에 대한 기본 사항으로, 우리는 향후 개인 구매가 어떻게 될지 예측할 수 있습니다.

    t = 10 #predict purchases in 10 periods
    individual = summary.iloc[20]
    # The below function is an alias to `bfg.conditional_expected_number_of_purchases_up_to_time`
    bgf.predict(t, individual['frequency'], individual['recency'], individual['T'])
    # 0.0576511



## 고객의 가능성 역사
고객 거래 내역을 감안할 때, 우리는 훈련 된 모델에 따라 생존 가능성에 대한 역사적 확률을 계산할 수 있습니다. 예 :

    from lifetimes.plotting import plot_history_alive

    id = 35
    days_since_birth = 200
    sp_trans = transaction_data.loc[transaction_data['id'] == id]
    plot_history_alive(bgf, days_since_birth, sp_trans, 'date')


## 감마 - 감마 모델을 사용하여 고객의 평생 가치를 추정
이 기간 동안 우리는 각 거래의 경제적 가치를 고려하지 않았으며 주로 거래의 발생에 초점을 맞추 었습니다. 이것을 추정하기 위해 감마 - 감마 서브 모델을 사용할 수 있습니다. 하지만 먼저 거래 별 경제적 가치 (즉, 수익 또는 수익)가 포함 된 거래 데이터로 요약 데이터를 작성해야합니다.

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
