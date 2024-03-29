# 강수 예측 모델

![WE44190704_ori](https://user-images.githubusercontent.com/90596549/236746108-776f3257-171c-4e60-95d8-eab1d38e2aaa.jpg)
(사진 출처: SBS 웹페이지, https://programs.sbs.co.kr/drama/christmas/main) <br>


질문! 오늘이 크리스마스 이브라고 가정해봅시다. 오늘 시드니의 습도가 31%고 북동풍이 분다면, 크리스마스에 눈이 올까요? <br> 
<br>
답을 얻기 위해 호주기상청의 기상데이터를 사용해서 내일의 강수 여부를 알려주는 모델을 만들었습니다. <br>

## 데이터 

<details>
<summary> 클릭하여 자세히 알아보기 </summary>

- 사용된 데이터는 22개 도시에서 2007년부터 2017년까지 매일 기록된 23가지 기상 정보 (앞으로는 변수라고 지칭하겠습니다.)를 포함합니다. <br>
- 2007년부터 2016년까지의 데이터는 트레이닝 데이터셋으로, 2017년의 데이터는 테스트셋으로 이용됐습니다.<br>
- 아래의 표를 통해 데이터를 조금 살펴보겠습니다. <br>
![test](https://user-images.githubusercontent.com/90596549/236992269-73b0e2bd-eece-4f39-b58c-c7527497686f.png) <br>
- 아래 표에 한국어로 변환된 변수명 기재했으니, 참고 부탁드려요 <br>
<img width="218" alt="스크린샷 2023-05-09 오후 1 34 08" src="https://user-images.githubusercontent.com/90596549/236995301-8160b5de-d468-47f3-bcd6-ec9f9718d382.png"> <br>
- 변수에는 범주형변수와 연속형변수가 섞여 있고, 결측치도 확인됩니다. <br>

### 전처리: 결측치 문제 
- 총 관측값 중 결측치가 35% 이상인 변수는 제거했어요. 대표성을 잃었다고 판단했거든요.  <br>
- 범주형 변수: 최빈값으로 대치했습니다. <br>
- 연속형 변수: 평균값으로 대치했습니다.  <br>
  
</details>

## 분석 방법 설정
### 왜 하필 혼합모형을? 

그거 아세요? 호주는 지역별로 정말 다른 기상특성을 가지고 있어요. 기상특성에 따라 크게 8개의 지역으로 나눌 수 있답니다. (아래 지도 참고) <br> 
![Australian-Climate-Zone-Map](https://user-images.githubusercontent.com/90596549/236750579-cad6fd38-f9e3-44de-8ac7-33b0e5f9fd52.jpg)<br>
(출처: 호주 건축법규 협회, https://www.abcb.gov.au/resources/climate-zone-map) <br>
그래서 저는 관측값들이 지역에 따라 서로 그룹을 이룰 것으로 예상하고 혼합모형을 사용하기로 했어요. 그리고 이런 예상을 추후에 통계적으로 확인해봤습니다. 

### 왜 하필 로지스틱모형을? 

- 제가 답하고 싶은 문제는 내일의 강수 여부입니다. 따라서 Yes! or No!로 답해 줄 로지스틱 모형이 가장 적절하다고 생각했답니다. 

### 그래서? 
- 두 이유를 근거로 로지스틱 일반화된 선형 혼합 모형으로 분석했습니다. (우리는 이걸 로지스틱 GLMM이라고 부르기로 했어요.)

## 다중공선성 문제

- 다중공선성: 회귀모델의 독립변수들 간에 강한 상관관계가 존재하는 것.<br>
- 왜 문제가 되나요? 회귀분석의 전제 가정을 위배하는 것으로 다중공선성을 지닌 모형은 회귀계수의 해석을 불가능하게 합니다.

<details>
<summary> 클릭하여 자세히 알아보기 </summary>
  
### Correlogram과 VIF

- 독립변수간의 쌍별 상관관계 (pairwise correlation)을 스피어만 상관계수를 통해 확인합니다. <br>
- Correlogram을 통해 상관계수와 상관관계의 정도를 시각화했습니다. (아래 참고)

![무제](https://user-images.githubusercontent.com/90596549/236833387-06c538a0-75b6-4dc3-9933-901f6612b803.png) <br>

- 우측 하단 노란색 원을 보면, Temp (기온) 관련 변수들 간에 강한 상관관계가 확인됩니다. <br>
- 좌층 중앙 파란색 원을 보면, WindSpeed (풍속) 관련 변수들 간에 강한 상관관계가 확인됩니다. <br>
- Pressure9am과 Pressure3pm은 각각 오전 9시와 오후 3시에 관측된 기압이며, 변수들 간에 강한 상관관계가 확인됩니다. <br>  
- Humidity9am과 Humidity3pm은 각각 오전 9시와 오후 3시에 관측된 습도며, 변수들 간에 강한 상관관계가 확인됩니다. <br>  

- 아래의 그래프를 통해 VIF를 확인해보겠습니다. <br>
<img width="645" alt="스크린샷 2023-05-08 오후 10 38 04" src="https://user-images.githubusercontent.com/90596549/236838780-f77ba53d-f38d-40c7-bd2f-d49ebc09386f.png"> <br>  
- WindGustDir, WindDir3pm은 풍향 관련 변수이고, 우리는 단위벡터 접근법으로 평균 풍향을 구해 이 두 변수를 대치할겁니다. <br>
- 관측지 변수 (Location)는 앞서 임의효과로 지정했기에 VIF는 무시해도 좋습니다. <br>
- 마지막으로 강수량 변수의 경우 (Rainfall) 히스토그램을 통해 확인 시 오른쪽으로 긴 꼬리를 가지고 있고, 강수량이 0인 경우, $log(0)$ 문제가 생길 수 있어 $log(1+Rainfall)$로 로그변환을 해주겠습니다. (아래 히스토그램 참고)  
<img width="650" alt="스크린샷 2023-05-08 오후 10 48 57" src="https://user-images.githubusercontent.com/90596549/236841348-58c078c4-565d-412e-806a-ef01f6a69891.png">

#### 해결

- 하루 중 최고기온은 주로 오후 2시에서 3시 사이에, 최저기온은 주로 오전 6시에서 7시 사이에 관측됩니다. <br>
- 오전 9시에 관측된 기온과 오후 3시에 관측된 기온 (Temp9am, Temp3pm으로 기록된 변수)은 각각 최고기온 및 최저기온 (MaxTemp, MinTemp로 기록된 변수)과 중복되는 경향이 있습니다.<br>
- 따라서, 우리는 일교차와 평균기온에 관한 변수를 새로 만들어서 오전 9시에 관측된 기온과 오후 3시에 관측된 기온, 최고 및 최저기온 변수를 대치할 것입니다. <br>
- 오전 9시와 오후 3시에 관측된 기압과 오전 9시와 오후 3시에 관측된 습도는 모두 오전 9시와 오후 3시 관측값의 평균인, 평균기압과 평균습도로 대치될 것입니다. <br>
- 새로운 변수로 대치한 후, Correlogram으로 확인해보니, 모든 독립변수의 상관계수의 값이 양호해보입니다. (아래 참고) <br>
![무제](https://user-images.githubusercontent.com/90596549/236837936-c0532910-3055-4a3c-a882-344981140333.png) <br>
- 또한, 풍향 관련 변수 또한 더이상 높은 VIF를 보이지 않습니다 <br>

</details>  

## 모델
### 영모형 (null model)
- 먼저, 관측값들이 지역에 따라 서로 그룹을 이룰 것이라는 전제를 하지 않고, 로지스틱 일반화 선형모형 (로지스틱 GLM)을 만들거에요.  <br>
- 변수들은 단계선택법 (stepwise selection)을 이용하여 AIC와 BIC를 기준으로 선택되었습니다. (AIC: 107075.7, BIC: 107291.9)
- 이 모델의 종속변수는 내일의 강수여부이고, 독립변수는 평균기온, 일교차, 평균풍향, 평균풍속, 평균기압, 평균습도입니다.  <br>
### 로지스틱 일반화 선형 혼합 모형 (GLMM)
- 이 모형은 앞서 생각한 전제에 따라 관측지 변수를 랜덤효과항 (Radom effects term)으로 설정했습니다. <br>
- 변수는 단계선택법을 이용해 선택되었습니다. (AIC: 105169.6, BIC: 105395.6)<br>
- 이 모델의 종속변수는 내일의 강수여부이고, 독립변수는 평균기온, 일교차, 평균풍향, 평균풍속, 평균기압, 평균습도입니다. <br>
- 관측지 변수는 Random intercept으로 회귀식에 포함시켰습니다. <br>
### 비교
아래의 표에서 빨간색과 파란색 박스로 강조된 두 모델을 봐주세요!<br>
<img width="716" alt="스크린샷 2023-05-08 오후 10 56 15" src="https://user-images.githubusercontent.com/90596549/237000727-013a505a-faaa-4fb7-82d7-f2af1372984d.png"><br>
- GLM 보다 GLMM의 AIC와 BIC가 훨씬 작은 것을 확인했습니다. 따라서, GLMM이 더 나은 모델이 되겠군요!<br>
- 모든 변수가 통계적으로 유의미합니다. <br>
### 모형적합도
<details>
<summary> Marginal Effects Plot과 QQplot을 통한 가정 확인 </summary>
  
GLMM의 적합도를 확인하기 위해 Mariginal Effects plot과 랜덤효과항의 QQplot을 그렸습니다. <br>
<img width="640" alt="스크린샷 2023-05-09 오후 2 23 26" src="https://user-images.githubusercontent.com/90596549/237001465-333ce9ff-d555-4185-bf14-798b714a3083.png"><br>
- Marginal effects plot을 통해 습도변수의 비선형선을 확인해 제곱해주는 방법으로 선형가정을 만족시켰습니다. <br>
- 습도값의 제곱이 기존의 습도변수를 대치한 모델이 바로 노란색 박스로 강조된 모델입니다. <br>
- 이 모델의 AIC값은 104,496 이고 BIC값은 104,722로 위에서 확인한 GLMM 모델보다 더 낮습니다. <br>
- 따라서 우리는 제곱된 습도변수를 사용하는 로지스틱 GLMM 모델을 최종 모형으로 사용할겁니다. <br>
- 또한, QQplot을 통해 정규분포가정도 만족함을 확인했습니다. <br>

 </details>
   
<details>
<summary> 잔차분석을 통한 등분산 가정 확인 </summary>  
   
- 아래의 표는 잔차값의 분포를 나타냅니다.  
- 잔차값이 신뢰 밴드 (Confidence band)안에 위치하는지 확인해 등분산 가정을 만족하는지 확인해볼게요. <br>
<img width="635" alt="스크린샷 2023-05-09 오후 2 29 53" src="https://user-images.githubusercontent.com/90596549/237002541-96871cde-d696-48de-8a26-4d35d5c1042c.png"> <br>
- 밴드 밖에 약간의 이상치가 있긴 하지만 그렇게 많지 않고 총 잔차의 95% 이상이 밴드안에 위치하는 것으로 보입니다. <br>
- 왼쪽에 잔차값이 뭉쳐있는 것 처럼 보이나 전반적으로 흩어져 있으므로 등분산 가정을 만족하는 것으로 보입니다. <br>
</details>

<details>
<summary> ROC 커브를 통한 모형 적합성 확인 </summary>     
   
- 아래의 ROC 커브를 통해 모형의 적합성을 확인해볼게요.<br>
<img width="695" alt="스크린샷 2023-05-09 오후 3 50 15" src="https://user-images.githubusercontent.com/90596549/237017035-ffa11db7-e5fd-4315-8a3f-1c00d9e1faa9.png"><br>
- 쉽게 말해 민감도는 진짜로 비가 왔을 때, 모델도 비가 온 것을 응답한 비율이고, <br>
- 특이도는 실제 비가 안 왔을 때, 모델 비가 안 왔다고 응답한 비율입니다. <br>
- ROC 커브는 민감도와 $1-$특이도 값으로 만들어진 곡선입니다. <br>
- 이 커브의 안쪽 면적의 넓이를 AUC라고 하며 비가 오고 안 오고를 구별할 수 있는 확률이 얼마나 되는지 알려줍니다. <br>
- 제 모델은 AUC가 0.84로 이는 모델이 강수여부를 구별할 수 있는 확률이 약 84% 정도 된다는 뜻입니다. <br>
- 이 정도면.. 뭐.. 괜찮은 모델이라고 할 수 있겠네요! <br>
</details>

## 해석
- 우린 오늘의 기상 데이터를 통해 내일의 강수여부를 확인하기 위해 모형을 만들었습니다. <br>
- 앞서 변수를 선정했던 과정은 사실, 우리가 사용하는 데이터셋 속에 포함된 여러 변수 중, 어떤 변수가 내일의 강수여부에 영향을 미치는 지 알아보는 과정이였습니다. 
- 그렇다면, 각각의 독립변수는 내일의 강수여부에 어떤 영향을 미치는지 알아보겠습니다. 

<details>
<summary> 오즈비 해석 </summary>  

| 변수   | 오즈비  | 해석 |  
|------|------|----|
| 평균기온 | 1.01 |평균 기온이 1도씨 증가하면, 비가 올 오즈가 1.01배 증가한다|  
| 일교차  | 0.95 |일교차가 1도씨 증가하면, 비가 올 오즈가 0.95배 증가한다.    |   
| 강수량  | 1.18 |강수량이 1mm 증가하면, 비가 올 오즈가 1.18배 증가한다.    |  
| 풍속   | 1.05 |풍속이 1km/sec 증가하면, 비가 올 오즈가 1.05배 증가한다.      |   
| 기압   | 0.93 |기압이 1 헥토파스칼 증가하면,비가 올 오즈가 0.93배 증가한다.    |   
| 습도   | 1    |습도가 1단위 증가하면, 비가 올 오즈가 1배 증가한다     | 

이렇게 우리는 개별적인 기상 관측치가 어떻게 내일의 강수여부에 영향을 미치는지 확인했습니다. 

- 앞으로는 오늘의 평균기온을 통해.. 내일의 강수여부를 확인해볼수... 있지 않을까요?

</details>






