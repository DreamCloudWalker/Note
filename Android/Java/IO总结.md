### 装饰模式
* DataInput(Output)Stream就是典型的装饰模式，嵌套关系如：
    `DataOutputStream dos = new DataOutputStream(new BufferedOutoutStream(new FileOutputStream(new File(file))));`

### 字节流
* ByteArrayInput(Output)Stream
* PipedInput(Output)Stream
* FilterInput(Output)Stream <- BufferedInput(Output)Stream/DataInput(Output)Stream/PrintStream
* FileInput(Output)Stream
* ObjectInput(Output)Stream


### 字符流
* `BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File("/mnt/sdcard/test.txt")), "GBK"))`
* CharArrayReader(Writer)
* PipedReader(Writer)
* FilterReader(Writer)
* BufferedReader(Writer)
* OutputStreamReader(Writer) <- FileReader(Writer)
* PrintReader(Writer)

### 字符流/字节流
* FileOutputStream： 字节流
* OutputStreamWriter: 字符流
* BufferedWriter Buffer是一个缓冲区，为什么要用BUFFER?如果你直接用stream 或者writer，你的硬盘可能就是每读一个字符或一个字节就去读取硬盘一次，IO负担巨大。用Buffer会一次读一堆数据；


### 非流式部分
* File: 打哪指哪，只能从文件头开始读；
* RandomAccessFile： 指哪打哪，可从任意地方开始读；
* nio.FileChannel，读取速度更快