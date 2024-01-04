## 一、地址

https://github.com/Linaqruf/kohya-trainer

选择如下图所示的LoRA Training (Dreambooth method)

![image-20230714105941045](.asserts/image-20230714105941045.png)

教程参考：

https://home.gamer.com.tw/artwork.php?sn=5687897



数据准备：

準備一個想學習的資料集，圖片**不需要固定解析度**（enable_bucket預設為True）。

訓練角色時**不需要服裝一致**，除非你想要把服裝當成角色概念的一部分。

背景不需要去背，維持多元背景即可。

至於多角色圖，個人還是習慣裁掉以免影響訓練。



## 二、训练

### 1. Install Kohya Trainer

1.1勾选mount drive后直接运行，如有报错可再跑一次；(这一步Colab耗时4分钟左右)

1.2跳过



### 2. Pretrained Model Selection 预训练模型

2.1 Download Available Model，选择Stable Diffusion-v1-5运行；

2.2 跳过（如果不训练2.1，推荐用https://civitai.com/api/download/models/11745 这个ChilloutMix模型或https://civitai.com/api/download/models/177164 这个beautifulRealistic_v7.safetensors来训练，出人物图效果更好）

2.2 跳过（如果不训练2.1，推荐用https://civitai.com/api/download/models/11745 这个ChilloutMix模型来训练，出图效果更好），不过现在我一般用https://civitai-delivery-worker-prod-2023-09-01.5ac0637cfd0766c97916cefa3764fbdf.r2.cloudflarestorage.com/275927/model/bra6test.bhwb.safetensors?X-Amz-Expires=86400&response-content-disposition=attachment%3B%20filename%3D%22beautifulRealistic_v60.safetensors%22&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=2fea663d76bd24a496545da373d610fc/20230904/us-east-1/s3/aws4_request&X-Amz-Date=20230904T144623Z&X-Amz-SignedHeaders=host&X-Amz-Signature=01eb68ff67815888a6d12a1dcac862d959f6c1a1d6070124f3a3eb64348fc0cf 这个Beautiful Realistic Asians这个CheckPoint。



如果是训练插画风，可以用 https://civitai.com/api/download/models/93208?type=Model&format=SafeTensor&size=pruned&fp=fp16  Dark Sushi Mix 大颗寿司Mix 这个CheckPoint试试。



P.S.

今天给大家推荐 5 个超实用 Stable Diffusion 大模型，提高AI绘画效率的必备神器！

* 1、EpiCRealism：下载量高达27万的冠军级真人模型，与其他真人模型相比，它的人物形象不容易被认为是生成的AI脸，非常适合生成逼真的照片。https://civitai.com/api/download/models/143906?type=Model&format=SafeTensor&size=pruned&fp=fp16

![image-20231225104117445](.asserts/image-20231225104117445.png)

* 2、DarkSushi：模型非常适合绘制复杂场景的二次元插画，并能够呈现出正常亮度和出色的光影效果。  https://civitai.com/api/download/models/93208?type=Model&format=SafeTensor&size=pruned&fp=fp16

![image-20231225104034594](.asserts/image-20231225104034594.png)

* 3、RongHua：荣华国风大模型，适用于生成国风的真人摄影照片，以后用自己的照片做图生图是不是不用跑影楼了 https://civitai.com/api/download/models/253479?type=Model&format=SafeTensor&size=pruned&fp=bf16

![image-20231225103804261](.asserts/image-20231225103804261.png)

* 4、SDVN7-NijiStyleXL：二次元风格模型，色彩比较丰富，生出的图很有特色。 https://civitai.com/api/download/models/155870?type=Model&format=SafeTensor&size=full&fp=fp16

![image-20231225103850228](.asserts/image-20231225103850228.png)

* 5、DynaVision XL：专为绘制3D卡通IP而设计的模型，它能够制作出精美细腻、色彩丰富的3D卡通角色。对于那些希望绘制3D卡通IP的小伙伴来说，这款模型是必备的工具。https://civitai.com/api/download/models/198962?type=Model&format=SafeTensor&size=pruned&fp=fp16

![image-20231225103938325](.asserts/image-20231225103938325.png)



2.3 Download Available VAE (Optional)，选择stable diffusion vae.pt运行；



### 3. Data Acquisition 上传数据

3.1 用默认目录直接运行；

3.2 把Google Drive打包好的图片.zip的路径拷贝到zipfile_url再运行；

3.3 跳过



### 4. Data Cleaning

4.1 直接运行；(可以不跑，我碰到过把我正常图片都删了的情况)

4.2.1 直接运行；（生成提示词）（这一步耗时2m）

4.2.2 如果是人物图片的训练，可以把人物的**character_threshold**这个调高，比如调到0.8再运行；（生成关键字提示词）

4.2.3 跳过；



### 5. 训练

5.1 配置，<font color="red">project_name给出自己要训练的模型的名字</font>，**pretrained_model_name_or_path**这个要把工作目录下的/content/pretrained_model/Stable-Diffusion-v1-5.safetensor这个路径拷贝过来。vae这个要拷贝工作目录/content/vae/stablediffusion.vae.pt。output dir可以用默认的，下面可以勾选保存到drive。运行；

如果是重新训练新数据，记得把工作目录的train_data文件改名。

5.2 数据集配置，默认运行；

5.3 LoRA配置，默认运行；

5.4 训练参数配置，用默认值运行；也可以在这里修改训练参数。常改参数分别代表如下：

* **num_epochs**
* **save_n_epochs_type_value** 代表每几个epochs存一次檔

5.5 开始训练，直接运行，30多张图估计要20多分钟；



## 三、模型使用

1. 下载刚才训练的YourModelName.safetensors，放到Stable Diffusion目录/stable-diffusion-webui/models/Lora下。

2. 打开Stable Diffusion后，选择Stable Diffusion checkpoint v1-5-pruned.safetensors作为主模型。点击如下红色按钮(show extra networks)：

![image-20230714112555523](.asserts/image-20230714112555523.png)

3. 下面选择Lora，点击刚才拷贝加入的模型；
4. 输入提示词，比如a girl，就会生成对应模型训练的图片了。



P.S. Stable Diffusion WebUI可在地址末尾加上/?__theme=dark，开启黑夜模式。

