Title: Android TextView富文本自定义标签
Date: 2017-05-05 17:25
Category: Technology
Tags: Android, TextView
Slug: android-richtext
Author: xymelon

在Android开发中，经常会遇到UI同学的“奇思妙想”，比如同一行文本需要展示不同的样式（字体、颜色、背景、点击事件等），通常的做法当然是利用`TextView Spannable`针对不同的文本添加不同的span，而这个过程非常繁琐，所以利用空闲时间，写了个自定义富文本标签的开源库，欢迎大家拍砖并提出建议。

<!-- PELICAN_END_SUMMARY -->

## 特性
1. 超链接点击 hyperlink click event (with pressed text and background color)
1. 文本点击 click event (with pressed text and background color)
1. 文本背景色 text background color
1. 文本前景色 text foreground color
1. 文本大小 text size
1. 字体样式 text style (bold, italic)
1. 字体 text font

当然，本开源库扩展性非常强，你可以利用提供的api `addTypeSpan`添加自定义span。

![image](/images/RichText.gif)

动态图中， `Rich Text Format`为超链接，点击回调会返回当前链接，普通点击会返回当前word和word中心坐标；`Microsoft Corporation`指定了不同字体；更多特性你可自行添加。
## 安装
In your project level `build.gradle` :

```java
allprojects {
    repositories {
        ...
        maven { url "https://jitpack.io" }
    }
}
```
In your app level `build.gradle` :

```java
dependencies {
    compile 'com.github.xymelon:richtext:1.0.4'
}
```

## Demo

```java
TextView textView = (TextView) findViewById(R.id.textView);

final int foregroundTextColor = ContextCompat.getColor(this, R.color.T1);
final int linkTextColor = ContextCompat.getColor(this, R.color.colorPrimary);
final int normalTextColor = ContextCompat.getColor(this, R.color.R1);
final int pressedTextColor = ContextCompat.getColor(this, R.color.W1);
final int pressedBackgroundColor = ContextCompat.getColor(this, R.color.B1);
final Typeface georgiaTypeface = Typeface.createFromAsset(getAssets(), "fonts/Georgia Italic.ttf");

RichText richText = new RichText.Builder()
		//'c'标签添加ClickSpan
        .addBlockTypeSpan(new ClickSpan(
                normalTextColor,
                pressedTextColor,
                pressedBackgroundColor,
                new ClickSpan.OnClickListener() {
                    @Override
                    public void onClick(CharSequence text, float rawX, float rawY) {
                        Toast.makeText(MainActivity.this, text, Toast.LENGTH_SHORT).show();
                    }
                }), "c")
		//'f'，'t'标签添加ForegroundColorSpan
        .addBlockTypeSpan(new IStyleSpan() {
            @Override
            public CharacterStyle getStyleSpan() {
                return new ForegroundColorSpan(foregroundTextColor);
            }
        }, "f", "t")
		//'bi'标签添加StyleSpan
        .addBlockTypeSpan(new IStyleSpan() {
            @Override
            public CharacterStyle getStyleSpan() {
                return new StyleSpan(Typeface.BOLD_ITALIC);
            }
        }, "bi")
		//'s'标签添加TextAppearanceSpan
        .addBlockTypeSpan(new IStyleSpan() {
            @Override
            public CharacterStyle getStyleSpan() {
                return new TextAppearanceSpan(MainActivity.this, R.style.TextSize);
            }
        }, "s")
		//'t'标签添加FontTypefaceSpan
        .addBlockTypeSpan(new FontTypefaceSpan(georgiaTypeface), "t")
		//超链接标签默认'a'
        .addLinkTypeSpan(new LinkClickSpan(
                linkTextColor,
                pressedTextColor,
                pressedBackgroundColor,
                new LinkClickSpan.OnLinkClickListener() {
                    @Override
                    public void onClick(String url) {
                        Toast.makeText(MainActivity.this, url, Toast.LENGTH_SHORT).show();
                    }
                })
        )
        .build();
//notice: if set click span, you must invoke this method.
richText.with(textView);

//不同标签，指定不同span。
String tagString = "The <a href='https://en.wikipedia.org/wiki/Rich_Text_Format'>Rich Text Format</a> " +
        "is a <c>proprietary</c> <f>document</f> file format with published <bi>specification</bi> " +
        "developed by <t>Microsoft Corporation</t> from 1987 until 2008 for <s>cross-platform</s> document interchange with Microsoft products.";
textView.setText(richText.parse(tagString));
```

有疑问或者觉得不对的地方还请指正，谢谢。

Rich Text源码：<a href="https://github.com/xymelon/richtext" target="_blank">https://github.com/xymelon/richtext</a>
