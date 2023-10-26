## 插件简介和安装

ebsynth_utility for SD WebUI : [https://github.com/s9roll7/ebsynth_ut...](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa0Z5TG1VdUstWGFmY0FaSm5VWEZleGs0d19id3xBQ3Jtc0tuVlN0d1dlR2U0YUl3ZUREaWVNWmJXNkc2OGVpUktWeTNjemQ3TTVHQlVBYzBWekdWd0NaUzIzUTZRVjN2WmhUd2l0NVRYYkt3Zm1JLVlUSk5feGFOVHdpb1ZsVHNtREFFRkk5b3dmYUw4eHlhNWZuSQ&q=https%3A%2F%2Fgithub.com%2Fs9roll7%2Febsynth_utility&v=AN2Qf7Gek4g) 

Ebsynth (For video frames generation) : [https://ebsynth.com/](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa01QNUZTTlA5MVV0LTlFbHVXbVNqS1YwRHdaZ3xBQ3Jtc0tsMGVGUEdLQVVuT2Z6Z2tEVExDV3FDNnloeVJXblZZMTBocWVhU1NOQnhnX3JTSVdmdndmYXlIMkJZal93Skd5elVTZGQwMjU2NXVyRXRYbmRCWmRNcTVNNTZhZ1A2MGlmOWQtWjFFaDlLQ1RINk5PWQ&q=https%3A%2F%2Febsynth.com%2F&v=AN2Qf7Gek4g) 

All Google Colab links here : [https://thefuturethinker.org/stable-d...](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbTB6RGptVTFoc3lzX2NrYXVMWWRCZG1WaWZpd3xBQ3Jtc0trNmh2YzZFOTA2N1QxbVNaTjFuekVlaVRubWZ3UEFUbGwzMWlDREk1ZXFTN2d3MnpQM2tLM3hJZWdodFdONG8yVVVaRk5MZ1JqVEV0VFdtNmtaemltdHhFbmo2Q0N5RjF3R1NoMGVXcUZUY0dzS3dFdw&q=https%3A%2F%2Fthefuturethinker.org%2Fstable-diffusion-google-colab-ipynb-list%2F&v=AN2Qf7Gek4g) 

直接在SD界面上插件安装那从https://github.com/s9roll7/ebsynth_utility 这个地址安装，然后重启webUI。



插件步骤提示如下：

The process of creating a video can be divided into the following stages.
(Stage 3, 4, and 6 only show a guide and do nothing actual processing.)

**stage 1**
Extract frames from the original video.
Generate a mask image.

**stage 2**
Select keyframes to be given to ebsynth.

**stage 3**
img2img keyframes.

**stage 3.5**
(this is optional. Perform color correction on the img2img results and expect flickering to decrease. Or, you can simply change the color tone from the generated result.)

**stage 4**
and upscale to the size of the original video.

**stage 5**
Rename keyframes.
Generate .ebs file.(ebsynth project file)

**stage 6**
Running ebsynth.(on your self)
Open the generated .ebs under project directory and press [Run All] button.
If out-* directory already exists in the Project directory, delete it manually before executing.
If multiple .ebs files are generated, run them all.

**stage 7**
Concatenate each frame while crossfading.
Composite audio files extracted from the original video onto the concatenated video.

**stage 8**
This is an extra stage.
You can put any image or images or video you like in the background.
You can specify in this field -> [Ebsynth Utility]->[configuration]->[stage 8]->[Background source]
If you have already created a background video in Invert Mask Mode([Ebsynth Utility]->[configuration]->[etc]->[Mask Mode]),
You can specify "path_to_project_dir/inv/crossfade_tmp".



## 使用步骤



### Step1 提取视频帧

可以生成出人物蒙版



### Step2 提取关键帧



