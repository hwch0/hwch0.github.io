# 📈 소비자 물가지수 시각화

<br><br>
파이썬 시각화 라이브러리 <code>plotly</code>를 사용하여 소비자 물가지수를 시각화 해보도록 하겠습니다.

## 필요한 라이브러리 설치


```python
pip install plotly
```

    Collecting plotly
      Using cached plotly-5.10.0-py2.py3-none-any.whl (15.2 MB)
    Requirement already satisfied: tenacity>=6.2.0 in c:\users\user\anaconda3\lib\site-packages (from plotly) (8.1.0)
    Installing collected packages: plotly
    Note: you may need to restart the kernel to use updated packages.
    

    ERROR: Could not install packages due to an OSError: [WinError 32] 다른 프로세스가 파일을 사용 중이기 때문에 프로세스가 액세스 할 수 없습니다: 'C:\\Users\\USER\\anaconda3\\Lib\\site-packages\\plotly\\graph_objs\\layout\\_hoverlabel.py'
    Consider using the `--user` option or check the permissions.
    
    


```python
import plotly.io as pio # Plotly input output
import plotly.express as px # 빠르게 그리는 방법
import plotly.graph_objects as go # 디테일한 설정
import plotly.figure_factory as ff # 템플릿 불러오기
from plotly.subplots import make_subplots # subplot 만들기
from plotly.validators.scatter.marker import SymbolValidator # Symbol 꾸미기에 사용됨
import numpy as np
import pandas as pd
```

    Collecting plotly
      Downloading plotly-5.10.0-py2.py3-none-any.whl (15.2 MB)
    Collecting tenacity>=6.2.0
      Downloading tenacity-8.1.0-py3-none-any.whl (23 kB)
    Installing collected packages: tenacity, plotly
    Successfully installed plotly-5.10.0 tenacity-8.1.0
    

## 1. 물가지수(품목성질별_소비자물가지수) DataFrame 변환

통계청에서 다운받은 소비자물가지수 파일을 Dataframe으로 불러옵니다.


```python
df = pd.read_excel('./품목성질별_소비자물가지수_2020100__20220727184250.xlsx')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>시도별</th>
      <th>시점</th>
      <th>총지수</th>
      <th>상품</th>
      <th>농축수산물</th>
      <th>공업제품</th>
      <th>전기 · 가스 · 수도</th>
      <th>서비스</th>
      <th>집세</th>
      <th>공공서비스</th>
      <th>개인서비스</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>전국</td>
      <td>2017.11</td>
      <td>97.347</td>
      <td>97.387</td>
      <td>87.968</td>
      <td>99.399</td>
      <td>100.415</td>
      <td>97.341</td>
      <td>99.639</td>
      <td>101.958</td>
      <td>94.753</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>2017.12</td>
      <td>97.698</td>
      <td>97.939</td>
      <td>90.058</td>
      <td>99.635</td>
      <td>100.438</td>
      <td>97.490</td>
      <td>99.648</td>
      <td>102.038</td>
      <td>94.987</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>2018.01</td>
      <td>98.106</td>
      <td>98.307</td>
      <td>92.106</td>
      <td>99.615</td>
      <td>100.472</td>
      <td>97.938</td>
      <td>99.725</td>
      <td>102.757</td>
      <td>95.418</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>2018.02</td>
      <td>98.855</td>
      <td>99.431</td>
      <td>96.765</td>
      <td>99.969</td>
      <td>100.541</td>
      <td>98.395</td>
      <td>99.792</td>
      <td>102.707</td>
      <td>96.200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>2018.03</td>
      <td>98.751</td>
      <td>98.947</td>
      <td>93.794</td>
      <td>100.047</td>
      <td>100.576</td>
      <td>98.600</td>
      <td>99.821</td>
      <td>102.587</td>
      <td>96.586</td>
    </tr>
  </tbody>
</table>
</div>



## 2. 데이터 전처리


```python
# 1. 원본은 남겨두고 사본에서 진행합니다. 
df1 = df.copy()

# 2. 컬렴명 '시도별' 데이터는 제외하고 가져옵니다. 
df1 = df1.loc[:, ~df1.columns.isin(['시도별'])]

# 3. 컬럼명 '시점'을 '연월'로 변경합니다.
df1.rename(columns={'시점':'연월'}, inplace=True) # inplace = True는 원본에 바로 반영

df1.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>연월</th>
      <th>총지수</th>
      <th>상품</th>
      <th>농축수산물</th>
      <th>공업제품</th>
      <th>전기 · 가스 · 수도</th>
      <th>서비스</th>
      <th>집세</th>
      <th>공공서비스</th>
      <th>개인서비스</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017.11</td>
      <td>97.347</td>
      <td>97.387</td>
      <td>87.968</td>
      <td>99.399</td>
      <td>100.415</td>
      <td>97.341</td>
      <td>99.639</td>
      <td>101.958</td>
      <td>94.753</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017.12</td>
      <td>97.698</td>
      <td>97.939</td>
      <td>90.058</td>
      <td>99.635</td>
      <td>100.438</td>
      <td>97.490</td>
      <td>99.648</td>
      <td>102.038</td>
      <td>94.987</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018.01</td>
      <td>98.106</td>
      <td>98.307</td>
      <td>92.106</td>
      <td>99.615</td>
      <td>100.472</td>
      <td>97.938</td>
      <td>99.725</td>
      <td>102.757</td>
      <td>95.418</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018.02</td>
      <td>98.855</td>
      <td>99.431</td>
      <td>96.765</td>
      <td>99.969</td>
      <td>100.541</td>
      <td>98.395</td>
      <td>99.792</td>
      <td>102.707</td>
      <td>96.200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018.03</td>
      <td>98.751</td>
      <td>98.947</td>
      <td>93.794</td>
      <td>100.047</td>
      <td>100.576</td>
      <td>98.600</td>
      <td>99.821</td>
      <td>102.587</td>
      <td>96.586</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 컬럼별 null의 개수와 컬럼별 데이터타입을 확인합니다.

df1.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 56 entries, 0 to 55
    Data columns (total 10 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   연월            56 non-null     float64
     1   총지수           56 non-null     float64
     2   상품            56 non-null     float64
     3   농축수산물         56 non-null     float64
     4   공업제품          56 non-null     float64
     5   전기 · 가스 · 수도  56 non-null     float64
     6   서비스           56 non-null     float64
     7   집세            56 non-null     float64
     8   공공서비스         56 non-null     float64
     9   개인서비스         56 non-null     float64
    dtypes: float64(10)
    memory usage: 4.5 KB
    


```python
# '연월' 컬럼의 데이터 타입 변환 (float -> string)

df1['연월'] = df1['연월'].astype(str)
df1.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 56 entries, 0 to 55
    Data columns (total 10 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   연월            56 non-null     object 
     1   총지수           56 non-null     float64
     2   상품            56 non-null     float64
     3   농축수산물         56 non-null     float64
     4   공업제품          56 non-null     float64
     5   전기 · 가스 · 수도  56 non-null     float64
     6   서비스           56 non-null     float64
     7   집세            56 non-null     float64
     8   공공서비스         56 non-null     float64
     9   개인서비스         56 non-null     float64
    dtypes: float64(9), object(1)
    memory usage: 4.5+ KB
    


```python
# '연월' 기준으로 2018년 데이터만 출력해서 데이터 확인

df1.loc[df1['연월'].str.contains('2018')]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>연월</th>
      <th>총지수</th>
      <th>상품</th>
      <th>농축수산물</th>
      <th>공업제품</th>
      <th>전기 · 가스 · 수도</th>
      <th>서비스</th>
      <th>집세</th>
      <th>공공서비스</th>
      <th>개인서비스</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>2018.01</td>
      <td>98.106</td>
      <td>98.307</td>
      <td>92.106</td>
      <td>99.615</td>
      <td>100.472</td>
      <td>97.938</td>
      <td>99.725</td>
      <td>102.757</td>
      <td>95.418</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018.02</td>
      <td>98.855</td>
      <td>99.431</td>
      <td>96.765</td>
      <td>99.969</td>
      <td>100.541</td>
      <td>98.395</td>
      <td>99.792</td>
      <td>102.707</td>
      <td>96.200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018.03</td>
      <td>98.751</td>
      <td>98.947</td>
      <td>93.794</td>
      <td>100.047</td>
      <td>100.576</td>
      <td>98.600</td>
      <td>99.821</td>
      <td>102.587</td>
      <td>96.586</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2018.04</td>
      <td>98.931</td>
      <td>99.063</td>
      <td>94.180</td>
      <td>100.097</td>
      <td>100.576</td>
      <td>98.843</td>
      <td>99.859</td>
      <td>102.517</td>
      <td>97.027</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2018.05</td>
      <td>98.979</td>
      <td>99.015</td>
      <td>92.509</td>
      <td>100.460</td>
      <td>100.576</td>
      <td>98.974</td>
      <td>99.898</td>
      <td>102.557</td>
      <td>97.215</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2018.06</td>
      <td>98.779</td>
      <td>98.782</td>
      <td>90.217</td>
      <td>100.735</td>
      <td>100.576</td>
      <td>98.768</td>
      <td>99.898</td>
      <td>102.497</td>
      <td>96.883</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2018.07</td>
      <td>98.590</td>
      <td>98.133</td>
      <td>90.771</td>
      <td>100.430</td>
      <td>93.921</td>
      <td>98.983</td>
      <td>99.917</td>
      <td>102.447</td>
      <td>97.269</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2018.08</td>
      <td>99.462</td>
      <td>99.799</td>
      <td>99.166</td>
      <td>100.607</td>
      <td>93.898</td>
      <td>99.198</td>
      <td>99.936</td>
      <td>102.417</td>
      <td>97.638</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2018.09</td>
      <td>100.221</td>
      <td>101.639</td>
      <td>104.874</td>
      <td>100.804</td>
      <td>101.897</td>
      <td>99.086</td>
      <td>99.936</td>
      <td>102.238</td>
      <td>97.530</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2018.1</td>
      <td>100.041</td>
      <td>101.116</td>
      <td>100.207</td>
      <td>101.265</td>
      <td>101.886</td>
      <td>99.160</td>
      <td>99.946</td>
      <td>102.228</td>
      <td>97.656</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2018.11</td>
      <td>99.330</td>
      <td>99.770</td>
      <td>94.641</td>
      <td>100.813</td>
      <td>101.886</td>
      <td>98.992</td>
      <td>99.955</td>
      <td>102.138</td>
      <td>97.404</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2018.12</td>
      <td>98.988</td>
      <td>99.024</td>
      <td>94.700</td>
      <td>99.773</td>
      <td>101.886</td>
      <td>98.983</td>
      <td>99.936</td>
      <td>102.108</td>
      <td>97.404</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 10월 자료가 '연월'을 string으로 형변환 하면서 끝에 0이 날라가버림 
# '2018.1' -> '2018.10' 으로 변경해줘야 함 (2018 ~ 2020년 모두)

df1 = df1.replace('2018.1', '2018.10').replace('2019.1', '2019.10').replace('2020.1', '2020.10').replace('2021.1', '2021.10')
df1.loc[df1['연월'].str.contains('2018')]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>연월</th>
      <th>총지수</th>
      <th>상품</th>
      <th>농축수산물</th>
      <th>공업제품</th>
      <th>전기 · 가스 · 수도</th>
      <th>서비스</th>
      <th>집세</th>
      <th>공공서비스</th>
      <th>개인서비스</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>2018.01</td>
      <td>98.106</td>
      <td>98.307</td>
      <td>92.106</td>
      <td>99.615</td>
      <td>100.472</td>
      <td>97.938</td>
      <td>99.725</td>
      <td>102.757</td>
      <td>95.418</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018.02</td>
      <td>98.855</td>
      <td>99.431</td>
      <td>96.765</td>
      <td>99.969</td>
      <td>100.541</td>
      <td>98.395</td>
      <td>99.792</td>
      <td>102.707</td>
      <td>96.200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018.03</td>
      <td>98.751</td>
      <td>98.947</td>
      <td>93.794</td>
      <td>100.047</td>
      <td>100.576</td>
      <td>98.600</td>
      <td>99.821</td>
      <td>102.587</td>
      <td>96.586</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2018.04</td>
      <td>98.931</td>
      <td>99.063</td>
      <td>94.180</td>
      <td>100.097</td>
      <td>100.576</td>
      <td>98.843</td>
      <td>99.859</td>
      <td>102.517</td>
      <td>97.027</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2018.05</td>
      <td>98.979</td>
      <td>99.015</td>
      <td>92.509</td>
      <td>100.460</td>
      <td>100.576</td>
      <td>98.974</td>
      <td>99.898</td>
      <td>102.557</td>
      <td>97.215</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2018.06</td>
      <td>98.779</td>
      <td>98.782</td>
      <td>90.217</td>
      <td>100.735</td>
      <td>100.576</td>
      <td>98.768</td>
      <td>99.898</td>
      <td>102.497</td>
      <td>96.883</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2018.07</td>
      <td>98.590</td>
      <td>98.133</td>
      <td>90.771</td>
      <td>100.430</td>
      <td>93.921</td>
      <td>98.983</td>
      <td>99.917</td>
      <td>102.447</td>
      <td>97.269</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2018.08</td>
      <td>99.462</td>
      <td>99.799</td>
      <td>99.166</td>
      <td>100.607</td>
      <td>93.898</td>
      <td>99.198</td>
      <td>99.936</td>
      <td>102.417</td>
      <td>97.638</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2018.09</td>
      <td>100.221</td>
      <td>101.639</td>
      <td>104.874</td>
      <td>100.804</td>
      <td>101.897</td>
      <td>99.086</td>
      <td>99.936</td>
      <td>102.238</td>
      <td>97.530</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2018.10</td>
      <td>100.041</td>
      <td>101.116</td>
      <td>100.207</td>
      <td>101.265</td>
      <td>101.886</td>
      <td>99.160</td>
      <td>99.946</td>
      <td>102.228</td>
      <td>97.656</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2018.11</td>
      <td>99.330</td>
      <td>99.770</td>
      <td>94.641</td>
      <td>100.813</td>
      <td>101.886</td>
      <td>98.992</td>
      <td>99.955</td>
      <td>102.138</td>
      <td>97.404</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2018.12</td>
      <td>98.988</td>
      <td>99.024</td>
      <td>94.700</td>
      <td>99.773</td>
      <td>101.886</td>
      <td>98.983</td>
      <td>99.936</td>
      <td>102.108</td>
      <td>97.404</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 시각화를 하려면 물가지수와 품목성질을 열로 재구조화 필요 
# columns: '연월' , '품목성질', '물가지수'

df2 = pd.melt(df1, id_vars=['연월'], var_name='품목성질', value_name='물가지수')
df2.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>연월</th>
      <th>품목성질</th>
      <th>물가지수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017.11</td>
      <td>총지수</td>
      <td>97.347</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017.12</td>
      <td>총지수</td>
      <td>97.698</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018.01</td>
      <td>총지수</td>
      <td>98.106</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018.02</td>
      <td>총지수</td>
      <td>98.855</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018.03</td>
      <td>총지수</td>
      <td>98.751</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2018.04</td>
      <td>총지수</td>
      <td>98.931</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2018.05</td>
      <td>총지수</td>
      <td>98.979</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2018.06</td>
      <td>총지수</td>
      <td>98.779</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2018.07</td>
      <td>총지수</td>
      <td>98.590</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2018.08</td>
      <td>총지수</td>
      <td>99.462</td>
    </tr>
  </tbody>
</table>
</div>



## 3. 시각화

plotly를 사용하여 아래의 4가지 정보에 대해 라인 그래프로 시각화 해보겠습니다.

- graph1. Price index Summary (2020=100)
- graph2. 총지수와 물가지수(상품지수, 서비스지수) 시각화
- graph3. 총지수와 서비스지수(집세, 공공서비스, 개인서비스) 시각화
- graph4. 총지수와 상품지수(농축수산물, 공업제품, 전기가스수도) 시각화


```python
# summary

graph1 = px.line(df2, x=df2['연월'], y=df2['물가지수'], color=df2['품목성질'],
              title = '<b>Price index Summary (2020=100)</b>',
              hover_name = df2['품목성질'])

graph1.update_layout(xaxis_title = 'Month', yaxis_title = 'Price index (2020=100)')

graph1.show()
```


<div>                            <div id="9f36ad20-65f6-4cb7-8869-eeab10c79beb" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("9f36ad20-65f6-4cb7-8869-eeab10c79beb")) {                    Plotly.newPlot(                        "9f36ad20-65f6-4cb7-8869-eeab10c79beb",                        [{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\ucd1d\uc9c0\uc218<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218","\ucd1d\uc9c0\uc218"],"legendgroup":"\ucd1d\uc9c0\uc218","line":{"color":"#636efa","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\ucd1d\uc9c0\uc218","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[97.347,97.698,98.106,98.855,98.751,98.931,98.979,98.779,98.59,99.462,100.221,100.041,99.33,98.988,98.884,99.311,99.121,99.481,99.652,99.491,99.187,99.425,99.794,100.041,99.481,99.719,100.09,100.16,99.94,99.5,99.44,99.71,99.63,100.19,100.74,100.18,100.09,100.33,101.04,101.58,101.84,101.98,102.05,102.05,102.26,102.75,103.17,103.35,103.87,104.04,104.69,105.3,106.06,106.85,107.56,108.22],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uc0c1\ud488<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488","\uc0c1\ud488"],"legendgroup":"\uc0c1\ud488","line":{"color":"#EF553B","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uc0c1\ud488","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[97.387,97.939,98.307,99.431,98.947,99.063,99.015,98.782,98.133,99.799,101.639,101.116,99.77,99.024,98.375,98.734,98.482,99.179,99.518,99.179,98.22,98.462,100.012,100.293,99.295,99.809,100.39,100.35,100.08,99.04,98.89,99.41,99.04,99.91,101.34,101.33,99.97,100.25,101.21,102.07,102.4,102.43,102.39,102.33,102.38,103.09,104.21,104.38,105.33,105.27,105.74,106.46,107.89,109.19,110.16,111.04],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\ub18d\ucd95\uc218\uc0b0\ubb3c<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c","\ub18d\ucd95\uc218\uc0b0\ubb3c"],"legendgroup":"\ub18d\ucd95\uc218\uc0b0\ubb3c","line":{"color":"#00cc96","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\ub18d\ucd95\uc218\uc0b0\ubb3c","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[87.968,90.058,92.106,96.765,93.794,94.18,92.509,90.217,90.771,99.166,104.874,100.207,94.641,94.7,94.406,95.397,93.525,94.801,93.634,91.804,90.511,91.955,96.32,96.421,92.064,93.953,95.63,94.73,96.57,95.78,96.98,97.4,98.45,103.3,108.68,108.45,101.74,102.3,105.7,109.97,109.41,108.4,107.94,106.63,106.81,109.06,112.01,109.01,109.47,110.3,112.31,111.72,109.89,110.42,112.45,111.8],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uacf5\uc5c5\uc81c\ud488<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488","\uacf5\uc5c5\uc81c\ud488"],"legendgroup":"\uacf5\uc5c5\uc81c\ud488","line":{"color":"#ab63fa","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uacf5\uc5c5\uc81c\ud488","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[99.399,99.635,99.615,99.969,100.047,100.097,100.46,100.735,100.43,100.607,100.804,101.265,100.813,99.773,98.967,99.213,99.35,99.969,100.725,100.725,100.391,100.362,100.568,100.912,100.637,100.872,101.24,101.4,100.61,99.42,98.93,99.54,99.92,99.89,99.77,99.81,99.61,99.86,100.42,100.55,101.13,101.41,101.47,101.7,102.34,102.74,102.8,103.63,104.79,104.51,104.61,105.73,108.08,109.32,109.86,111.19],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4"],"legendgroup":"\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","line":{"color":"#FFA15A","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[100.415,100.438,100.472,100.541,100.576,100.576,100.576,100.576,93.921,93.898,101.897,101.886,101.886,101.886,101.874,101.84,101.851,101.851,101.851,101.851,95.863,96.013,103.403,103.414,103.426,103.403,103.39,103.39,103.22,103.13,103.13,103.05,91.6,91.82,99.32,99.33,99.33,99.27,98.21,98.2,98.2,98.2,98.2,98.21,92.06,92.06,99.46,100.67,100.67,100.69,101.1,101.07,101.07,104.88,107.62,107.66],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uc11c\ube44\uc2a4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4","\uc11c\ube44\uc2a4"],"legendgroup":"\uc11c\ube44\uc2a4","line":{"color":"#19d3f3","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uc11c\ube44\uc2a4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[97.341,97.49,97.938,98.395,98.6,98.843,98.974,98.768,98.983,99.198,99.086,99.16,98.992,98.983,99.31,99.785,99.636,99.739,99.776,99.748,99.981,100.205,99.617,99.841,99.645,99.655,99.83,99.98,99.82,99.91,99.92,99.98,100.15,100.44,100.22,99.17,100.19,100.39,100.89,101.16,101.35,101.59,101.75,101.82,102.14,102.45,102.26,102.46,102.59,102.97,103.78,104.29,104.47,104.8,105.28,105.75],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uc9d1\uc138<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138","\uc9d1\uc138"],"legendgroup":"\uc9d1\uc138","line":{"color":"#FF6692","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uc9d1\uc138","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[99.639,99.648,99.725,99.792,99.821,99.859,99.898,99.898,99.917,99.936,99.936,99.946,99.955,99.936,99.926,99.917,99.879,99.84,99.811,99.744,99.725,99.696,99.696,99.706,99.715,99.735,99.79,99.81,99.83,99.86,99.88,99.91,99.96,100.03,100.09,100.19,100.28,100.37,100.46,100.64,100.84,100.98,101.14,101.25,101.39,101.58,101.76,101.93,102.18,102.42,102.57,102.73,102.88,102.99,103.13,103.22],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uacf5\uacf5\uc11c\ube44\uc2a4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4","\uacf5\uacf5\uc11c\ube44\uc2a4"],"legendgroup":"\uacf5\uacf5\uc11c\ube44\uc2a4","line":{"color":"#B6E880","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uacf5\uacf5\uc11c\ube44\uc2a4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[101.958,102.038,102.757,102.707,102.587,102.517,102.557,102.497,102.447,102.417,102.238,102.228,102.138,102.108,102.497,102.437,102.268,102.248,102.317,102.278,102.317,102.367,101.03,101.23,101.17,101.15,100.83,100.76,100.61,100.59,100.46,100.42,100.47,100.6,100.31,95.11,99.72,100.12,100.88,100.84,100.93,100.96,100.97,100.98,101.05,101.06,101.02,100.89,100.92,101.01,101.82,101.78,101.51,101.64,101.65,101.71],"yaxis":"y","type":"scatter"},{"hovertemplate":"<b>%{hovertext}</b><br><br>\ud488\ubaa9\uc131\uc9c8=\uac1c\uc778\uc11c\ube44\uc2a4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","hovertext":["\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4","\uac1c\uc778\uc11c\ube44\uc2a4"],"legendgroup":"\uac1c\uc778\uc11c\ube44\uc2a4","line":{"color":"#FF97FF","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uac1c\uc778\uc11c\ube44\uc2a4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[94.753,94.987,95.418,96.2,96.586,97.027,97.215,96.883,97.269,97.638,97.53,97.656,97.404,97.404,97.799,98.653,98.473,98.653,98.707,98.698,99.075,99.435,99.003,99.282,98.985,99.003,99.42,99.72,99.49,99.65,99.71,99.81,100.09,100.51,100.22,100.53,100.35,100.51,101.03,101.46,101.69,102.04,102.27,102.34,102.83,103.3,102.93,103.28,103.41,103.96,104.97,105.82,106.2,106.68,107.47,108.23],"yaxis":"y","type":"scatter"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Month"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Price index (2020=100)"}},"legend":{"title":{"text":"\ud488\ubaa9\uc131\uc9c8"},"tracegroupgap":0},"title":{"text":"<b>Price index Summary (2020=100)</b>"}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('9f36ad20-65f6-4cb7-8869-eeab10c79beb');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



```python
# x축, y축 항목 리스트로 정리 

month = df1['연월']
p_index = df1['총지수']
service = df1['서비스']
product = df1['상품']

# 그래프 
graph2 = go.Figure()

graph2.add_trace(go.Scatter(x = month, y = p_index, name='총지수',
                            mode='lines+markers',
                            line = dict(color = 'grey', width=4)))
graph2.add_trace(go.Scatter(x = month, y = service, name='서비스',
                            line = dict(color = 'blue', width=2))) # dash options include 'dash', 'dot', and 'dashdot'
graph2.add_trace(go.Scatter(x = month, y = product, name='상품',
                            line = dict(color = 'green', width=2)))

graph2.update_layout(title_text = '<b>소비자 물가지수 시각화(총지수|서비스|상품)</b>',
                     xaxis_title = 'Month',
                     yaxis_title = 'Price index (2020=100)'
                    # , paper_bgcolor = 'whitesmoke'
                    ,plot_bgcolor= 'whitesmoke'# 그래프 배경색
)
graph2.show()                     
```


<div>                            <div id="14a98ca9-223d-4880-b69b-5da8f6e779e5" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("14a98ca9-223d-4880-b69b-5da8f6e779e5")) {                    Plotly.newPlot(                        "14a98ca9-223d-4880-b69b-5da8f6e779e5",                        [{"line":{"color":"grey","width":4},"mode":"lines+markers","name":"\ucd1d\uc9c0\uc218","x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"y":[97.347,97.698,98.106,98.855,98.751,98.931,98.979,98.779,98.59,99.462,100.221,100.041,99.33,98.988,98.884,99.311,99.121,99.481,99.652,99.491,99.187,99.425,99.794,100.041,99.481,99.719,100.09,100.16,99.94,99.5,99.44,99.71,99.63,100.19,100.74,100.18,100.09,100.33,101.04,101.58,101.84,101.98,102.05,102.05,102.26,102.75,103.17,103.35,103.87,104.04,104.69,105.3,106.06,106.85,107.56,108.22],"type":"scatter"},{"line":{"color":"blue","width":2},"name":"\uc11c\ube44\uc2a4","x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"y":[97.341,97.49,97.938,98.395,98.6,98.843,98.974,98.768,98.983,99.198,99.086,99.16,98.992,98.983,99.31,99.785,99.636,99.739,99.776,99.748,99.981,100.205,99.617,99.841,99.645,99.655,99.83,99.98,99.82,99.91,99.92,99.98,100.15,100.44,100.22,99.17,100.19,100.39,100.89,101.16,101.35,101.59,101.75,101.82,102.14,102.45,102.26,102.46,102.59,102.97,103.78,104.29,104.47,104.8,105.28,105.75],"type":"scatter"},{"line":{"color":"green","width":2},"name":"\uc0c1\ud488","x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"y":[97.387,97.939,98.307,99.431,98.947,99.063,99.015,98.782,98.133,99.799,101.639,101.116,99.77,99.024,98.375,98.734,98.482,99.179,99.518,99.179,98.22,98.462,100.012,100.293,99.295,99.809,100.39,100.35,100.08,99.04,98.89,99.41,99.04,99.91,101.34,101.33,99.97,100.25,101.21,102.07,102.4,102.43,102.39,102.33,102.38,103.09,104.21,104.38,105.33,105.27,105.74,106.46,107.89,109.19,110.16,111.04],"type":"scatter"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"title":{"text":"<b>\uc18c\ube44\uc790 \ubb3c\uac00\uc9c0\uc218 \uc2dc\uac01\ud654(\ucd1d\uc9c0\uc218|\uc11c\ube44\uc2a4|\uc0c1\ud488)</b>"},"xaxis":{"title":{"text":"Month"}},"yaxis":{"title":{"text":"Price index (2020=100)"}},"plot_bgcolor":"whitesmoke"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('14a98ca9-223d-4880-b69b-5da8f6e779e5');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



```python
# df2에서 서비스 하위지수만 가져오기
# 집세, 공공서비스, 개인서비스

service_desc = df2.loc[(df2['품목성질']=='집세') | (df2['품목성질']=='공공서비스') | (df2['품목성질']=='개인서비스')]

# 그래프
graph3 = px.line(service_desc, x=service_desc['연월'], y=service_desc['물가지수'], color=service_desc['품목성질']
                 ,color_discrete_sequence=[
                 "lightsteelblue", "cornflowerblue", "royalblue"] )

graph3.add_trace(go.Scatter(x = month, y = service, name='서비스',
                            mode='lines+markers',
                            line = dict(color = 'blue', width=3)))

graph3.update_layout(title = '<b>물가지수(서비스지수|하위지수)</b>',
                     xaxis_title = 'Month',
                     yaxis_title = 'Price index (2020=100)'
                     # ,modebar_bgcolor= '' # 우측상단 모드바 배경색
                     # ,modebar_color= '' # 우측상단 모드바 아이콘색
                     # ,paper_bgcolor=  # 배경 색
                      ,plot_bgcolor= 'whitesmoke'# 그래프 배경색
)

graph3.show()
```


<div>                            <div id="88ef8c5e-f709-4faa-bee2-9f976c3f1f6e" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("88ef8c5e-f709-4faa-bee2-9f976c3f1f6e")) {                    Plotly.newPlot(                        "88ef8c5e-f709-4faa-bee2-9f976c3f1f6e",                        [{"hovertemplate":"\ud488\ubaa9\uc131\uc9c8=\uc9d1\uc138<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","legendgroup":"\uc9d1\uc138","line":{"color":"lightsteelblue","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uc9d1\uc138","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[99.639,99.648,99.725,99.792,99.821,99.859,99.898,99.898,99.917,99.936,99.936,99.946,99.955,99.936,99.926,99.917,99.879,99.84,99.811,99.744,99.725,99.696,99.696,99.706,99.715,99.735,99.79,99.81,99.83,99.86,99.88,99.91,99.96,100.03,100.09,100.19,100.28,100.37,100.46,100.64,100.84,100.98,101.14,101.25,101.39,101.58,101.76,101.93,102.18,102.42,102.57,102.73,102.88,102.99,103.13,103.22],"yaxis":"y","type":"scatter"},{"hovertemplate":"\ud488\ubaa9\uc131\uc9c8=\uacf5\uacf5\uc11c\ube44\uc2a4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","legendgroup":"\uacf5\uacf5\uc11c\ube44\uc2a4","line":{"color":"cornflowerblue","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uacf5\uacf5\uc11c\ube44\uc2a4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[101.958,102.038,102.757,102.707,102.587,102.517,102.557,102.497,102.447,102.417,102.238,102.228,102.138,102.108,102.497,102.437,102.268,102.248,102.317,102.278,102.317,102.367,101.03,101.23,101.17,101.15,100.83,100.76,100.61,100.59,100.46,100.42,100.47,100.6,100.31,95.11,99.72,100.12,100.88,100.84,100.93,100.96,100.97,100.98,101.05,101.06,101.02,100.89,100.92,101.01,101.82,101.78,101.51,101.64,101.65,101.71],"yaxis":"y","type":"scatter"},{"hovertemplate":"\ud488\ubaa9\uc131\uc9c8=\uac1c\uc778\uc11c\ube44\uc2a4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","legendgroup":"\uac1c\uc778\uc11c\ube44\uc2a4","line":{"color":"royalblue","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uac1c\uc778\uc11c\ube44\uc2a4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[94.753,94.987,95.418,96.2,96.586,97.027,97.215,96.883,97.269,97.638,97.53,97.656,97.404,97.404,97.799,98.653,98.473,98.653,98.707,98.698,99.075,99.435,99.003,99.282,98.985,99.003,99.42,99.72,99.49,99.65,99.71,99.81,100.09,100.51,100.22,100.53,100.35,100.51,101.03,101.46,101.69,102.04,102.27,102.34,102.83,103.3,102.93,103.28,103.41,103.96,104.97,105.82,106.2,106.68,107.47,108.23],"yaxis":"y","type":"scatter"},{"line":{"color":"blue","width":3},"mode":"lines+markers","name":"\uc11c\ube44\uc2a4","x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"y":[97.341,97.49,97.938,98.395,98.6,98.843,98.974,98.768,98.983,99.198,99.086,99.16,98.992,98.983,99.31,99.785,99.636,99.739,99.776,99.748,99.981,100.205,99.617,99.841,99.645,99.655,99.83,99.98,99.82,99.91,99.92,99.98,100.15,100.44,100.22,99.17,100.19,100.39,100.89,101.16,101.35,101.59,101.75,101.82,102.14,102.45,102.26,102.46,102.59,102.97,103.78,104.29,104.47,104.8,105.28,105.75],"type":"scatter"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Month"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Price index (2020=100)"}},"legend":{"title":{"text":"\ud488\ubaa9\uc131\uc9c8"},"tracegroupgap":0},"margin":{"t":60},"title":{"text":"<b>\ubb3c\uac00\uc9c0\uc218(\uc11c\ube44\uc2a4\uc9c0\uc218|\ud558\uc704\uc9c0\uc218)</b>"},"plot_bgcolor":"whitesmoke"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('88ef8c5e-f709-4faa-bee2-9f976c3f1f6e');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



```python
# df2에서 상품 하위지수만 가져오기
# 농축수산물, 공업제품, 전기 · 가스 · 수도

product_desc = df2.loc[(df2['품목성질']=='농축수산물') | (df2['품목성질']=='공업제품') | (df2['품목성질']=='전기 · 가스 · 수도')]

# 그래프 

graph4 = px.line(product_desc, x=product_desc['연월'], y=product_desc['물가지수']
                 , color=product_desc['품목성질']
                 , color_discrete_sequence=[
                 "darkseagreen","yellowgreen", "olivedrab"])

graph4.add_trace(go.Scatter(x = month, y = product, name='상품',
                            mode='lines+markers',
                            line = dict(color = 'green', width=3)))

graph4.update_layout(title = '<b>물가지수(상품지수|하위지수)</b>',
                     xaxis_title = 'Month',
                     yaxis_title = 'Price index (2020=100)'
                     , plot_bgcolor = 'whitesmoke'
                     )

graph4.show()
```


<div>                            <div id="f7b300e6-a872-40a7-8177-cfbb6a5d04f1" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("f7b300e6-a872-40a7-8177-cfbb6a5d04f1")) {                    Plotly.newPlot(                        "f7b300e6-a872-40a7-8177-cfbb6a5d04f1",                        [{"hovertemplate":"\ud488\ubaa9\uc131\uc9c8=\ub18d\ucd95\uc218\uc0b0\ubb3c<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","legendgroup":"\ub18d\ucd95\uc218\uc0b0\ubb3c","line":{"color":"darkseagreen","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\ub18d\ucd95\uc218\uc0b0\ubb3c","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[87.968,90.058,92.106,96.765,93.794,94.18,92.509,90.217,90.771,99.166,104.874,100.207,94.641,94.7,94.406,95.397,93.525,94.801,93.634,91.804,90.511,91.955,96.32,96.421,92.064,93.953,95.63,94.73,96.57,95.78,96.98,97.4,98.45,103.3,108.68,108.45,101.74,102.3,105.7,109.97,109.41,108.4,107.94,106.63,106.81,109.06,112.01,109.01,109.47,110.3,112.31,111.72,109.89,110.42,112.45,111.8],"yaxis":"y","type":"scatter"},{"hovertemplate":"\ud488\ubaa9\uc131\uc9c8=\uacf5\uc5c5\uc81c\ud488<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","legendgroup":"\uacf5\uc5c5\uc81c\ud488","line":{"color":"yellowgreen","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uacf5\uc5c5\uc81c\ud488","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[99.399,99.635,99.615,99.969,100.047,100.097,100.46,100.735,100.43,100.607,100.804,101.265,100.813,99.773,98.967,99.213,99.35,99.969,100.725,100.725,100.391,100.362,100.568,100.912,100.637,100.872,101.24,101.4,100.61,99.42,98.93,99.54,99.92,99.89,99.77,99.81,99.61,99.86,100.42,100.55,101.13,101.41,101.47,101.7,102.34,102.74,102.8,103.63,104.79,104.51,104.61,105.73,108.08,109.32,109.86,111.19],"yaxis":"y","type":"scatter"},{"hovertemplate":"\ud488\ubaa9\uc131\uc9c8=\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4<br>\uc5f0\uc6d4=%{x}<br>\ubb3c\uac00\uc9c0\uc218=%{y}<extra></extra>","legendgroup":"\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","line":{"color":"olivedrab","dash":"solid"},"marker":{"symbol":"circle"},"mode":"lines","name":"\uc804\uae30 \u00b7 \uac00\uc2a4 \u00b7 \uc218\ub3c4","orientation":"v","showlegend":true,"x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"xaxis":"x","y":[100.415,100.438,100.472,100.541,100.576,100.576,100.576,100.576,93.921,93.898,101.897,101.886,101.886,101.886,101.874,101.84,101.851,101.851,101.851,101.851,95.863,96.013,103.403,103.414,103.426,103.403,103.39,103.39,103.22,103.13,103.13,103.05,91.6,91.82,99.32,99.33,99.33,99.27,98.21,98.2,98.2,98.2,98.2,98.21,92.06,92.06,99.46,100.67,100.67,100.69,101.1,101.07,101.07,104.88,107.62,107.66],"yaxis":"y","type":"scatter"},{"line":{"color":"green","width":3},"mode":"lines+markers","name":"\uc0c1\ud488","x":["2017.11","2017.12","2018.01","2018.02","2018.03","2018.04","2018.05","2018.06","2018.07","2018.08","2018.09","2018.10","2018.11","2018.12","2019.01","2019.02","2019.03","2019.04","2019.05","2019.06","2019.07","2019.08","2019.09","2019.10","2019.11","2019.12","2020.01","2020.02","2020.03","2020.04","2020.05","2020.06","2020.07","2020.08","2020.09","2020.10","2020.11","2020.12","2021.01","2021.02","2021.03","2021.04","2021.05","2021.06","2021.07","2021.08","2021.09","2021.10","2021.11","2021.12","2022.01","2022.02","2022.03","2022.04","2022.05","2022.06"],"y":[97.387,97.939,98.307,99.431,98.947,99.063,99.015,98.782,98.133,99.799,101.639,101.116,99.77,99.024,98.375,98.734,98.482,99.179,99.518,99.179,98.22,98.462,100.012,100.293,99.295,99.809,100.39,100.35,100.08,99.04,98.89,99.41,99.04,99.91,101.34,101.33,99.97,100.25,101.21,102.07,102.4,102.43,102.39,102.33,102.38,103.09,104.21,104.38,105.33,105.27,105.74,106.46,107.89,109.19,110.16,111.04],"type":"scatter"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Month"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Price index (2020=100)"}},"legend":{"title":{"text":"\ud488\ubaa9\uc131\uc9c8"},"tracegroupgap":0},"margin":{"t":60},"title":{"text":"<b>\ubb3c\uac00\uc9c0\uc218(\uc0c1\ud488\uc9c0\uc218|\ud558\uc704\uc9c0\uc218)</b>"},"plot_bgcolor":"whitesmoke"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('f7b300e6-a872-40a7-8177-cfbb6a5d04f1');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>



```python

```
