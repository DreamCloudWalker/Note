### GLFW库简介

GLFW是配合OpenGL使用的轻量级工具库，全称 Graphics Library Framework（图形库框架），官网：

[https://www.glfw.org/download.htmlwww.glfw.org/download.html](https://link.zhihu.com/?target=https%3A//www.glfw.org/download.html)



### **GLAD库配置**

因为OpenGL只是一个标准/规范，具体的实现是由**驱动开发商**针对**特定显卡**实现的（例如不同型号的显卡可能就会对应一个不同版本的OpenGL库）。

由于OpenGL驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时**查询**。

所以任务就落在了开发者身上，开发者需要在运行时**获取**函数地址并将其保存在一个函数指针中供以后使用。取得地址的方法因平台而异，在Windows上会是类似这样：

![image-20240318163134696](.asserts/image-20240318163134696.png)

我们可以看到上面代码非常复杂，步骤很繁琐不说，而且我们需要对每个可能使用的OPENGL函数都要重复这个获取指针的过程。幸运的是，有些库能简化此过程，其中GLAD是目前最新，也是最流行的库。有了GLAD我们就只需要一句代码就可以了（当然要先初始化GLAD）。glGenBuffers(1, &buffer);

GLAD是一个开源的库，它能解决我们上面提到的那个繁琐的问题。GLAD的配置与大多数的开源库有些许的不同，GLAD使用了一个在线服务（网址：[http://glad.dav1d.de](https://link.zhihu.com/?target=http%3A//glad.dav1d.de)）。在这个网页里我们选中需要配置的OpenGL版本，然后就会自动生成对应这个版本的OpenGL函数库。配置方法如下：

打开GLAD的网址，将语言(Language)设置为C/C++，在API选项中，选择3.3以上的OpenGL(gl)版本（我们的教程中将使用3.3版本，但更新的版本也能用）。之后将模式(Profile)设置为Core，并且保证选中了生成加载器(Generate a loader)选项。现在可以先（暂时）忽略扩展(Extensions)中的内容。都选择完之后，点击生成(Generate)按钮来生成库文件。

GLAD现在应该提供给你了一个zip压缩文件，包含两个头文件目录，和一个glad.c文件。将两个头文件目录（glad和KHR）复制到你的Include文件夹中（或者增加一个额外的项目指向这些目录），并添加glad.c文件到你的工程中。