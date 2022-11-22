# 2022-2 IT집중교육 Data Mining Project
## 주성분 분석 및 K-Means 클러스터링을 통한 축구 선수 스카우팅 시스템 :soccer:
데이터 과학과 결합하여 괄목할 만한 성장을 이루고 있는 스포츠 산업이지만 선수를 영입하는 기준이 애매모호한 경우
가 많다. FIFA23 데이터에는 전 세계 축구선수들의 다양한 능력치가 있는데 이를 주성분 분석을 통해 차원 축소를 진행
하고 K-means 클러스터링을 통해 알맞은 군집 분석을 수행한다. 이를 다양한 시각화 방법을 활용하여 각 군집의 특성을
파악한 뒤에 같은 군집 내에 코사인 유사도를 구하여 이 수치가 높은 선수를 알려준다. 즉, 팀 내 핵심 선수의 대체자를 이
와 같은 데이터 마이닝 알고리즘을 활용하여 객관적이고 논리적으로 구할 수 있게 된다.

Table of contents 목차
---
1. **Data gathering & preprocessing**
    * package import and download csv file 
      * [kaggle FIFA23](https://www.kaggle.com/datasets/bryanb/fifa-player-stats-database?select=FIFA23_official_data.csv "FIFA23")
    * missing value analysis
    * spark 데이터를 pandas 로 변환
    * 골키퍼를 제외한 필드 플레이어 데이터 분류 및 index 정리 
    * Web scraping from [Understat.com](https://understat.com/) and find team with problems
2. **Problem and Solution Approach**
    * Problem
    * PCA
    * K-Means Clustering
    * Visualization
3. **Radar chart for similar player**
    * Cosine similarity
    * Search player name and show top10 similar player
-------------------
### Data gathering & preprocessing
이 프로젝트를 위해 kaggle에 있는 FIFA23 dataset를 활용한다. 이 데이터는 18,000명 이상의 선수와 90여개 정도의 신체 능력치, 포지션별 능력치, 세부 능력치를 feature(특징)로 
갖고 있다. 무엇보다도 선수의 재능을 발견하고 축구 클럽이 선수의 가치를 결정할 때 작용하는 특징을 스카우트 분석에 활용할 수 있다. FIFA23 data는 정제된 데이터이기 때문에 
결측치가 없다. 다만 column name과 column 안의 데이터에 공백이 있을 수가 있어 없애는 작업(trim)을 한다. 이후 선수의 데이터를 골키퍼(GK: goalkeeper), 필드 플레이어(Non_GK: field player)로 분류한 후 선수마다 능력치를 나타내는 feature data만 30여 개나 되기 때문에 특징을 잘 나타낼
수 있는 7개의 데이터(pass, shoot, pace, skill, movement, defense, physical) 로 재분류한다. 밑의 표는 재분류한 `nongk_df` 이다.
|index|pass\_index|shoot\_index|pace\_index|skill\_index|movement\_index|defense\_index|physical\_index|
|---|---|---|---|---|---|---|---|
|0|3\.43|2\.8|1\.2|3\.15|3\.67|-1\.22|0\.77|
|1|2\.45|2\.67|1\.02|2\.65|3\.09|-1\.15|2\.02|
|2|2\.24|2\.85|0\.65|2\.45|3\.09|-0\.85|2\.82|
|3|3\.52|2\.8|0\.56|2\.65|2\.86|0\.67|1\.2|
|4|2\.14|2\.58|2\.62|2\.95|3\.49|-1\.07|1\.76|
|5|2\.31|2\.7|1\.98|2\.65|3\.52|-0\.4|1\.45|
|6|2\.22|2\.93|1\.15|2\.4|3\.35|-1\.36|2\.02|
