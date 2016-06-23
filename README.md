# SpannableString

一个人需要隐藏多少秘密才能巧妙地度过一生       — 仓央嘉措

##前言

上次看到一款学习的 `App`，有这样一个功能，在一个 `TextView` 中有一段英文，点击英文单词通过语音朗读出来。语音先不考虑，怎么去实现 `TextView` 点击获取每个单词的内容的呢？

肯定是用`SpannableString`去实现的呗，不然你今天讲它干嘛。嘿嘿，我说的对不对？

答案是肯定的，由于惯性我们先来看看效果图：

![sp](http://img.blog.csdn.net/20160620195358057)

我这里没有获取每个单词，而是获取的每段句子。效果图上也展示了一些其他的`SpannableString`效果，大多数可能你已经见过了。那么你肯定会问`SpannableString`是什么，`SpannableString`是一种字符串类型，可以通过使用其方法`setSpan`方法实现字符串各种形式风格的显示。

##具体案例

`setSpan`方法预览：

```
setSpan(Object what, int start, int end, int flags)
```

-  what       ：  文本格式，可以设置成前景色，背景色，下划线，中划线，模糊等
-  start        ：  字符串设置格式的起始下标
-  end         ：  字符串设置格式结束下标
-  flags        ：  标识

flags 常用的几种属性：

-   `Spanned.SPAN_INCLUSIVE_EXCLUSIVE`     从起始下标到结束下标，包括起始下标不包含结束坐标
-   `Spanned.SPAN_EXCLUSIVE_EXCLUSIVE`     从起始下标到结束下标，但都不包括起始下标和结束下标 
-   `Spanned.SPAN_INCLUSIVE_INCLUSIVE`      从起始下标到终了下标，同时包括起始下标和结束下标 
-   `Spanned.SPAN_EXCLUSIVE_INCLUSIVE`      从起始下标到终了下标，包括结束下标不包含起始坐标

###TextView点击获取部分内容

首先我们先来看`TextView`点击获取部分内容，对`TextView`设置`Spannable`，设置每段单词的响应方法`getEachParagraph()`

```
 testText.setText(getResources().getString(R.string.text), TextView.BufferType.SPANNABLE);
 getEachParagraph(testText);
 testText.setMovementMethod(LinkMovementMethod.getInstance());
```

点击响应方法`getEachParagraph()`：

```
//paragraph
public void getEachParagraph(TextView textView) {
    Spannable spans = (Spannable) textView.getText();
    Integer[] indices = getIndices(
            textView.getText().toString().trim(), ',');
    int start = 0;
    int end = 0;
    // recycle
    for (int i = 0; i <= indices.length; i++) {
        ClickableSpan clickSpan = getClickableSpan();
       //setspan
        end = (i < indices.length ? indices[i] : spans.length());
        spans.setSpan(clickSpan, start, end,
                Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
        start = end + 1;
    }
    //改变选中文本的高亮颜色
    textView.setHighlightColor(Color.BLUE);
}

//click
private ClickableSpan getClickableSpan() {
    return new ClickableSpan() {
        @Override
        public void onClick(View widget) {
            TextView tv = (TextView) widget;
            String s = tv
                    .getText()
                    .subSequence(tv.getSelectionStart(),
                            tv.getSelectionEnd()).toString();
            Log.e("onclick--:", s);
        }
        @Override
        public void updateDrawState(TextPaint ds) {
            ds.setColor(Color.BLACK);
            ds.setUnderlineText(false);
        }
    };
}

//array
public static Integer[] getIndices(String s, char c) {
        int pos = s.indexOf(c, 0);
        List<Integer> indices = new ArrayList<Integer>();
        while (pos != -1) {
            indices.add(pos);
            pos = s.indexOf(c, pos + 1);
        }
        return (Integer[]) indices.toArray(new Integer[0]);
    }
```

首先将`TextView`内容转换为`Spannable`对象；然后通过`getIndices`方法将文本内容通过字符`c`获取每段索引数组；然后为每段添加`ClickableSpan`；最后`onClick`方法里面获取每段内容。

###前景色ForegroundColorSpan

前景色等同于`setTextColor`，具体实现如下：

```
SpannableString spannableString = new SpannableString("设置文字的前景色");
ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.parseColor("#0099EE"));
spannableString.setSpan(colorSpan, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_INCLUSIVE);
mTv.setText(spannableString);

```

看看效果展示：

![sp](http://img.blog.csdn.net/20160620222956100)

起始下标包含5，结束下标`spannableString`长度不包含。

###背景色BackgroundColorSpan

背景色等同于`setBackgroundColor`，具体实现如下：

```
SpannableString spannableString = new SpannableString("设置文字的背景色");
BackgroundColorSpan colorSpan = new BackgroundColorSpan(Color.parseColor("#AC00FF30"));
spannableString.setSpan(colorSpan, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

效果图展示：

![sp](http://img.blog.csdn.net/20160620223523430)


###RelativeSizeSpan

先来看看效果图：

![sp](http://img.blog.csdn.net/20160620224909358)

我还清晰的记得刚开始做项目那会，做出这样的效果图需要2个`TextView`，`9月`一个；`22日`一个，现在想想真憋屈，要是相对字体大小在文本中间我不是要3个`TextView`，那真的要崩溃。

接着看看具体实现：

```
 SpannableString spannableString = new SpannableString("9月22日");
 RelativeSizeSpan sizeSpan = new RelativeSizeSpan(2.0f);
 spannableString.setSpan(sizeSpan, 0, 2, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
 mTv.setText(spannableString);
 
```

###中划线StrikethroughSpan

中划线用得比较多的就是过时的价格，看看它的具体实现：

方法一：

```
SpannableString spannableString = new SpannableString("我是文本中划线");
StrikethroughSpan strikethroughSpan = new StrikethroughSpan();
spannableString.setSpan(strikethroughSpan, 3, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

方法二：

```

  mTv.getPaint().setFlags(Paint.STRIKE_THRU_TEXT_FLAG | Paint.ANTI_ALIAS_FLAG);  // 设置中划线并加清晰
  
```

我们可以通过调用`mTv.getPaint().setFlags(0);  // 取消设置的的划线`

效果图登场：

![sp](http://img.blog.csdn.net/20160620230023441)


###下滑线UnderlineSpan

实现下滑线也有两种方法，同上：

方法一：

```

SpannableString spannableString = new SpannableString("我是文本下划线");
UnderlineSpan underlineSpan = new UnderlineSpan();
spannableString.setSpan(underlineSpan, 4, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

方法二：

```

 mTv.getPaint().setFlags(Paint.UNDERLINE_TEXT_FLAG); //下划线
 
```

效果图奉上：

![sp](http://img.blog.csdn.net/20160620230329146)


###上标SuperscriptSpan

上标加上前景色可以设计出未读消息效果，我们一起来看看它的实现：

```

SpannableString spannableString = new SpannableString("你有新消息了" + "●");
SuperscriptSpan superscriptSpan = new SuperscriptSpan();
ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.parseColor("#FF0000"));
spannableString.setSpan(colorSpan, 6, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(superscriptSpan, 6, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

以前看看文本未读消息感觉很高大上，还在苦苦追寻是怎么实现的，其实它非常`easy`下标还可以用于数学公式。

效果图：

![sp](http://img.blog.csdn.net/20160620231016023)

###下标SubscriptSpan

当然有了上标就会出现下标：

```

SpannableString spannableString = new SpannableString("注释1");
SubscriptSpan subscriptSpan = new SubscriptSpan();
spannableString.setSpan(subscriptSpan, 2, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);
        
```

效果图：

![sp](http://img.blog.csdn.net/20160620231406322)

###设置风格（粗体、斜体）StyleSpan

直接上代码：

```

SpannableString spannableString = new SpannableString("为文字设置粗体,斜体风格");
StyleSpan styleSpan_B = new StyleSpan(Typeface.BOLD);
StyleSpan styleSpan_I = new StyleSpan(Typeface.ITALIC);
spannableString.setSpan(styleSpan_B, 5, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(styleSpan_I, 8, 10, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

效果图贴上：

![sp](http://img.blog.csdn.net/20160620231702135)

###ImageSpan

做过IM相关需求的童鞋，都对这个不会陌生，它的身影在聊天`App`软件中随处可见：

```

SpannableString spannableString = new SpannableString("在文本中添加xx");
Drawable drawable = getResources().getDrawable(R.drawable.mini_face_nine);
int drawHeight = drawable.getMinimumHeight();
drawable.setBounds(0, 0, drawHeight, drawHeight);
ImageSpan imageSpan = new ImageSpan(drawable);
spannableString.setSpan(imageSpan, 6, 8, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

`setBounds`方法设置图片的边框大小，使图片与文本居中对齐。

效果图一览：

![sp](http://img.blog.csdn.net/20160620232240143)

###ClickableSpan

从名字上就可以知道和点击响应相关的：

```

SpannableString spannableString = new SpannableString("为文字设置点击事件");
spannableString.setSpan(new ClickableSpan() {
    @Override
    public void onClick(View view) {
        Snackbar.make(view, "你点击了我", Snackbar.LENGTH_SHORT).show();
        Uri uri = Uri.parse("http://www.baidu.com");
        Intent intent = new Intent(Intent.ACTION_VIEW, uri);
        intent.putExtra(Browser.EXTRA_APPLICATION_ID, _mActivity.getPackageName());
        try {
            _mActivity.startActivity(intent);
        } catch (ActivityNotFoundException e) {
            Log.e("--ClickableSpan--", "Actvity was not found for intent, " + intent.toString());
        }
    }
    @Override
    public void updateDrawState(TextPaint ds) {
        ds.setColor(Color.parseColor("#abc123"));
        ds.setUnderlineText(true);
    }
}, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setMovementMethod(LinkMovementMethod.getInstance());
mTv.setHighlightColor(Color.parseColor("#abc123"));
mTv.setText(spannableString);

```

为文本设置点击事件并且跳转网址。

效果图：

![sp](http://img.blog.csdn.net/20160620232947313)

###URLSpan

`URLSpan`跳转网址：

```

SpannableString spannableString = new SpannableString("为文字设置超链接");
URLSpan urlSpan = new URLSpan("http://www.baidu.com/");
spannableString.setSpan(urlSpan, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setMovementMethod(LinkMovementMethod.getInstance());
mTv.setHighlightColor(Color.parseColor("#ff0000"));
mTv.setText(spannableString);

```

效果图：

![sp](http://img.blog.csdn.net/20160620233225330)


###模糊BlurMaskFilter

比较简单的用法，直接上代码：

```

SpannableString spannableString = new SpannableString("为文字设置模糊");
MaskFilterSpan maskFilterSpan = new MaskFilterSpan(new BlurMaskFilter(10, BlurMaskFilter.Blur.NORMAL));
spannableString.setSpan(maskFilterSpan, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);

mTv.setText(spannableString);

```

构造方法：

```

BlurMaskFilter(float radius, Blur style)

```

-  radius    模糊半径
-  style      风格

`style`为枚举类型：

`BlurMaskFilter.Blur.NORMAL`  默认类型，模糊内外边界

![sp](http://img.blog.csdn.net/20160620234201381)

`BlurMaskFilter.Blur.INNER`   内部模糊    

![sp](http://img.blog.csdn.net/20160620234029512)

`BlurMaskFilter.Blur.OUTER`   外部模糊    

![sp](http://img.blog.csdn.net/20160620234351971)

`BlurMaskFilter.Blur.SOLID`   在边界内绘制固体，模糊

![sp](http://img.blog.csdn.net/20160620234908243)

###浮雕EmbossMaskFilter

```

SpannableString spannableString = new SpannableString("为文字设置浮雕");
MaskFilterSpan maskFilterSpan = new MaskFilterSpan(
        new EmbossMaskFilter(new float[]{10, 10, 10}, 0.5f, 1f, 1f));
spannableString.setSpan(maskFilterSpan, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

构造方法：

```
/**
     * Create an emboss maskfilter
     *
     * @param direction  array of 3 scalars [x, y, z] specifying the direction of the light source
     * @param ambient    0...1 amount of ambient light
     * @param specular   coefficient for specular highlights (e.g. 8)
     * @param blurRadius amount to blur before applying lighting (e.g. 3)
     * @return           the emboss maskfilter
     */
    public EmbossMaskFilter(float[] direction, float ambient, float specular, float blurRadius) 
    
```

//direction 是float数组，定义长度为3的数组标量[x,y,z]，来指定光源的方向
// ambient 取值在0到1之间，定义背景光 或者说是周围光
//  specular 定义镜面反射系数。
// blurRadius 模糊半径。

效果图：

![sp](http://img.blog.csdn.net/20160620235433776)

###光栅LayerRasterizer

```

SpannableString spannableString = new SpannableString("为文字设置光栅");
LayerRasterizer layerRasterizer = new LayerRasterizer();
layerRasterizer.addLayer(new Paint(Color.CYAN), 10, 10);
RasterizerSpan rasterizerSpan = new RasterizerSpan(layerRasterizer);
spannableString.setSpan(rasterizerSpan, 5, spannableString.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTv.setText(spannableString);

```

`addLayer`方法：

```

addLayer(Paint paint, float dx, float dy) //dx,dy便宜位置

```

效果图：

![sp](http://img.blog.csdn.net/20160620235810162)

看了这么多效果，你应该知道的这些效果显示。

##后续

`String`有`StringBuilder`用于字符串的拼接；那么`SpannableString`也有`SpannableStringBuilder`用于拼接`SpannableString`，可以把各种风格效果拼接在一起。

当然文中有什么描述错误，不当的地方，还请指出，欢迎关注后续博客交流，谢谢大家。

尽请关注

![sp](http://img.blog.csdn.net/20160623111558735)
