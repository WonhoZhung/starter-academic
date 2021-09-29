---
title: Pixel Recurrent Neural Networks
date: 2021-09-29T03:21:10.609Z
summary: This post is based on a paper "Recurrent Pixel Neural Networks" by Oord
  et al. at ICML 2016.
draft: false
featured: false
tags:
  - Paper Summary
  - AI602
  - Autoregressive
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
## Paper Summary: [Pixel Recurrent Neural Networks](http://proceedings.mlr.press/v48/oord16.pdf)

### 1. Intro

 생성 모델은 비지도학습(**unsupervised learning**)으로, 주어진 데이터의 분포(**probabilistic density**)를 학습하여 그 분포에 포함되는 새로운 데이터를 샘플링하는 방법이다. 특히 이 방법은 이미지 데이터의 inpainting, deblurring과 같은 복원 과정에 사용될 수 있다. 하지만 이미지 데이터의 분포는 굉장히 복잡하여, 이 분포를 잘 표현하면서 **tractable**하고 **scalable**한 모델을 만드는 것은 굉장히 어렵다. 
    
 Tractable이라 함은 학습한 데이터의 분포를 직접적으로 알 수 있음을 이야기하는데, 이에 대한 한 가지 방법으로 **autoregressive** 모델을 사용할 수 있다. 이는 예측할 **joint distribution**을 factorizing하여 각 **conditional distribution**을 예측하게 하는 것이다. 
    
 $$\tag{1}p(\mathbf{x})=\prod_{i=1}^{n^2}p(x_i|x_1,...,x_{i-1})$$
    
 식 (1)에서 $n\times n$ 픽셀의 이미지에 대한 joint distribution을 $p(\mathbf{x})=p(x_1,...,x_{n^2})$라 할 때, $p(x_i|x_1,...,x_{i-1})$는 첫 번째 픽셀부터 $i-1$ 번째 픽셀에 대한 정보가 주어졌을 때 $i$ 번째 픽셀에 대한 conditional probability이다. 이러한 방법을 통해 아래의 그림처럼 왼쪽 위에서부터 오른쪽 아래의 픽셀까지의 각각의 확률 분포를 순차적으로 예측할 수 있다. 
    
 {{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%200.png?raw=true" title="Fig 1. Illustration of equation (1)">}}  
    
 또한, 각 픽셀은 Red, Green, Blue의 세 개의 채널 값을 가지고 있는데 이 값들 또한 factorizing하여 아래와 같이 표현할 수 있다.
           
$$\tag{2}p(x_i|\mathbf{x}_{<i})=p(x_{i,R}|\mathbf{x}_{<i})p(x_{i,G}|\mathbf{x}_{<i},x_{i,R})p(x_{i,B}|\mathbf{x}_{<i},x_{i,R},x_{i,G})$$
    
 이러한 conditional distribution을 예측할 때, 멀리 있는 픽셀들 사이의 long-range correlation과 같은 복잡한 분포를 표현할 수 있어야 하므로 적절한 architecture가 필요하다. 본 논문에서는 recurrent neural networks (RNN)을 사용하였는데, 특히 two-dimensional long short-term memory (**LSTM**) layer를 사용하여 모델을 구현하였다. 추가적으로, 깊게 LSTM layer를 쌓기 위해 residual connection을 사용하였다.
    
 또한, 연속적인 분포를 사용한 기존 연구들과 다르게 각 conditional distribution을 $[0, 255]$의 256개의 값에 대한 불연속적인 분포로 나타내었다. 생성 과정 동안 softmax를 거쳐 만들어진 multinomial distribution에서 sampling하여 순차적인 픽셀 생성이 이루어진다.
    

---                       

### 2. Main Contributions

본 논문에서는 네 가지의 모델을 소개하고 있다.
    
 1. *PixelRNN* with Row LSTM layer
 2. *PixelRNN* with Diagonal BiLSTM layer
 3. *PixelCNN*
 4. Multi-scale *PixelRNN*
    
 한 가지씩 살펴보고, 각 모델에 대한 결과를 알아보자.
    
1. *PixelRNN*
  
   a. LSTM
        
      LSTM은 기본적으로 input-to-state와 state-to-state 파트로 이루어져 있다. 기존의 two-dimensional LSTM은 식 (3)과 같이 hidden-state $h_{i,j}$가 업데이트 되며, 픽셀들이 서로 inter-dependent하므로 픽셀 단위로 학습해야한다. 따라서 병렬화가 안되며 속도가 느리다는 단점이 있다.
        
      $$\tag{3} h_{i,j}=LSTM(h_{i-1,j},h_{i,j-1},x_{i,j})$$
        
    b. Row LSTM
        
      Row LSTM은 위의 단점을 보완하기 위한 한 가지 방법으로, row 단위로 $k\times 1$의 one-dimensional convolution 계산을 수행한다. 아래 식 (4)는 kernel size $k=3$ 일 때의 hidden-state update를 나타낸다. 이 방법은 픽셀 단위의 LSTM 보다는 학습 속도가 빠르지만, Fig 2.의 왼쪽 그림과 같이 triangular receptive field로 인하여 누락되는 정보가 생긴다.
        
      $$\tag{4} h_{i,j}=LSTM(h_{i-1,j-1},h_{i-1,j},h_{i-1,j+1},x_{i,j})$$
        
    c. Diagonal BiLSTM
        
      주어진 모든 이전 픽셀 정보를 활용하여 다음 픽셀을 예측하기 위해 제안된 방법이 Diagonal BiLSTM이다. 이를 수행하기 위해서 Fig 3.과 같이 기존 이미지의 각 row를 윗 row에 비해서 한 칸씩 오른쪽으로 skewing하는 pre-processing을 한다. 이후 식 (5)와 같이 column-wise $2\times 1$ convolution을 통해 다음 hidden-state를 구하게 된다. 이 방법을 통해서 모든 context를 활용하며 병렬화된 계산을 수행할 수 있다.
        
      $$\tag{5}h_{i,j}=LSTM(h_{i-1,j-1},h_{i,j-1},x_{i,j})$$
        
      {{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%201.png?raw=true" title="Fig 2. Illustration of the Row LSTM and the Diagonal BiLSTM">}} 
        
      {{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%202.png?raw=true" title="Fig 3. Skewing operation of the input map for the Diagonal BiLSTM">}} 
        
2. *PixelCNN*
    
    RNN에 비해서 computational cost를 낮추기 위한 방법으로 masked CNN을 사용하였다. Fig 4.와 같이 아직 예측하지 않은 pixel을 masking한 kernel을 이용하여 convolution을 수행하였다. 이렇게 하면 recurrent한 계산을 하지 않고 모든 픽셀에 대한 값을 한 번에 계산할 수 있어 속도가 빠르지만 bounded receptive field로 인하여 long-range correlation은 고려하지 못한다(blind spot). 
    
    {{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%203.png?raw=true" title="Fig 4. Illustration of the PixelCNN (Source: [http://slazebni.cs.illinois.edu/spring17/lec13_advanced.pdf](http://slazebni.cs.illinois.edu/spring17/lec13_advanced.pdf))">}} 
    
3. Multi-scale *PixelRNN*
    
    먼저 기존 이미지의 subsampled 이미지를 PixelRNN으로 생성한 후, 이로부터 deconvolution layer를 통해 다시 원래 크기의 이미지를 생성하는 방식이다. 
    
    {{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%204.png?raw=true" title="Fig 5. Illustration of multi-scale case">}}
    
<br>

이제 결과를 살펴보자. 먼저 눈여겨볼 부분은 역시 각 픽셀 RGB 채널의 tractable한 probability distribution이다. Fig 6.과 같이, 모델에 불연속적인 분포를 사용하였음에도 well-behaving하는 확률 분포를 보여주었다.

{{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%205.png?raw=true" title="Fig 6. Example distributions from the model">}}

본 논문에서는 MNIST, CIFAR-10 데이터셋에 대해 모델 성능을 평가하였다. 기존의 모델들의 성능보다 state-of-the-art를 보여주었다. 특히, Diagonal BiLSTM의 넓은 receptive field 덕분에 이 경우 가장 좋은 성능을 보여주었다.

{{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%206.png?raw=true" title="Table 1. NLL test performance on MNIST">}}

{{< figure src="https://github.com/WonhoZhung/starter-academic/blob/master/images/post4/Untitled%207.png?raw=true" title="Table 2. NLL test performance on CIFAR-10">}}

### 3. Opinion

본 논문은 autoregressive model의 정석으로, sequential하게 픽셀의 conditional distribution을 구하는 방식으로 이미지를 생성하는 방법론을 소개하였다. 특히, Diagonal BiLSTM을 사용하여 PixelRNN에서 parallelization을 가능하게 하는 동시에 이미지 크기에 상관없이 모든 context를 포함하게 한 아이디어가 인상적이었다. 아래는 논문을 읽으며 생각한 discussion point들이다.
    
- How can the autoregressive model control the randomness of the sampling?
- What if we train/generate the model in other sequential paths of image pixel rather than from top-left to bottom-right?
- Can we use attention to interpret the long-range correlations between pixels?

$$***$$