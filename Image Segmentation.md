**Image Segmentation** 

입력  이미지에서  픽셀의  각  클래스를  구분하는  작업 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.001.jpeg)

1) **Idea** 

Object detection -> Bounding Box 

Image segmentation -> Classification of every pixel

입력  이미지  안의  모든  픽셀을  **지정된  개수**의  클래스로  분류 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.002.png)

Segmentation -> Semantic, Instance

Semantic:  각  pixel이  어떤  클래스인지  구분하는  문제 

Instance: (Advanced)  같은  클래스  안에서  서로  다른  객체까지  구분 

Image  Segmentation  Network는  각  pixel이  N개의  클래스  중  어디에  속하는지를  나타내는 Segmentation map을  출력함  (map은  N개의  채널로  구성됨) 

최종적으로  argmax를  통해  1  채널  이미지를  출력 

2) **기본  구조** 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.003.jpeg)

기본  구조: Encoder and Decoder

이미지의  사이즈를  줄이는  Encoder-Decoder 

**핵심  아이디어** 

1. 입력  이미지의  W, H를  줄이고  채널  수를  늘려  feature의  개수를  증가시킨다
1. W,  H를  입력  이미지의  사이즈로  회복,  채널  수는  클래스의  사이로  맞춰  Segmentation map을  생성 

**Down-sampling** 

인코딩  할  때, data의  개수를  줄이는  처리과정 

1. **Pooling**  

특정  규칙(Max, Average)에  의해  Kernel  내에서  값을  만들어  내거나  추출하는  방법 Pooling의  방법으로  Down-sampling  하는  것이  일반적임. 

2. **Dilated (Atrous) convolution (Trainable)** 

**Convolution**을  Down-sampling  기법으로  사용.  Pooling과는  다르게  **학습**을  거치므로  효과적으로 Down-sampling  할  수  있다.  그러나  **receptive field**를  크게  할  수  없음  (receptive field는  출력  레 이어의  뉴런  하나에  영향을  미치는  입력  뉴런들의  공간  크기이다.) 

Convolution은  receptive field를  크게  하기가  어렵다는  단점이  있음, trainable parameter가  무수히 늘어나기  때문에. -> Dilated convolution  고안됨 

Dilated convolution은  일반적인  convolution filter  사이에  빈  공간을  넣어  구멍이  뚫려  있는  듯한 구조를  가짐 

Semantic Segmentation에서  높은  성능을  내기  위해선  CNN의  마지막  feature map에  존재하는  한 pixel이  입력  값에서  어느  크기의  영역에서  커버하는  지를  결정하는  **receptive field**가  얼마나  큰 지가  중요함. 

따라서,  Dilated convolution을  통해  **receptive field를  넓히면서도  trainable parameter을  유지해 기존  convolution과  동일한  양의  parameter  개수와  계산  복잡도를  유지할  수  있음.** 

Hyperparameter:  확장비율(dilation rate, r) / dilation rate를  지정해  filter  사이에  빈  공간을  얼마나 둘  지를  결정함. 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.004.jpeg)

1. **Depthwise convolution**

Standard convolution ->  하나의  feature map으로  출력하기에  **1.  특정  채널만의  Spatial Feature을 추출할  수  없으며**, **2. Convolution layer가  깊어질수록  연산  복잡도가  증폭됨**. 

**-> Depthwise convolution**

각  채널마다  spatial feature을  추출하는  것이므로  각  채널  별로  filter가  존재한다. Input channel  수 와  output channel  수가  같아지게  됨  ![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.005.png)

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.006.png) ![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.007.png)

**Parameter와  연산량  비교** 

W: width, H: height, C: channel, K: kernel, M: output의  channel (K << M) output의  크기: H X W  이므로, # of parameters X H X W =  연산량 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.008.png)

2. **Depthwise separable convolution** 

: Depthwise convolution  뒤에  1 x 1 convolution을  연결한  구조 

**-> Spatial dimension  과  Channel dimension을  동시에  처리하던  것을  따로  분리시켜  처리** Parameter수  작음  

**연산량이  높은  곳에서는  최대한  Feature  Map을  적게  생성하고,  연산량이  낮은  곳에서  Feature Map의  숫자를  조절** 

\# of parameters: Depthwise convolution에  1 x 1 x C  크기의  Convolution을  M개  연결한  것 

= K x K x C + 1 x 1 x C x M +) 1 x 1 Convolution의  역할  

1) 공간적인  특성  X 
1) 연산량이  가장  적기에, Feature Map  개수를  조절할  때  사용됨 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.009.png)

**Up-Sampling(Deconvolution)** 

1. **Unpooling** 

**Maxpooling을  거꾸로  재현**  ->  동일한  값으로  채우거나  (Nearest Neighbor Unpooling), 0으로  채 워주는  방식  (Bed of NailsUnpooling) 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.010.png) ![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.011.png)

2. **Max Unpooling** 

Unpooling의  문제: Unpooling을  했을  때,  원래  **Max Pooled된  값의  위치를  알  수  없음** 

->  Max Unpooling  (max pooling  된  위치를  기억,  그  위치에  값을  복원):  정보  손실을  방지할  수 있음 

3. **Transpose Convolution** 

행렬을  이용한  방법 

Padding = 0, stride = 1  로  3x3 kernel을  4x4 input에 

-> 3x3 kernel을  4x16  행렬로, input -> 16x1, 2x2 output -> 4x1로  분해해서  계산 

4x16  행렬을  Transpose해서  input feature map을  output으로  출력함 

결과적으로  pooling layer로  축소된  이미지가  원본  이미지의  사이즈와  같게  복원이  됨. 

![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.012.jpeg) ![](Aspose.Words.830bdcb0-731d-4f42-92db-95e405487395.013.png)

**FCN** 

FCN: CNN  기반  우수한  모델들을  Semantic Segmentation Task에  맞게  변형시킨  모델 

기존  이미지  분류에  쓰인  네트워크(pretrained), Feature Extraction layer는  그대로  활용해  Feature를 추출, FC layer  대신에  1x1 Conv + Up-sampling으로  변경. (Fine tuning) -> input size와  같아짐 

1. Convolution layer을  통해  feature  추출 
1. 1x1 Convolution layer을  이용해  feature map의  채널  수를  객체의  수와  같게  변경 
1. Up-sampling 
1. 최종  feature map과  label feature map의  차이를  이용해  네트워크  학습 

문제점:  feature  map을  입력  이미지의  위치  정보를  대략적으로만  갖고  있음  ->  이것을  up- sampling  했을  때  기존  이미지보다  흐려짐. 

➔  **Skip architecture** 

피처  추출  단계의  피처맵도  업샘플링에  포함(Sum)하여  위치  정보  손실을  막자라는  것이  핵심 

FC layer를  삭제 

1. FC layer를  통과하고  나면  이미지의  위치  정보가  사라지게  됨 
1. FC layer는  고정된  크기의  input image만  받을  수  있음 
1. FC layer는  parameter  개수가  너무  많음 
