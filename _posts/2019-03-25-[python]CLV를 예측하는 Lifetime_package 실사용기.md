

비식별 예약데이터를 활용하여서, 고객 일부의 데이터를 분석해보았습니다.
데이터는 미리 준비된 자료를 가져옵니다.

### CLV모델 정의
-frequency(빈도)는 고객이 반복 구매 한 횟수를 나타냅니다. 즉, 총 구매 횟수보다 1 적은 값 입니다.
- T(생존나이)는 선택된 시간 단위의 고객의 생존일수를 나타냅니다. 이것은 고객의 첫 구매와 연구 기간의 끝 사이의 기간과 동일합니다.
- recency(최근 구매 일)은 가장 최근에 구매 한 고객의 T를 나타냅니다. 이는 고객의 첫 구매와 최신 구매 간의 기간과 동일합니다. (따라서 1회만 구매 한 경우 recency 값이 0 입니다)


### EDA(데이터 탐색)
```
import pandas as pd

col_names = ['CustomerID', 'id', 'InvoiceDate', 'Sales']
df=pd.read_csv('../../../Downloads/sample_.csv', header=-1)
df.columns = col_names
df.head()
```
![cap1](https://www.dropbox.com/s/se334saq0xy65rq/Screenshot%20at%20Mar%2025%2000-44-59.png?dl=1)

총 3539개의 예약데이터들로 이루어져 있고, 우리가 관심 있는 데이터는 [고객ID,구매일,판매액] 입니다.
lifetimes 패키지에 맞게 데이터를 다음과 같이 변환을 해줍니다.

```
import datetime as dt
df['InvoiceDate'] = pd.to_datetime(df['InvoiceDate']).dt.date
df = df[pd.notnull(df['CustomerID'])]

cols_of_interest = ['CustomerID', 'InvoiceDate', 'Sales']
df = df[cols_of_interest]
```

```
from lifetimes.plotting import *
from lifetimes.utils import *
data = summary_data_from_transaction_data(df, 'CustomerID'
                                            , 'InvoiceDate'
                                            , monetary_value_col='Sales')
data.head()
```

![cap2](https://www.dropbox.com/s/x2o7nzzu7t6you9/Screenshot%20at%20Mar%2025%2000-45-38.png?dl=1)

고객당 frequency, recency, T, monetary_value(고객가치)를 확인 할 수 있습니다.
1350667고객은 1번만 구매 했으므로(반복 없음), frequency(빈도) 및 recency(최신성)은 0 이며 그의 총 생존나이는 731일 입니다. (첫 구매와 분석 기간의 종료 사이의 기간)


```
data['frequency'].plot(kind='hist', bins=50)
print(data['frequency'].describe())
print(sum(data['frequency'] == 0)/float(len(data)))
```

![cap3](https://www.dropbox.com/s/sh124zj3rgxxvsf/Screenshot%20at%20Mar%2025%2000-45-50.png?dl=1)

419명의 평균적인 구매 분포는 다음과 같습니다.
계산에 의하면 전체 34.3%가 단 한 번만 구매를 했습니다.

### BG/NBD 모델을 사용한 빈도/최신성 분석
```
from lifetimes import BetaGeoFitter
bgf = BetaGeoFitter(penalizer_coef=0.0)
bgf.fit(data['frequency'], data['recency'], data['T'])
print(bgf)

<lifetimes.BetaGeoFitter: fitted with 419 subjects, a: 1.10, alpha: 15.23, b: 10.14, r: 0.29>
```

### 빈도 / 날짜 행렬 시각화
- 우하향 : 가장 최고의 고객(고매출)이 밝은 색.
- 우상향 : 고매출이나, 자주 오지 않는 사람.

(우하향)지금까지 140번 구매를 한 고객은 첫구매와 최근구매가 약 1200일의 간격 동안 이루어졌습니다.(꾸준히 매출을 올려주는 고객!)

그러나, 표의 중간지점 즉, 드물게 구매하는 고객(40,1000)은 다시 구매할지 확실하진 않습니다.

```
from lifetimes.plotting import plot_frequency_recency_matrix
import matplotlib.pyplot as plt
fig = plt.figure(figsize=(12,8))
plot_frequency_recency_matrix(bgf)
```
![cap4](https://www.dropbox.com/s/c8oqt7rrju00ybj/Screenshot%20at%20Mar%2025%2000-46-00.png?dl=1)

그래서 `plot_probability_alive_matrix`를 통해 좀 더 명확하게 생존여부를 확률로 표현이 가능합니다.

```
from lifetimes.plotting import plot_probability_alive_matrix
fig = plt.figure(figsize=(12,8))
plot_probability_alive_matrix(bgf)
```
![cap5](https://www.dropbox.com/s/fjosjaijctiht8k/Screenshot%20at%20Mar%2025%2000-46-08.png?dl=1)

최근에 구입한 고객은 거의 확실하게 알 수 있습니다. 최근에 구입하지는 않았지만 많이 구매 한 고객은 이탈했을 가능성이 큽니다.


모델의 `conditional_expected_number_of_purchases_up_to_time` 함수를 사용하면, 모델에서 "기간동안 가장 높은 예상 구매액"을 예측 할 수 있습니다.
```
t = 1
data['predicted_purchases'] = bgf.conditional_expected_number_of_purchases_up_to_time(t, data['frequency'], data['recency'], data['T'])
data.sort_values(by='predicted_purchases').tail(5)
```

![cap6](https://www.dropbox.com/s/taqiny8l5jjkia1/Screenshot%20at%20Mar%2025%2000-46-30.png?dl=1)

위 고객들은 다음날 (t=1)구매를 기대하는 상위 5명을 보여줍니다. predicted_purchases 열은 에상 구매 확률을 보여줍니다.


### 모델 적합성 평가

```
from lifetimes.plotting import plot_period_transactions
plot_period_transactions(bgf)
```

![cap7](https://www.dropbox.com/s/86xh0fx4bcjvq4f/Screenshot%20at%20Mar%2025%2000-46-39.png?dl=1)

모델이 나쁘지 않지만 검증할 필요는 있습니다.

데이터셋을 calibration(교정)데이터셋 / holdout(홀드아웃)데이터셋으로 분할합니다.
모델이 아직 보지 못한 데이터에 대해 어떻게 수행하는지 테스트하고자 할 때 중요합니다.(기계 학습에서 교차 검증과 동일)


```
from lifetimes.utils import calibration_and_holdout_data
summary_cal_holdout = calibration_and_holdout_data(df, 'CustomerID', 'InvoiceDate'
                                                   ,
                                        calibration_period_end='2019-02-01',
                                        observation_period_end='2019-03-25'
                                                  )   
print(summary_cal_holdout.head())
```


![cap8](https://www.dropbox.com/s/09b9mcwk35b31lw/Screenshot%20at%20Mar%2025%2000-47-17.png?dl=1)

```
from lifetimes.plotting import plot_calibration_purchases_vs_holdout_purchases
bgf.fit(summary_cal_holdout['frequency_cal'], summary_cal_holdout['recency_cal'], summary_cal_holdout['T_cal'])
plot_calibration_purchases_vs_holdout_purchases(bgf, summary_cal_holdout)
```

![cap](https://www.dropbox.com/s/qjc0g2ykqrt6cgi/Screenshot%20at%20Mar%2025%2000-47-27.png?dl=1)


이 플롯에서 데이터를 샘플 내(교정) 및 유효성 검사(홀드아웃)기간으로 분리합니다. 샘플 기간은 2019-02-01부터 시작됩니다. 유효 기간은 2019-02-02에서 2019-03-25까지입니다. 플롯은 교정주기의 모든 고객을 반복 구매 횟수(x축)로 분류한 다음 보유 기간(y축)에서 반복 구매한 평균을 그룹화합니다. 주황색과 파란색선은 모델 예측과 y축의 실제 결과를 각각 나타냅니다. 만든 모델은 견본에서 고객 기반의 행동을 비교적 정확하게 예측할 수 있습니다. 모델은 3회 구매시 다소 정확도가 떨어집니다. 5회 구매시에는 과소 평가됩니다.

### 고객 거래 예측
고객 기록을 기반으로 향후 개인의 향후 구매 내역을 예측할 수 있습니다.

```
t = 10
individual = data.loc[1350666]
bgf.predict(t, individual['frequency'], individual['recency'], individual['T'])
```

![cap](https://www.dropbox.com/s/axnur39j6n060fl/Screenshot%20at%20Mar%2025%2000-47-38.png?dl=1)

모델은 1350666고객이 향후 거래가 10일 안에 약 94% 재구매가 이루어 질 것이라고 예측합니다.


### 고객 확률 기록

고객 거래 내역을 감안할 때, 우리는 훈련 된 모델에 따라 생존 가능성에 대한 역사적 확률을 계산할 수 있습니다. 예를 들어, 우리는 최고의 고객의 거래 내역을보고 살아있을 확률을보고 싶습니다.

```
from lifetimes.plotting import plot_history_alive
import matplotlib.pyplot as plt
fig = plt.figure(figsize=(12,8))
id = 1350666
days_since_birth = 365
sp_trans = df.loc[df['CustomerID'] == id]
plot_history_alive(bgf, days_since_birth, sp_trans, 'InvoiceDate')
```

![cap](https://www.dropbox.com/s/ma37ivtiraldpff/Screenshot%20at%20Mar%2025%2000-47-48.png?dl=1)

135066고객은 확실히 생존해 있습니다.

```
fig = plt.figure(figsize=(12,8))
id = 1350947
days_since_birth = 365
sp_trans = df.loc[df['CustomerID'] == id]
plot_history_alive(bgf, days_since_birth, sp_trans, 'InvoiceDate')
```

1350947고객은 2016-08 기준으로 더이상 재구매가 일어나지 않았습니다. 즉, 더이상 생존하지 않습니다.

![cap](https://www.dropbox.com/s/48z358id4ube0xw/Screenshot%20at%20Mar%2025%2000-48-10.png?dl=1)


1350932 고객은 2016-09에서 생존하지 않았으나, 2017-01 쯤에 다시 생존해서, 중간중간 생존과 죽음을 반복합니다.

```
fig = plt.figure(figsize=(12,8))
id = 1350932
days_since_birth = 365
sp_trans = df.loc[df['CustomerID'] == id]
plot_history_alive(bgf, days_since_birth, sp_trans, 'InvoiceDate')
```
![caps](https://www.dropbox.com/s/ak81fm4gz80hoea/Screenshot%20at%20Mar%2025%2000-48-18.png?dl=1)


### 금전적 가치의 감마 - 감마 모델을 사용하여 고객의 평생 가치 산정

이제 각 거래의 경제적 가치를 고려할 수 있습니다. 예측하기 위해 감마-감마 모델을 사용 하여 향후 고객 수준에서 지출 가능성을 예측합니다.

적어도 우리와 함께 반복 구입 한 고객만을 추정하고 있습니다. 따라서 275명의 ​​고객을 예상하고 있습니다.

```
returning_customers_summary = data[data['frequency']>0]

print(len(returning_customers_summary))
returning_customers_summary.head()
```

![cap](https://www.dropbox.com/s/29qpvwiaudijhoj/Screenshot%20at%20Mar%2025%2000-48-33.png?dl=1)


감마-감마 모델을 적용한 후에는 각 고객의 평균 거래 가격을 산정 할 수 있습니다.
```
from lifetimes import GammaGammaFitter
ggf = GammaGammaFitter(penalizer_coef = 0)
ggf.fit(returning_customers_summary['frequency'],
        returning_customers_summary['monetary_value'])
print(ggf)


print(ggf.conditional_expected_average_profit(
        data['frequency'],
        data['monetary_value']
    ).head(10))
```

![cap](https://www.dropbox.com/s/7xwwqwkxssdcumf/Screenshot%20at%20Mar%2025%2000-48-44.png?dl=1)
