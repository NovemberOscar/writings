# DCGAN을 알아보자

## 1. GAN의 한계

GAN은 분명히 혁신적인 알고리즘 이었지만 한계 또한 존재했다. 
- 고해상도 이미지 생성 불가

- 학습 불안정
이와 같은 문제는 Minimax problem을 해결하는 네트워크 구조와 Fully connected network를 사용하는 GAN 자체의 구조에서 기인한 것이었다. 

오늘 알아볼 DCGAN은 FCN을 사용하는 네트워크 구조를 바꾸어 이와 같은 문제점을 해결했다. 

## 2. GAN과의 차이점

 사실 GAN과의 차이는 별로 없다. GAN의 Fully-connected network를 Deep convolution network로 대체했을 뿐이다. 

 하지만 이와 같은 차이점이 다음과 같은 결과를 이끌어 낼 수 있었다

- Scene을 이해하고 기존 GAN보다 고해상도의 이미지 생성

-  기존 GAN에 비해 안정된 학습 가능

- DCGAN의 특정 컨벌루션 필터가 특정 물체를 학습한다는 것 확인

- G에서 memorization이 일어난 것이 아니라는 것을 확인

- G의 입력으로 들어가는 벡터에 대한 산술 연산이 가능한 점을 가지고 sementic하게 출력 결과를 조절 가능

## 3. 모델 구조

DCGAN은 앞에서 보다시피 FCN 부분을 DCN으로 교체했을 뿐이라 loss 함수는 기존과 동일하다. 다만 단순히 FCN으로의 교체만 일어난 것이 아니라 연구팀이 수많은 실험을 통해 최적의 네트워크 구조를 만들었다. 

원 논문을 찾아보면 연구팀이 사실상 매뉴얼에 가깝게 모델 구조를 정리한 것을 볼 수 있을 것이다. 

[그림 1]

주요한 모델 구조는 다음과 같다. 

- D에서는 일반적인 CNN과 같이 Strided Convolution을 사용한다. 

- G에서는 업샘플링을 위해 Fractional Strided Convolution을 사용한다.(tensorflow의 tf.layers.conv2d_transpose)

- G와 D 모두에서 Batch Normalization을 적용하지만 불안정 및 발산 문제로 인해 D 입력 레이어와 G 출력 레이어에서는 사용하지 않는다. 

- G에서는 ReLU를 사용하고 마지막 레이어에서만 tanh를 사용

- D에서는 모두 Leaky ReLU를 사용한다.


> Fractional Strided Convolution이란?"""  
> 
> 간단하게 설명하자면 기존 합성곱 연산처럼 필터를 거치며 다운샘플링되는것이 아니라 연산을 거치면 업샘플링되어 사이즈가 늘어나는 합성곱 연산이다.  
>  텐서플로우에서는 tf.layers.conv2d_transpose라는 이름으로 구현되어 있으며 원 논문에서 언급한 deconvolution이란 단어는 잘못된 표현이다.
>
>[그림 2]  
> .

## 4. DCGAN의 결과



