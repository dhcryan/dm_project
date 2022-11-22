# 2022-2 IT집중교육 Data Mining Project
## 주성분 분석 및 K-Means 클러스터링을 통한 축구 선수 스카우팅 시스템 :soccer:
`데이터 과학과 결합하여 괄목할 만한 성장을 이루고 있는 스포츠 산업이지만 선수를 영입하는 기준이 애매모호한 경우
가 많다. FIFA23 데이터에는 전 세계 축구선수들의 다양한 능력치가 있는데 이를 주성분 분석을 통해 차원 축소를 진행
하고 K-means 클러스터링을 통해 알맞은 군집 분석을 수행한다. 이를 다양한 시각화 방법을 활용하여 각 군집의 특성을
파악한 뒤에 같은 군집 내에 코사인 유사도를 구하여 이 수치가 높은 선수를 알려준다. 즉, 팀 내 핵심 선수의 대체자를 이
와 같은 데이터 마이닝 알고리즘을 활용하여 객관적이고 논리적으로 구할 수 있게 된다.`

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
4. **Conclusion**
-------------------
### 1. Data gathering & preprocessing
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
<br>
실제 축구에서 사용되는 xG(기대 득점:ExpectedGoals), xGA(기대 실점:Expected Goals Against) 등의 지표를 가져오기 위해 Understat 사이트에서 유럽 최상위 리그에 대한 자세한 
통계를 확인할 수 있다. xG란 팀이나 플레이어가 시도한 슛의 질과 양을 바탕으로 예상되는 골 수이다. xG값이 0.5이면 득점 확률이 50%이며 xG값은 슛의 질을 결정하는 좋은 방법이다. 반대로 xGA란 시도한 슛의 질과 양을 바탕으로 실점할 것으로 예상되는 팀의 골 수이다. 위와 같은 예측 지표를 통해 단순히 결과만 보던 기존 분석 방식에서 벗어나 향후 구단의 성적을 예측해볼 수 있다. <br>
경기별로 나누어져 있던 통계를 연산을 통해 구단별로 feature별 한 가지 값만 갖도록 한다.

`EPL_df`는 리그 순위별로 각 팀들의 정보를 알려주는 dataframe이다.
|position|team|matches|wins|draws|loses|scored|missed|pts|xG|xG\_diff|npxG|xGA|xGA\_diff|npxGA|npxGD|ppda|ppda\_allowed|deep|deep\_allowed|xpts|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|1|Arsenal|14|12|1|1|33|11|37|29\.543937|-3\.4560630000000003|28\.782767|11\.810814000000002|0\.8108140000000024|10\.288470000000002|18\.494296999999996|9\.973419962163158|15\.17651114465703|158|58|31\.0803|
|2|Manchester City|14|10|2|2|40|14|32|30\.742790000000003|-9\.257209999999997|28\.459290000000003|13\.164878|-0\.8351220000000001|11\.6425348|16\.8167552|12\.263169896798603|30\.42601543487325|193|44|30\.936700000000002|
|3|Newcastle United|15|8|6|1|29|11|30|28\.426177999999997|-0\.5738220000000034|26\.903844999999997|17\.4519147|6\.4519147|16\.6907477|10\.213097299999998|10\.397541622377325|12\.961694786694785|158|90|26\.3167|
|4|Tottenham|15|9|2|4|31|21|29|26\.087301999999998|-4\.9126980000000025|23\.803788|18\.065461000000003|-2\.9345389999999973|17\.304297|6\.499491000000001|14\.628979782400837|17\.10443888562043|107|114|25\.8902|
|5|Manchester United|14|8|2|4|20|20|26|21\.848452|1\.8484520000000018|21\.087277999999998|17\.343279|-2\.656721000000001|16\.582105|4\.505173000000001|14\.198471052660716|14\.210457722225192|114|112|22\.421499999999998|
|6|Liverpool|14|6|4|4|28|17|22|29\.580370999999996|1\.580370999999996|29\.580370999999996|23\.09607|6\.096070000000001|20\.812557000000005|8\.767814|11\.409020995218798|22\.577947710184553|165|97|22\.440600000000003|
|7|Brighton|14|6|3|5|23|19|21|24\.895262000000006|1\.895262000000006|22\.611748000000002|16\.068099999999998|-2\.9319000000000024|13\.023428|9\.588320000000001|11\.39068463924031|16\.644709702062645|128|89|25\.5227|
|8|Chelsea|14|6|3|5|17|17|21|16\.81194|-0\.18806000000000012|15\.289595999999998|21\.865976999999997|4\.865976999999997|21\.865976999999997|-6\.576381|9\.474393878415176|16\.439409482430964|100|62|15\.7153|
|9|Brentford|15|4|7|4|23|25|19|23\.450046999999998|0\.45004699999999787|20\.405394|22\.633553999999997|-2\.3664460000000034|21\.872383999999997|-1\.46699|12\.500112185372247|10\.627552478048488|88|139|22\.0899|
|10|Fulham|15|5|4|6|24|26|19|21\.512275999999996|-2\.4877240000000036|17\.7064258|32\.605482|6\.605482000000002|30\.321968|-12\.6155422|16\.19237058263374|13\.53618455131613|71|116|15\.323900000000002|
|11|Crystal Palace|14|5|4|5|15|18|19|15\.297109|0\.29710900000000073|13\.631195000000002|20\.259949999999996|2\.2599499999999964|20\.259949999999996|-6\.628755|18\.732062817499237|10\.146462929374463|63|133|15\.8744|
|12|Aston Villa|15|5|3|7|16|22|18|17\.354909|1\.3549089999999993|15\.832567000000001|21\.336209999999998|-0\.6637900000000023|18\.909129999999998|-3\.0765629999999997|13\.982148845203122|12\.711655165167317|93|104|20\.2426|
|13|Leicester|15|5|2|8|25|25|17|16\.240655|-8\.759345|14\.718323999999999|21\.077363000000005|-3\.9226369999999946|20\.316193000000002|-5\.597869000000002|16\.534367734639414|18\.808202446790684|96|107|17\.144700000000004|
|14|Bournemouth|15|4|4|7|18|32|16|13\.324752999999998|-4\.675247000000002|13\.324752999999998|25\.408921|-6\.591079000000001|21\.603063000000002|-8\.278310000000001|21\.27518570255069|9\.286105316629431|75|149|13\.093|
|15|Leeds|14|4|3|7|22|26|15|21\.186476000000003|-0\.8135239999999975|19\.664136000000003|22\.278798|-3\.7212020000000017|21\.517637999999998|-1\.8535020000000002|9\.969121804841707|11\.199083391913998|95|89|18\.775399999999998|
|16|Everton|15|3|5|7|11|17|14|16\.2209987|5\.220998699999999|16\.2209987|25\.378982999999998|8\.378982999999998|23\.856643000000002|-7\.635644300000001|16\.629291178620793|10\.93977756542974|80|123|14\.2567|
|17|West Ham|15|4|2|9|12|17|14|19\.594012|7\.594011999999999|16\.549332|17\.530361|0\.5303609999999992|15\.246864|1\.3024679999999993|19\.268269508269505|16\.916086083215777|66|83|21\.754800000000003|
|18|Nottingham Forest|15|3|4|8|11|30|13|15\.529207000000001|4\.529207000000001|14\.006869000000002|26\.719358000000007|-3\.280641999999993|22\.91352|-8\.906651|19\.673632697356144|10\.161905333608239|57|149|14\.174099999999997|
|19|Southampton|15|3|3|9|13|27|12|16\.154818|3\.154817999999999|16\.154818|23\.15029|-3\.8497100000000017|23\.15029|-6\.995472000000001|14\.110154475792408|10\.028090637841087|93|119|15\.969899999999997|
|20|Wolverhampton Wanderers|15|2|4|9|8|24|10|14\.225281000000003|6\.2252810000000025|12\.702943000000003|20\.781010000000002|-3\.218989999999998|19\.258678999999997|-6\.555736|16\.803715867970144|12\.1311746266655|91|114|15\.491599999999998|

`The expected score (xG) is a new revolutionary football indicator that can evaluate the performance of teams and players.`
<br>

### 2. Problem and Solution Approach
#### 용어 설명
PPDA: 압박 강도(우리 팀)<br>
PPDA_allowd: 압박 강도(상대 팀)<br>
xG_diff : xG와 실제 득점 수의 차이, 이 값이 양의 방향으로 클수록 기대 득점에 비해 골을 넣지 못했으며 팀의 골결정력이 낮다고 볼 수 있다.<br>
xGA_diff : xGA와 실제 실점 수의 차이, 이 값이 음의 방향으로 클수록 기대 실점에 비해 더 많은 실점을 기록했으며 팀의 수비력이 좋지 않다고 볼 수 있다.<br>
xpts_diff: xpts와 실제 승점간의 차이, 이 값이 양의 방향으로 클수록 기대 승점에 비해 승점을 쌓지 못했다고 볼 수 있다.

#### Problem
`xG_diff>EPL_df['xG_diff'].mean(), xGA_diff<EPL_df['xGA_diff'].mean(), xpts_diff>EPL_df['xpts_diff'].mean()` 세 조건 모두 해당되는 팀은 하위권일 확률이 높다.<br>
xG & xGA difference 를 통해 알아보는 강등권 팀 지표<br>
![EPL_XDIFF](https://user-images.githubusercontent.com/98611647/203234362-131f59c1-2796-4133-aada-1d51b9a805f8.png)
<br>
이를 시각화하여 표현할 경우 그림에서 볼 수 있듯이 xGA_diff의 평균값을 x축, xG_diff의 평균값을 y축이라 가정했을 때, 제 사분면에 해당하는 팀들이 하위권일
확률이 높으며 실제로 중위권인 Brighton(Brighton &Hove Albion FC)을 제외하곤 강등권에 가까운 팀들임을 확인할 수 있다.<br>

#### PCA
FIFA23 데이터에 대해 주성분 분석을 수행하였다. 우선 앞서 필드 플레이어 데이터는 표준화(standardazation)를 통해 데이터 스케일링(data scaling)을 수행하여 분산량이 
왜곡되는 것을 막는다. 그 후 7개의 선수 특성 관련 데이터를 차원 축소를 통해 줄이고자 한다. 아래의 그림2는 주성분 각각의 고윳값을 고윳값 전체를 더한 값으로 나눠 준 
것이며 이를 통해 해당 주성분의 고윳값이 차지하는 비율을 알 수 있다. 또한 알고리즘을 통해 누적 분산량이 전체 중 95%를 넘을 때 기준으로 주성분 개수(n_components)를 
4개로 결정하였다. 즉, 4개의 주성분으로 전체 분산의 95% 이상을 설명할 수 있다는 뜻이다.<br>
![PCA](https://user-images.githubusercontent.com/98611647/203235603-df427acf-5a54-4ca5-a47f-e3ed25fffd70.png)
<br>
#### K-Means 클러스터링
PCA를 통해 차원 축소한 데이터는 원래의 데이터에서 변형이 되었기 때문에 의미를 찾아내고 시각화하기 위해서는 K-Means 클러스터링이 필요하다. 그림을 통해 보면 K-means는 
미리 클러스터 수 k를 지정해야 하는데, 같은 군집 내 WCSS(Within Clusters Sum of Squares)가 급격히 완만해지는 구간의 k를 선택하는 ‘Elbow Method’를 사용했다. 그림에서 
제일 많이 구부러지는 구간이 5라고 판단, k 값을 5로 설정했다.<br>
![kmeans](https://user-images.githubusercontent.com/98611647/203236308-a42be7e1-595c-4ad7-8965-9d9eb7149f10.png)
<br>
PCA와 클러스터링을 마친 필드플레이어 데이터를 보기 좋게 정렬하여 `nongk_pca_kmeans_ordered_df` 를 따로 만들어둔다.<br>
|index|FullName|Cluster|BestPosition|PCA Component 1|PCA Component 2|PCA Component 3|PCA Component 4|
|---|---|---|---|---|---|---|---|
|0|Lionel Messi|4|CAM|6\.726875065674799|0\.34223703992104687|0\.6151663857887155|0\.1818814577997888|
|1|Karim Benzema|4|CF|5\.81627982229692|-0\.33601425783181144|0\.13266939982782247|1\.3940524811560353|
|2|Robert Lewandowski|4|ST|5\.701663691690086|-1\.0781429485361387|0\.18994213169147892|1\.9998437341982238|
|3|Kevin De Bruyne|4|CM|5\.87885244944403|-1\.4075809716525158|0\.854529663227299|-0\.41736482679129916|
|4|Kylian Mbappé|4|ST|6\.356857994119857|0\.24921799047173887|-1\.293110204879021|0\.9641799809836442|
<br>

#### Visualization (시각화)
**PCA component 2개를 활용해 2차원으로 시각화**<br>
![2d](https://user-images.githubusercontent.com/98611647/203243205-65ecf17c-a57c-4637-88a0-b9e16ab11c9a.png)<br>
군집화는 잘 됐지만 pca component들이 무엇을 의미하는지 모른다<br>
![3d](https://user-images.githubusercontent.com/98611647/203243306-829d4222-cbdb-4ace-bbd2-577a41579671.png)<br>

### 3. Radar chart for similar player
방사형 차트(radar chart)를 통해 시각화하고 데이터프레임의 선수들과 비교해보면 군집 분석 결과를 쉽게 설명할 수 있다. 군집마다
주성분 분석을 하기 전의 원래 필드 플레이어 Feature에 투영하여 특징을 정확하게 파악할 수 있다. 
![radar](https://user-images.githubusercontent.com/98611647/203243506-e5fb2cdc-72be-4af2-8ec0-2c16435c0eca.png)
<br>
0번, 3번 군집은 거의 모양이 유사한 공격수 군집이다. 하지만 속도, 슛, 힘 등 전반적인 능력치가 3번 군집이 더 좋다. 1번, 2번 군집도 마찬가지로 서로 모양이 비슷한 수비수 군집이지만 1번 군집의 전반적인 능력치가 더 좋다. 마지막으로 4번 군집은 거의 모든 능력치가 골고루 좋으며 all-round player, 혹은 수준급 선수들의 군집으로 분류됨을 확인할 수 있다. 즉, 각 군집의 특성을 정확한 세부 능력치를 통해 파악할 수 있다.<br>
![cluster0](https://user-images.githubusercontent.com/98611647/203243998-e3857437-77d3-446f-b2a3-d3cfa83af89b.png)
![cluster1](https://user-images.githubusercontent.com/98611647/203244383-a731fc41-838b-4afe-ad7e-7c08fb5086ee.png)
![cluster2](https://user-images.githubusercontent.com/98611647/203244426-aab2b775-a085-4990-bed0-e0ae41e0b942.png)
![cluster3](https://user-images.githubusercontent.com/98611647/203244479-1cb342b7-c37d-4548-84c5-fdbed3261d7c.png)
![cluster4](https://user-images.githubusercontent.com/98611647/203244510-aa140f67-4a66-4bbc-aa67-614b42de74c8.png)

<br>
앞서 군집화한 데이터들을 바탕으로 입력한 선수를 대체할 수 있을 만큼의 유사한 선수를 알려주는 프로그램을 만들 수 있다. 본 논문에서는 다
양한 유사도 공식 중에 제일 보편적인 코사인 유사도 공식(Cosine similarity)에 따른 유사도 분석을 수행할 것이다.  이 기법은 서로 다른 두 개의 벡터의 유
사도를 측정하는 데 특화된 수식으로 클러스터링 연구 분야에서 많이 활용되는 기법이다.
<br>

`ipywidgets` 는 UI 라이브러리로 함수를 전달하면 셀렉트 박스나 슬라이더의 조작으로 인수를 변경하면서 함수 실행 가능하여 이를 통해 선수를 검색해 볼 수 있다.
<br>
![sonny](https://user-images.githubusercontent.com/98611647/203247139-8cacaa01-52a4-42fb-80c9-a4a689b146e6.png)

![radar_sonny](https://user-images.githubusercontent.com/98611647/203247147-b4836aa9-a896-4a52-8fc0-ac66a0fda6b8.png)

선수 이름(FullName)을 입력한 뒤에 코사인 유사도를 구하는 함수를 만들어 실행할 수 있다. 예를 들어 손흥민(Heung Min Son) 선수를 입력값으로 할 때 같은 군집 안에 
있는 모든 데이터와 유사도를 계산하여 그 수치가 높은 순서대로 출력한다. <br>
|index|FullName|Similarity|
|---|---|---|
|0|Kai Havertz|0\.99759|
|1|José Luis Morales Nogales|0\.99744|
|2|João Félix Sequeira|0\.99724|
|3|Christian Bernardi|0\.99709|
|4|Jack Harrison|0\.99682|
|5|Jarrod Bowen|0\.99632|
|6|Nikola Vlašić|0\.99617|
|7|Ali Gholizadeh|0\.99544|
|8|Nabil Fekir|0\.99451|
|9|Domenico Berardi|0\.99450|
<br>
또한 이번엔 선수의 특성을 방사형 차트에 나타내는 알고리즘을 만들어 확인해 보면 그림과 같이 선수와 유사한 능력치 혹은 그래프 모양을
가진 선수들이 similar player로 제대로 출력되는 것을 확인할 수 있다. 이들의 포지션(Position)은모두 공격수이며 손흥민 선수처럼 빠른 속도를
가진 선수들이다. 이들의 손흥민 선수에 대한 유사도가 높게 나온 것을 보면 앞서 수행한 데이터 마이닝 알고리즘들이 정상적으로 실행되었음을 확인할 수 있다.<br>

![sonny_similar](https://user-images.githubusercontent.com/98611647/203247826-a10904ed-d62b-46dd-80be-852c047f838d.png)
<br> 유명선수인 `크리스티아누 호날두 (Cristiano Ronaldo)` 선수에 대해서도 확인할 수 있다. 최근 방출설이 나오므로 구단 입장에서는 이 알고리즘을 통해 대체자
를 찾을 수 있다.<br>
![ronaldo](https://user-images.githubusercontent.com/98611647/203248521-24b6f01d-8061-4ee6-848d-a6f5d553d253.png)<br>

![cr7_radar](https://user-images.githubusercontent.com/98611647/203248535-ccdbfedc-9719-4ab7-9271-4deec20e64a5.png)<br>

|index|FullName|Similarity|
|---|---|---|
|0|Karim Benzema|0\.99807|
|1|Antonio José Rodríguez Díaz|0\.99744|
|2|Odsonne Edouard|0\.99612|
|3|João Pedro G\. Santos Galvão|0\.99565|
|4|Wissam Ben Yedder|0\.99541|
|5|Rodrigo Moreno Machado|0\.99533|
|6|Anthony Nwakaeme|0\.99489|
|7|Gerard Moreno Balagueró|0\.99383|
|8|Davidson Da Luz Pereira|0\.99379|
|9|Danny Ings|0\.99290|
<br>

![cr7_similar](https://user-images.githubusercontent.com/98611647/203248742-4b21f35d-a384-4a25-a25f-0df4d2729e4e.png)


### 4. Conclusion
xG, xGA, xpts와 같은 실제 축구에서 사용 중인 데이터를 통해 구단의 문제점을 파악하고 이에 대한 해결책을 게임 데이터를 통해 찾아 다양한 데이터들을 활용했다는
데 의의가 있다. 실제로 선수를 나타내는 데이터가 상당히 많아서 K-means 클러스터링을 바로 하지 않고 주성분 분석으로 적절히 데이터를 처리한 뒤에 클러스터링하
여 더 좋은 군집화 성능을 보였다. 이를 바탕으로 유사도 분석까지 하여 선수들의 특성을 파악하고 대체자를 추천해 주는 알고리즘을 만들어 앞서 말한 구단의 문제점을
해결하는 방안을 만들었다고 생각한다. 다만 보완해야 할 점이 몇 가지가 있다. 본 논문은 필드 플레이어에 대해서만 데이터 마이닝을 수행하여서 골키퍼를 고려하지 않
았다. 또한 알고리즘적 요소나 앱으로 구현한 요소가 부족했다고 생각한다. 마지막으로 본 논문은 구단에서 활약이 좋은 선수의 대체자 영입에 집중했지만 활약이 저조한
선수일 경우 이들의 능력치를 기반으로 유사도 분석하면 유의미한 결과를 얻기 어렵다. 따라서 해당 구단의 전술을 분석하여 그에 맞는 새로운 선수를 찾는 알고리즘을
추후 고안해볼 수 있다.
