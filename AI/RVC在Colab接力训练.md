## 核心代码：

```python
#@title  Model saving/pretrained model preferences
ModelName = "Lu"  #@param {type:"string"}
%cd /content/Retrieval-based-Voice-Conversion-WebUI

#@markdown **Save the trained model files to Google Drive instead of the colab runtime filesystem. You also need to check and execute this when resuming training. If you are short on Drive space you may want to uncheck this (in which case you must use the file menu to manually download the model checkpoints in so-vits-svc/logs/ yourself and copy the latest checkpoint back into so-vits-svc/logs/ to resume training).**
Save_to_drive = True #@param {type:"boolean"}
if Save_to_drive:
  # !rm -rf /content/Retrieval-based-Voice-Conversion-WebUI/logs/"{ModelName}"
  !mkdir -p /content/drive/MyDrive/RVC/models/"{ModelName}"
  !mkdir -p /content/drive/MyDrive/RVC/weights/"{ModelName}"
  !ln -s /content/drive/MyDrive/RVC/models/"{ModelName}" /content/Retrieval-based-Voice-Conversion-WebUI/logs/"{ModelName}"
  !ln -s /content/drive/MyDrive/RVC/weights/"{ModelName}" /content/Retrieval-based-Voice-Conversion-WebUI/weights
```



注意共享的时候，需要各个账号都互相共享到。比如A训练了一部分，共享给B，B继续训练，共享给C。C训练时，如果没有A给C共享权限，C可能拿不到必要的数据而训练失败。



## 注意事项

* 拷贝dataset路径的时候，路径要精确到模型名字，比如/content/dataset/dlrbNew
* 如果出现接力训练一开始就被手动结束，又没有任何异常，可以把比如/content/Retrieval-based-Voice-Conversion-WebUI/logs/dlrbNew/0_gt_wavs这个目录删了之后再开始训练。还能续上。
* Weights下的文件经常无法映射正常，最好这个生成后就马上下载下来备份一下。
