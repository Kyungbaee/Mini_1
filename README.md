### KOSIS - 대한민국 17개 도시 범죄 유형별 발생빈도
### 데이터분석, 데이터프레임 구축 및 시각화

1. **대한민국 17개 도시 영역별 시각화**

```python
# 필요한 라이브러리 불러오기, josn 데이터 로드하기 때문에 json도 추가.
import pandas as pd
import plotly.express as px
import json

# geojson 데이터 열기
with open('지도데이터이름', encoding='UTF-8') as f:
    data = json.load(f)

# json 데이터 안에 features > properties > CTP_KOR_NM <- 요놈이 지역별 이름
# json 데이터 안에 features > properties > coordinates <- 요놈이 지역별 위도,경도
# json 데이터 안에 CTP_KOR_NM 값을 우리가 필요한 데이터 지역이름과 똑같게 맞춘다.
# 아래 FOR문 두개는 지역이름 확인할 때만 사용

for x in data['features']:
    x['id'] = x['properties']['CTP_KOR_NM'] 
    
for idx, _ in enumerate(data['features']):
    print(data['features'][idx]['id'])
    
# 기존의 지역별 데이터 이름을 CTP_KOR_NM과 같이 맞춰준다. KEY값으로 쓸 예정
mapper = [
    ('경기', '경기도'),
    ('서울', '서울특별시'),
    ('충북', '충청북도'),
    ('인천', '인천광역시'),
    ('충남', '충청남도'),
    ('광주', '광주광역시'),
    ('부산', '부산광역시'),
    ('강원', '강원도'),
    ('전남', '전라남도'),
    ('대구', '대구광역시'),
    ('전북', '전라북도'),
    ('울산', '울산광역시'),
    ('제주', '제주특별자치도'),
    ('경북', '경상북도'),
    ('세종', '세종특별자치시'),
    ('경남', '경상남도'),
    ('대전', '대전광역시'),
]
get_region = lambda gubun: [x[1] for x in mapper if x[0] == gubun][0]
데이터프레임이름['geo_region'] = 데이터프레임이름.발생지역별.apply(get_region)

# choropleth_mapbox로 시각화한다.
fig = px.choropleth_mapbox(
   데이터프레임이름, 
   geojson=data,                            # 위에서 load한 geojson 데이터 이름
   locations='geo_region',                  # CTP_KOR_NM과 맞춘 지역별 이름 컬럼
   color='2014 년',                         # 색으로 칠할 시리즈 데이터
   color_continuous_scale="Viridis",        # 색 조합
   featureidkey="properties.CTP_KOR_NM",    # featureidkey를 사용하여 id 값을 갖는 키값 지정
   mapbox_style="carto-positron",           # 지도 스타일
   zoom=5.5,
   center = {"lat": 35.757981, "lon": 127.661132},
   opacity=0.6,
   labels={'localocccnt'}
)
fig.update_geos(fitbounds="locations", visible=False)
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
```

![Untitled](https://user-images.githubusercontent.com/105343823/174715082-7e52939a-152b-4036-a697-7533bf2d6e0c.png)


2. **전국 범죄율에 따른 heat map 시각화** 

```python
# 필요한 라이브러리 불러오기
import pandas as pd
import plotly.express as px

# 2014-2020년 강력범죄(폭력) 데이터 불러오기 및 전처리
df = pd.read_csv("파일이름.csv",encoding="cp949").iloc[:,:-1]
df = df[df["발생지역별"] != "계"]

# 폭력 범죄 건수와 지역별 인구 수집
population = df[df["범죄별"] == "인구(B)"]
crime = df[df["범죄별"] == "강력범죄(폭력)"]

# 폭력 범죄 건수와 지역별 인구 전처리
num_pop = population.iloc[:,4:]
num_crime = crime.iloc[:,4:]

# 지역별 범죄율 = 범죄 건수 / 지역별 인원 * 100으로 수치화
num_crime = num_crime.astype("float")
num_pop = num_pop.astype("float")
ratio_list = [num_crime.iloc[i] / num_pop.iloc[i] * 100 for i in range(len(num_pop))]

# 지역별 범죄율 데이터프레임 생성 및 컬럼, 인덱스 수정
ratio_list_df = pd.DataFrame(ratio_list, columns=num_crime.columns)
crime_df = pd.DataFrame(crime["발생지역별"]).reset_index(drop=True)
ratio_table = pd.concat([ratio_list_df,crime_df],axis=1).set_index("발생지역별")

# 히트맵으로 시각화
px.imshow(ratio_table.T,width=800,title="2014년~2020년 전국 걍력범죄(폭력) 범죄율")
```

![Untitled 1](https://user-images.githubusercontent.com/105343823/174715109-d55dc1c8-b054-474b-9009-18041439688d.png)
