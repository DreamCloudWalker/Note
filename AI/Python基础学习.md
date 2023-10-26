## 编码

### 忽略错误

```python
text = 'hello,你好'
# ASCII 编码将字符串转换为字符码，无法处理则忽略错误(不加会抛出UnicodeEncodeError之类的异常)
text1 = text.encode('ascii', errors='ignore')
print(text1)
# ASCII 编码将字符串转换为字符码，无法编码的用问号代替
text2 = text.encode('ascii', errors='replace')
print(text2)
```

系统默认把文件当文本文件，如果要打开二进制文件，需要添加encoding=None，如下：

```python
import codecs

with codecs.open('./bin_data.bin', mode='rb', encoding=None) as file:
  binary_data = file.read()
  
print(binary_data)
```



