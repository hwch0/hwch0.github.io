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




```python

```
