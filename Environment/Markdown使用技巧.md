#### 1. markdown中插入图片怎么定义图片的大小或比例

**方法一：嵌入HTML代码**
使用img标签

```html
 <img src="./xxx.png" width = "300" height = "200" alt="图片名称" align=center />
```

**附：**如果需要居中的话只要在外面包围div标签即可

```html
<div  align="center">    
...
</div>
```



**方法二：Typora**

如果你用Typora，可以用<img> 标签更改，支持设定宽度、高度。

```markdown
<img src="https://www.google.com/doodles/kamma-rahbeks-241st-birthday" width="200px" />
<img src="https://www.google.com/doodles/kamma-rahbeks-241st-birthday" style="height:200px" />
```

也可以指定缩小比例：

```text
<img src="https://www.google.com/doodles/kamma-rahbeks-241st-birthday" style="zoom:50%" />
```

详细可以看官方说明：[Resize Images](https://link.zhihu.com/?target=http%3A//support.typora.io/Resize-Image/)

