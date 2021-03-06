# Ch_03 word2vec

2021/01/29(금) 박준영 정리

**3장 word2vec**

단어를 벡터로 표현하는 법은 크게 '통계 기반 기법' '추론 기반 기법' 2가지로 나눌 수 있다. 이 2가지 기법의 의미를 얻는 방식은 서로 크게
다르지만, 그 배경에는 분포 가설이 있다. 

'통계 기반 기법'에 대한 자세한 설명은 2장을 참조

**통계 기반 기법**은 단어의 동시발생 행렬을 만들고 그 행렬에 SVD를 적용하여 밀집벡터(단어의 분산표현)을 얻었는데 이 방식은 거대한 행렬에 SVD를 적용할때 연산량이 늘어나 적용하기 어렵다. 또, 학습 데이터 전체를  1회 학습하는 (배치학습)을 사용한다.

이에 반해, **추론 기반 기법**은 주변 단어의 '맥락'이 주어졌을때 단어를 추론하는 방법을 사용한다. 맥락정보를 입력받아 신경망모델으로 학습시켜 각 단어의 출현확률을 출력하는 방법이다. 또, 학습 데이터의 일부를 사용하는 (미니배치학습)을 사용하여 말뭉치가 많아 계산량이 큰 작업을 할 수 있고 병렬 계산도 진행할 수 있어서 학습 속도를 높일 수 있다. 


분산 표현의 측면에서 **통계 기반 기법**에서는 단어의 유사성이 인코딩되는 반면 **추론 기반 기법**은 단어의 유사성은 물론 단어사이의 패턴도 파악되어 인코딩 된다.

**인코딩** : 은닉층의 정보
<br>
**디코딩** : 인코딩된 정보로부터 원하는 결과를 얻는 작업

**word2vec**의 구조


- CBOW 모델 : input(맥락)-은닉층-출력층(타깃)-(softmax)-확률 (input의 갯수 조절가능)
- Skip-gram 모델 : input(타깃)-은닉층-출력층(맥락)-(softmax)-확률 (출력층의 갯수 조절가능)

단어의 분산표현도 정밀도 면에서 Skip-gram 모델의 결과가 좋다. 특히 말뭉치가 커질수록 저빈도 단어나 유추의 성능에 좋다. 그러나 학습속도는 CBOW 모델이 더 빠르다. 그 이유는 Skip-gram 모델은 손실을 맥락의 수만큼 구해야하기 때문이다.




**[정리]**

- 추론 기반 기법은 추측하는 것이 목적이며, 그 부산물로 단어의 분산 표현을 얻을 수 있다.
- word2vec은 추론 기반 기법이며, 단순한 2층 신경망이다.
- word2vec은 skip-gram모델과 CBOW 모델을 제공한다
- CBOW 모델은 여러 단어(맥락)으로부터 하나의 단어(타깃)을 추측한다.
- Skip-gram 모델은 하나의 단어(타깃)으로부터 다수의 단어(맥락)을 추측한다.
- word2vec은 가중치를 다시 학습할 수 있으므로, 단어의 분산 표현 갱신이나 새로운 단어 추가를 효율적으로 수행




**[파일 설명]**

cbow_predict : cbow 모델의 추론 처리를 구현한 코드입니다.
<br>
simple_cbow : simpole_cbow 모델을 구현한 코드입니다.
<br>
simple_skip_gram : skip_gram 모델을 구현한 코드입니다.
<br>
train : simple_cbow으로 'You say goodbye and I say hello.' 학습하는 코드입니다.
<br>




**[심화]**

- Glove
Glove는 기존의 카운트 기반의 LSA(latent semantic analysis)와 예측 기반의 word2vec의 단점을 보완하기 위해 등장하였다. 

LSA : TDM, TF-IDF 행렬같이 각 문서에서 각 단어의 빈도수를 카운트한 행렬을 축소하여 잠재된 의미를 끌어오는 방법 
<br>
Word2vec : 실제값과 예측값에 대한 오차를 손실함수를 통해 줄여나가며 학습하는 예측기반의 방법론이다.

LSA는 말뭉치 전체의 통계적인 정보를 모두 활용하지만, LSA 결과물을 가지고 단어/문서간 유사도 측정이 어렵다.
Word2vec은 사용자가 지정한 맥락의 갯수에서만 학습이 이루어지기때문에 말뭉치 전체의 통계정보를 모두 활용하기 어렵다.

따라서 Glove는 말뭉치 전체의 통계정보를 잘 반영하면서도 단어간 유사도 측정을 편하게 하기위해 **동시등장확률**을 고려한다.

![image](https://user-images.githubusercontent.com/63804074/106175463-80b3da00-61d9-11eb-84a4-ad6a99abc775.png)

사진을 보면, ice라는 단어가 주어졌을 때 solid가 등장할 확률은 steam이 주어졌을 때 solid가 나타날 확률보다 높다. solid는 ice와 관련석이 높기때문이다. 그러면 P(solid |ice)/P(solid|steam)는 1보다 훨씬 큰값을 가진다.

P(gas |ice)/P(gas|steam)은 1보다 훨씬 작은 값(0.0085)이고. ice, steam과 관련성이 높거나 별로 없는 water와 fashion은 모두 1 안팎의 값이 나온다.

**이렇게 특정단어 k가 주어졌을 때 밀집벡터(임베딩)된 두 단어벡터의 내적이 두 단어의 동시등장확률 비율이 되도록 임베딩하였다.** solid가 주어지면 ice와 steam 벡터사이의 내적값이 8.9가 되도록 하자는 뜻이다. 

**목적함수**

<br>

![6](https://user-images.githubusercontent.com/63804074/106189359-8aded400-61eb-11eb-80fb-c96ba5bf8731.jpg)
![2](https://user-images.githubusercontent.com/63804074/106189176-46533880-61eb-11eb-80aa-5cf9438d9752.jpg)
![3](https://user-images.githubusercontent.com/63804074/106189178-46ebcf00-61eb-11eb-8ba0-fe18d11d38cb.jpg)
![7](https://user-images.githubusercontent.com/63804074/106253164-7b4aa400-625a-11eb-866d-1c8fc89d475d.jpg)
![8](https://user-images.githubusercontent.com/63804074/106253169-7c7bd100-625a-11eb-90bc-03fa966f8bd4.jpg)


**[심화내용 정리]**
Glove는 동시등장확률을 사용하여 말뭉치 전체의 통계정보를 손실함수에 도입하여 미니배치학습을하는 추론기반기법과 통계기반기법을 융합하였다.

**[출처]**
glove : https://ratsgo.github.io/from%20frequency%20to%20semantics/2017/04/09/glove/
<br>
LSA 설명 : https://ratsgo.github.io/from%20frequency%20to%20semantics/2017/04/06/pcasvdlsa/
