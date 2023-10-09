## 一、地址

https://github.com/Linaqruf/kohya-trainer

选择如下图所示的LoRA Training (Dreambooth method)

![image-20230714105941045](.asserts/image-20230714105941045.png)



## 二、训练

### 1. Install Kohya Trainer

1.1勾选mount drive后直接运行，如有报错可再跑一次；(这一步Colab耗时4分钟左右)

1.2跳过



### 2. Pretrained Model Selection 预训练模型

2.1 Download Available Model，选择Stable Diffusion-v1-5运行；

2.2 跳过（如果不训练2.1，推荐用https://civitai.com/api/download/models/11745 这个ChilloutMix模型或https://civitai.com/api/download/models/177164 这个beautifulRealistic_v7.safetensors来训练，出人物图效果更好）

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

5.4 训练参数配置，用默认值运行；

5.5 开始训练，直接运行，30多张图估计要20多分钟；



## 三、模型使用

1. 下载刚才训练的YourModelName.safetensors，放到Stable Diffusion目录/stable-diffusion-webui/models/Lora下。

2. 打开Stable Diffusion后，选择Stable Diffusion checkpoint v1-5-pruned.safetensors作为主模型。点击如下红色按钮(show extra networks)：

![image-20230714112555523](.asserts/image-20230714112555523.png)

3. 下面选择Lora，点击刚才拷贝加入的模型；
4. 输入提示词，比如a girl，就会生成对应模型训练的图片了。



P.S. Stable Diffusion WebUI可在地址末尾加上/?__theme=dark，开启黑夜模式。

