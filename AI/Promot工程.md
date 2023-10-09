## 一、环境搭建

以ChatGpt为例：

```python
# 加载环境变量
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())  # 读取本地 .env 文件，里面定义了 OPENAI_API_KEY

openai.api_key = os.getenv('OPENAI_API_KEY')

```

### 查看可调用模型

```python
import openai
import os

# 加载 .env 文件
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())

# 从环境变量中获得你的 OpenAI Key
openai.api_key  = os.getenv('OPENAI_API_KEY')

# 模型列表
models = openai.Model.list()

for model in models.data:
    print(model.id)
```

### 定义角色发消息

```python
# 消息格式
messages=[
    { 
        "role": "system", 
        "content": "你是AI助教，提醒学生每周一到周五白天上课"
    },
    { 
        "role": "user", 
        "content": "你是干什么的?什么时间上课"
    },
    
]

# 调用ChatGPT-3.5
chat_completion = openai.ChatCompletion.create(model="gpt-3.5-turbo", messages=messages)

# 输出回复
print(chat_completion.choices[0].message.content)
```

### 定义基于prompt生成文本的函数

```python
# 基于 prompt 生成文本
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0,  # 模型输出的随机性，0 表示随机性最小
    )
    return response.choices[0].message["content"]
```





## 五、Trick（常用咒语）

```
1. I want you to become my Expert Prompt Creator. Your goal is to help me craft the best possible prompt for my needs. The prompt you provide should be written from the perspective of me making the request to ChatGPT. Consider in your prompt creation that this prompt will be entered into an interface for ChatGpT. The process is as follows:1. You will generate the following sections:

Prompt: {provide the best possible prompt according to my request)

Critique: {provide a concise paragraph on how to improve the prompt. Be very critical in your response}

Questions:
{ask any questions pertaining to what additional information is needed from me toimprove the prompt  (max of 3). lf the prompt needs more clarification or details incertain areas, ask questions to get more information to include in the prompt}

2. I will provide my answers to your response which you will then incorporate into your next response using the same format. We will continue this iterative process with me providing additional information to you and you updating the prompt until the prompt is perfected.Remember, the prompt we are creating should be written from the perspective of me making a request to ChatGPT. Think carefully and use your imagination to create an amazing prompt for me.
You're first response should only be a greeting to the user and to ask what the prompt should be about
```

这段咒语可以让chatGpt帮你写prompt，