#### 前言

在项目的开发过程中，对于资源的使用，无非有以下几种情况：

> 1、Application使用自身资源
>  2、Application使用Module资源
>  3、Module使用自身或另外的Module资源
>  4、Module使用Application资源

前3项在这里就不过多介绍，这里主要说下第4项，有些时候我们需要在抽取的module中使用到Application的资源，而且我们并不想在Module中重新定义。如果直接调用的话，是调用不到的。所以我们需要通过getIdentifier()方法来实现，具体如下：



```java
  /**
     * 获取context中对应类型的资源id
     *
     * @param context
     * @param type 资源类型，"layout","string","drawable","color"等
     * @param resName
     * @return
     */
    private static int getResId(Context context, String type, String resName) {
        return context.getResources().getIdentifier(resName, type, context.getPackageName());
    }
```

举个栗子🌰，在下图中，包含**2个Application(app/doctor)**和**5个Module(calendarview/commlib/date_picker/MPChartLib/NativeH5Lib)**，如果要在commlib使用app的资源，就使用以上的方法去获取。

![img](.asserts/webp)



#### 调用方式

调用方法非常简单，只需传入context和res名称即可，如下所示：



```java
ResourceUtil.getColorResId(this, "colorPrimary"); //colorPrimary是color.xml文件中定义的资源
```

#### 完整的工具类



```java
import android.content.Context;

/**
 * Created by chenyk on 2017/6/2.
 * 资源工具类
 * 功能：module中根据context动态获取application中对应资源文件
 */

public class ResourceUtil {
    /**
     * 获取布局资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getLayoutResId(Context context, String resName) {
        return getResId(context, "layout", resName);
    }

    /**
     * 获取字符串资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getStringResId(Context context, String resName) {
        return getResId(context, "string", resName);
    }

    /**
     * 获取Drawable资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getDrawableResId(Context context, String resName) {
        return getResId(context, "drawable", resName);
    }

    /**
     * 获取颜色资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getColorResId(Context context, String resName) {
        return getResId(context, "color", resName);
    }

    /**
     * 获取id文件中资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getIdRes(Context context, String resName) {
        return getResId(context, "id", resName);
    }

    /**
     * 获取数组资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getArrayResId(Context context, String resName) {
        return getResId(context, "array", resName);
    }

    /**
     * 获取style中资源
     *
     * @param context
     * @param resName
     * @return
     */
    public static int getStyleResId(Context context, String resName) {
        return getResId(context, "style", resName);
    }

    /**
     * 获取context中对应类型的资源id
     *
     * @param context
     * @param type
     * @param resName
     * @return
     */
    private static int getResId(Context context, String type, String resName) {
        return context.getResources().getIdentifier(resName, type, context.getPackageName());
    }
```



作者：chenyk
链接：https://www.jianshu.com/p/282b0d1c142c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。