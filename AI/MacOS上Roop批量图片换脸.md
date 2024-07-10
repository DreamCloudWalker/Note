git clone了roop项目后，安装依赖

pip3.10 install -r requirements.txt



如果运行中出现类似：Frame processor face_enhancer not found. 这样的错误，重新安装一下依赖。



使用如下脚本批处理：

```python
# 批处理Roop
import subprocess
import time
import os

# 指定要遍历的目录路径
source = '/Users/jian.deng/AI/Roop/Faces/zr/zr.jpg'
directory = '/Users/jian.deng/AI/Lora/DressSliverMoxiong'
result_dir = os.path.join(directory, 'result')
if not os.path.exists(result_dir):
    os.makedirs(result_dir)
    print(f"The directory {result_dir} has been created.")
else:
    print(f"The directory {result_dir} already exists.")

# 使用os.listdir()函数列出目录下的所有文件和子目录
for filename in os.listdir(directory):
    # 拼接文件的完整路径
    file_path = os.path.join(directory, filename)

    # 检查文件是否是一个文件而不是目录，并且文件名以".png"结尾
    if os.path.isfile(file_path) :
        if filename.lower().endswith('.jpg') or filename.lower().endswith('.png') or filename.lower().endswith('.jpeg'):
            file_result = os.path.join(directory, 'result', filename)
            cmd = f"python3.10 run.py --execution-provider cpu -s {source} -t {file_path} -o {file_result} --frame-processor face_swapper face_enhancer --similar-face-distance 1.5 --reference-face-position 1 --output-video-encoder libx264 --output-video-quality 35  --keep-fps --many-faces --temp-frame-format jpg --temp-frame-quality 0"
            print("cmd:"+cmd)
            # 使用subprocess模块执行另一个Python脚本
            try:
                subprocess.run(cmd, shell=True)
            except Exception as e:
                print(f"执行脚本时出错：{e}")

print("批处理Roop执行结束")
```

