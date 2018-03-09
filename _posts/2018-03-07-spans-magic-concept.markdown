---
layout: post
title:  "Грокаем Spans."
published: true
author: "st235"
---

__Замечание:__ эта статья - это перевод. [Оригинал](http://flavienlaurent.com/blog/2014/01/31/spans/).

### Как и когда применять Span?
- если Span влияет на форматирование отдельных символов, следует наследоваться от [CharacterStyle](https://developer.android.com/reference/android/text/style/CharacterStyle.html).
- если Span влияет на форматирование параграфа, следует наследоваться от [ParagraphStyle](https://developer.android.com/reference/android/text/style/ParagraphStyle.html).
- если Span изменяет внешний вид символа, следует реализовывать [UpdateAppearance](https://developer.android.com/reference/android/text/style/UpdateAppearance.html).
- если Span меняет размер или иную метрику шрифта, следует реализовывать [UpdateLayout](https://developer.android.com/reference/android/text/style/UpdateLayout.html).

### Окей, как же это работает?

#### Лейаутинг __(размещение)__

Для размещения и рендеринга текста в TextView применяется класс [Layout](https://developer.android.com/reference/android/text/Layout.html). Он содержит флаг [__boolean mSpannedText__](https://github.com/aosp-mirror/platform_frameworks_base/blob/b056324630b8adfeb38393bcab49f3b9c720f4fd/core/java/android/text/Layout.java#L232), который обозначает является ли текст наследником [SpannableString](https://developer.android.com/reference/android/text/SpannableString.html). **Layout** обрабатывает только [ParagraphStyle Spans](https://developer.android.com/reference/android/text/style/ParagraphStyle.html).

Метод __draw__ вызывает два других метода:

- drawBackground

Для каждой строки текста, если присутствует [LineBackgroundSpan](https://developer.android.com/reference/android/text/style/LineBackgroundSpan.html), вызывается [LineBackgroundSpan#drawBackground](https://developer.android.com/reference/android/text/style/LineBackgroundSpan.html#drawBackground(android.graphics.Canvas,%20android.graphics.Paint,%20int,%20int,%20int,%20int,%20int,%20java.lang.CharSequence,%20int,%20int,%20int)).

- drawText

Для каждой строки текста вычисляется [LeadingMarginSpan](https://developer.android.com/reference/android/text/style/LeadingMarginSpan.html) и [LeadingMarginSpan2](https://developer.android.com/reference/android/text/style/LeadingMarginSpan.LeadingMarginSpan2.html) и вызывает [LeadingMarginSpan#drawLeadingMargin](https://developer.android.com/reference/android/text/style/LeadingMarginSpan.html#drawLeadingMargin(android.graphics.Canvas,%20android.graphics.Paint,%20int,%20int,%20int,%20int,%20int,%20java.lang.CharSequence,%20int,%20int,%20boolean,%20android.text.Layout)), когда это необходимо. Также используется [AlignmentSpan](https://developer.android.com/reference/android/text/style/AlignmentSpan.html) для определения выравнивания текста. Наконец, если все преобразования для текущей строки вычислены, Layout вызывает TextLine#draw(для каждой строки создается объект TextLine).

#### TextLine

Согласно документации к [android.text.TextLine](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/TextLine.java): Представляет строку стилизованного текста готового для измерений и рендеринга.

TextLine класс содержит 3 набора Spans:

- MetricAffectingSpan
- CharacterStyle 
- ReplacementSpan

Однако, наибольшее внимание привлекает метод [TextLine#handleRun](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/TextLine.java#985), в котором все Spans-объекты используются для рендеринга текста. В соответсвии со Span, TextLine вызывает:

- CharacterStyle#updateDrawState для изменения параметров TextPaint полученных от MetricAffectingSpan и CharacterStyle Spans.
- TextLine#handleReplacement для ReplacementSpan. Этот метод вызывает [Replacement#getSize](https://developer.android.com/reference/android/text/style/ReplacementSpan.html#getSize(android.graphics.Paint,%20java.lang.CharSequence,%20int,%20int,%20android.graphics.Paint.FontMetricsInt)) чтобы получить ширину текста, который необходимо заменить, обновить его размеры, если это необходимо, и наконец вызвать Replacement#draw.

#### FontMetrics

Для того чтобы понять, как работаюет FontMetrics достаточно просто взглянуть на следующую картинку.

![FontMetrics]({{ "/assets/images/spans/fontmetrics.png" }})

### Playground

Все примеры, описанные в этой статье реализованы в [sample приложении](https://github.com/st235/GrokkingSpans).  

#### Bulletspan

[android.text.style.BulletSpan](https://developer.android.com/reference/android/text/style/BulletSpan.html)

**BulletSpan** влияет на стиль всего параграфа и позволяет добавить точку в начало параграфа.

```java
/*
public BulletSpan (int gapWidth, int color)
-gapWidth: gap in px between bullet and text
-color: bullet color (optionnal, default is transparent)
*/

//create a black BulletSpan with a gap of 15px
span = new BulletSpan(15, Color.BLACK);
```

![BulletSpan]({{ "/assets/images/spans/bullet.jpg" }})

#### QuoteSpan
android.text.style.QuoteSpan

The QuoteSpan affects paragraph-level text formatting. It allows you to put a quote vertical line on a paragraph.

```java
/*
public QuoteSpan (int color)
-color: quote vertical line color (optionnal, default is Color.BLUE)
*/

//create a red quote
span = new QuoteSpan(Color.RED);
```

![QuoteSpan]({{ "/assets/images/spans/quote.jpg" }})

#### AlignmentSpan.Standard
android.text.style.AlignmentSpan.Standard

The AlignmentSpan.Standard affects paragraph-level text formatting. It allows you to align (normal, center, opposite) a paragraph.

```java
/*
public Standard(Layout.Alignment align)
-align: alignment to set
*/

//align center a paragraph
span = new AlignmentSpan.Standard(Layout.Alignment.ALIGN_CENTER);
```

![AlignmentSpan.Standard]({{ "/assets/images/spans/standart.jpg" }})

#### UnderlineSpan
android.text.style.UnderlineSpan

The UnderlineSpan affects character-level text formatting. It allows you to underline a character thanks to Paint#setUnderlineText(true) .

```java
//underline a character
span = new UnderlineSpan();
```

![UnderlineSpan]({{ "/assets/images/spans/underline.jpg" }})

#### StrikethroughSpan
android.text.style.StrikethroughSpan

The StrikethroughSpan affects character-level text formatting. It allows you to strikethrough a character thanks to Paint#setStrikeThruText(true)) .

```java
//strikethrough a character
span = new StrikethroughSpan();
```

![StrikethroughSpan]({{ "/assets/images/spans/strikethrough.jpg" }})

#### SubscriptSpan
android.text.style.SubscriptSpan

The SubscriptSpan affects character-level text formatting. It allows you to subscript a character by reducing the TextPaint#baselineShift .

```java
//subscript a character
span = new SubscriptSpan();
```

![Subscript]({{ "/assets/images/spans/subscript.jpg" }})

#### SuperscriptSpan
android.text.style.SuperscriptSpan

The SuperscriptSpan affects character-level text formatting. It allows you to superscript a character by increasing the TextPaint#baselineShift .

```java
//superscript a character
span = new SuperscriptSpan();
```

![Superscript]({{ "/assets/images/spans/superscript.jpg" }})

#### BackgroundColorSpan
android.text.style.BackgroundColorSpan

The BackgroundColorSpan affects character-level text formatting. It allows you to set a background color on a character.

```java
/*
public BackgroundColorSpan (int color)
-color: background color
*/

//set a green background
span = new BackgroundColorSpan(Color.GREEN);
```

![BackgroundColorSpan]({{ "/assets/images/spans/background.jpg" }})

#### ForegroundColorSpan
android.text.style.ForegroundColorSpan

The ForegroundColorSpan affects character-level text formatting. It allows you to set a foreground color on a character.

```java
/*
public ForegroundColorSpan (int color)
-color: foreground color
*/

//set a red foreground
span = new ForegroundColorSpan(Color.RED);
```

![ForegroundColorSpan]({{ "/assets/images/spans/foreground.jpg" }})

#### ImageSpan
android.text.style.ImageSpan

The ImageSpan affects character-level text formatting. It allows you to a character by an image. It’s one of the few span that is well documented so enjoy it!

```java
//replace a character by pic1_small image
span = new ImageSpan(this, R.drawable.pic1_small);
```

![ImageSpan]({{ "/assets/images/spans/image.jpg" }})

#### StyleSpan
android.text.style.StyleSpan

The StyleSpan affects character-level text formatting. It allows you to set a style (bold, italic, normal) on a character.

```java
/*
public StyleSpan (int style)
-style: int describing the style (android.graphics.Typeface)
*/

//set a bold+italic style
span = new StyleSpan(Typeface.BOLD | Typeface.ITALIC);
```

![StyleSpan]({{ "/assets/images/spans/style.jpg" }})

#### TypefaceSpan
android.text.style.TypefaceSpan

The TypefaceSpan affects character-level text formatting. It allows you to set a font family (monospace, serif etc) on a character.

```java
/*
public TypefaceSpan (String family)
-family: a font family
*/

//set the serif family
span = new TypefaceSpan("serif");
```

![TypefaceSpan]({{ "/assets/images/spans/typeface.jpg" }})

#### TextAppearanceSpan
android.text.style.TextAppearanceSpan

The TextAppearanceSpan affects character-level text formatting. It allows you to set a appearance on a character.

```java
/*
public  TextAppearanceSpan(Context context, int appearance, int colorList)
-context: a valid context
-appearance: text appearance resource (ex: android.R.style.TextAppearance_Small)
-colorList: a text color resource (ex: android.R.styleable.Theme_textColorPrimary)

public TextAppearanceSpan(String family, int style, int size, ColorStateList color, ColorStateList linkColor)
-family: a font family
-style: int describing the style (android.graphics.Typeface)
-size: text size
-color: a text color
-linkColor: a link text color
*/

//set the serif family
span = new TextAppearanceSpan(this/*a context*/, R.style.SpecialTextAppearance);
```

__styles.xml__
```xml
<style name="SpecialTextAppearance" parent="@android:style/TextAppearance">
    <item name="android:textColor">@color/color1</item>
    <item name="android:textColorHighlight">@color/color2</item>
    <item name="android:textColorHint">@color/color3</item>
    <item name="android:textColorLink">@color/color4</item>
    <item name="android:textSize">28sp</item>
    <item name="android:textStyle">italic</item>
</style>
```

#### AbsoluteSizeSpan
android.text.style.AbsoluteSizeSpan

The AbsoluteSizeSpan affects character-level text formatting. It allows you to set an absolute text size on a character.

```java
/*
public AbsoluteSizeSpan(int size, boolean dip)
-size: a size
-dip: false, size is in px; true, size is in dip (optionnal, default false)
*/

//set text size to 24dp
span = new AbsoluteSizeSpan(24, true);
```

![AbsoluteSizeSpan]({{ "/assets/images/spans/absolutesize.jpg" }})

#### RelativeSizeSpan
android.text.style.RelativeSizeSpan

The RelativeSizeSpan affects character-level text formatting. It allows you to set an relative text size on a character.

```java
/*
public RelativeSizeSpan(float proportion)
-proportion: a proportion of the actual text size
*/

//set text size 2 times bigger 
span = new RelativeSizeSpan(2.0f);
```

![RelativeSizeSpan]({{ "/assets/images/spans/relativesize.jpg" }})

#### ScaleXSpan
android.text.style.ScaleXSpan

The ScaleXSpan affects character-level text formatting. It allows you to scale on x a character.

```java
/*
public ScaleXSpan(float proportion)
-proportion: a proportion of actual text scale x
*/

//scale x 3 times bigger 
span = new ScaleXSpan(3.0f);
```

![ScaleXSpan]({{ "/assets/images/spans/scalex.jpg" }})

#### MaskFilterSpan
android.text.style.MaskFilterSpan

The MaskFilterSpan affects character-level text formatting. It allows you to set a android.graphics.MaskFilter on a character.

Warning: BlurMaskFilter is not supported with hardware acceleration.

```java
/*
public MaskFilterSpan(MaskFilter filter)
-filter: a filter to apply
*/

//Blur a character
span = new MaskFilterSpan(new BlurMaskFilter(density*2, BlurMaskFilter.Blur.NORMAL));
//Emboss a character
span = new MaskFilterSpan(new EmbossMaskFilter(new float[] { 1, 1, 1 }, 0.4f, 6, 3.5f));
```

- BlurMaskFilter

**Важно:** эта маска [не поддерживает аппартаное ускорение](https://stackoverflow.com/questions/11281404/android-blurmaskfilter-has-no-effect-in-canvas-drawoval-while-text-is-blurred).

EmbossMaskFilter with a blue ForegroundColorSpan and a bold StyleSpan

![MaskFilterSpan]({{ "/assets/images/spans/maskfilter.jpg" }})

