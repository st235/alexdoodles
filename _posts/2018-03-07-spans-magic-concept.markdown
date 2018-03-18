---
layout: post
title:  "Грокаем Spans."
published: true
author: "st235"
---

Очень часто, листая очередную сторис из инстаграма или просматривая пост в фейсбуке, я замечал, что невольно задумывался - как реализован тот или иной механизм выделения текста. После небольшого исследования я пришел к выводу, что в Android OS уже релизован мощный механизм работы с текстом - Spans.

__Замечание:__ эта статья основана на официальной документации, ответов со stackoverflow, статьи  [Spans, a Powerful Concept](http://flavienlaurent.com/blog/2014/01/31/spans/), [AOSP](https://github.com/aosp-mirror/platform_frameworks_base) и является агргерующей. Так же для этой статьи я написал [тестовое приложение](https://github.com/st235/GrokkingSpans), где вы можете посмотреть приемы, которые я применял в работе со Spannable.

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

Для того чтобы понять, как работает FontMetrics достаточно просто взглянуть на следующую картинку.

![FontMetrics]({{ "/assets/images/spans/fontmetrics.png" }})

### Playground

Все примеры, описанные в этой статье реализованы в [sample приложении](https://github.com/st235/GrokkingSpans).  

#### BulletSpan

[android.text.style.BulletSpan](https://developer.android.com/reference/android/text/style/BulletSpan.html)

**BulletSpan** влияет на стиль всего параграфа и позволяет добавить точку в начало параграфа.

```java
/*
public BulletSpan (int gapWidth, int color)
-gapWidth: gap in px between bullet and text
-color: bullet color (optionnal, default is transparent)
*/

// create a cyan BulletSpan with a gap of 8dp
// note: toPx - converts dp to pixels
span = new BulletSpan(toPx(8), Color.CYAN);
```

![BulletSpan]({{ "/assets/images/spans/bullet.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### QuoteSpan
[android.text.style.QuoteSpan](https://developer.android.com/reference/android/text/style/QuoteSpan.html)

QuoteSpan влияет на стиль параграфа и позволяет поставить вертикальную полосу цитирования.

```java
/*
public QuoteSpan (int color)
-color: quote vertical line color (optionnal, default is Color.BLUE)
*/

// create a cyan quote
span = new QuoteSpan(Color.CYAN);
```

![QuoteSpan]({{ "/assets/images/spans/quote.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### AlignmentSpan.Standard
[android.text.style.AlignmentSpan.Standard](https://developer.android.com/reference/android/text/style/AlignmentSpan.Standard.html)

The AlignmentSpan.Standard форматирует стиль параграфа и позволяет поменять выравнивание текста (нормально, центрирован, выровнен по противоположному краю).

```java
/*
public Standard(Layout.Alignment align)
-align: alignment to set
*/

// align center a paragraph
span = new AlignmentSpan.Standard(Layout.Alignment.ALIGN_OPPOSITE);
```

![AlignmentSpan.Standard]({{ "/assets/images/spans/standart.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### UnderlineSpan
[android.text.style.UnderlineSpan](https://developer.android.com/reference/android/text/style/UnderlineSpan.html)

UnderlineSpan влияет на отдельный символ. Позволяет проставить подчеркивание у отдельно заданных символов, вызывая [Paint#setUnderlineText(true)](https://developer.android.com/reference/android/graphics/Paint.html#setUnderlineText(boolean)).

```java
// underline a character
span = new UnderlineSpan();
```

![UnderlineSpan]({{ "/assets/images/spans/underline.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### StrikethroughSpan
[android.text.style.StrikethroughSpan](https://developer.android.com/reference/android/text/style/StrikethroughSpan.html)

StrikethroughSpan также изменяет отдельный символ, добавляя к нему зачеркивание. Вызывает: [Paint#setStrikeThruText(true))](https://developer.android.com/reference/android/graphics/Paint.html#setStrikeThruText(boolean)).

```java
// strikethrough a character
span = new StrikethroughSpan();
```

![StrikethroughSpan]({{ "/assets/images/spans/strikethrough.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### SubscriptSpan
[android.text.style.SubscriptSpan](https://developer.android.com/reference/android/text/style/SubscriptSpan.html)

SubscriptSpan позволяет опустить символы, относительно базовой линии, с помощью смещения этой линии [TextPaint#baselineShift](https://developer.android.com/reference/android/text/TextPaint.html#baselineShift). Влияет на отдельно взятый символ.

```java
// subscript a character
span = new SubscriptSpan();
```

![Subscript]({{ "/assets/images/spans/subscript.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### SuperscriptSpan
[android.text.style.SuperscriptSpan](https://developer.android.com/reference/android/text/style/SuperscriptSpan.html)

SuperscriptSpan поднимает символы относительно базовой линии, путем ее смещения [TextPaint#baselineShift](https://developer.android.com/reference/android/text/TextPaint.html#baselineShift). Влияет на отдельно взятый символ.

```java
// superscript a character
span = new SuperscriptSpan();
```

![Superscript]({{ "/assets/images/spans/superscript.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### BackgroundColorSpan
[android.text.style.BackgroundColorSpan](https://developer.android.com/reference/android/text/style/BackgroundColorSpan.html)

BackgroundColorSpan устанавливает фон для каждого символа.

```java
/*
public BackgroundColorSpan (int color)
-color: background color
*/

// set a cyan background
span = new BackgroundColorSpan(Color.CYAN);
```

![BackgroundColorSpan]({{ "/assets/images/spans/background.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### ForegroundColorSpan
[android.text.style.ForegroundColorSpan](https://developer.android.com/reference/android/text/style/ForegroundColorSpan.html)

ForegroundColorSpan устанавливает цвет текста. Влияет на отдельно взятый символ.

```java
/*
public ForegroundColorSpan (int color)
-color: foreground color
*/

// set a cyan foreground
span = new ForegroundColorSpan(Color.CYAN);
```

![ForegroundColorSpan]({{ "/assets/images/spans/foreground.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### ImageSpan
[android.text.style.ImageSpan](https://developer.android.com/reference/android/text/style/ImageSpan.html)

ImageSpan влияет на форматирование символов. Позволяет заменить 1 или несколько символов изображением. Довольно редко используется, но обладает отличной документацией.

```java
//replace a character by cat image
span = new ImageSpan(getApplicationContext(), R.drawable.cat);
```

![ImageSpan]({{ "/assets/images/spans/image.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### StyleSpan
[android.text.style.StyleSpan](https://developer.android.com/reference/android/text/style/StyleSpan.html)

StyleSpan позволяет выставить стиль символа (bold, italic, normal).

```java
/*
public StyleSpan (int style)
-style: int describing the style (android.graphics.Typeface)
*/

// set a bold style
span = new StyleSpan(Typeface.BOLD);
```

![StyleSpan]({{ "/assets/images/spans/style.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### TypefaceSpan
[android.text.style.TypefaceSpan](https://developer.android.com/reference/android/text/style/TypefaceSpan.html)

TypefaceSpan позволяет указать font family (monospace, serif etc) отдельно взятого символа.

```java
/*
public TypefaceSpan (String family)
-family: a font family
*/

// set the serif family
span = new TypefaceSpan("serif");
```

![TypefaceSpan]({{ "/assets/images/spans/typeface.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### TextAppearanceSpan
[android.text.style.TextAppearanceSpan](https://developer.android.com/reference/android/text/style/TextAppearanceSpan.html)

TextAppearanceSpan влияет на стиль каждого символа. Позволяет гибко настраивать комплексные параметры текста. 

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
[android.text.style.AbsoluteSizeSpan](https://developer.android.com/reference/android/text/style/AbsoluteSizeSpan.html)

AbsoluteSizeSpan позволяет выставить размер отдельно взятого символа в абсолютных величинах (пикселах).

```java
/*
public AbsoluteSizeSpan(int size, boolean dip)
-size: a size
-dip: false, size is in px; true, size is in dip (optionnal, default false)
*/

// set text size to 8dp
span = new new AbsoluteSizeSpan(toPx(8));
```

![AbsoluteSizeSpan]({{ "/assets/images/spans/absolutesize.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### RelativeSizeSpan
[android.text.style.RelativeSizeSpan](https://developer.android.com/reference/android/text/style/RelativeSizeSpan.html)

RelativeSizeSpan позволяет выставить высоту отдельно взятого символа, относительно текущего размера текста.

```java
/*
public RelativeSizeSpan(float proportion)
-proportion: a proportion of the actual text size
*/

// set text size 2 times bigger 
span = new RelativeSizeSpan(2.0f);
```

![RelativeSizeSpan]({{ "/assets/images/spans/relativesize.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### ScaleXSpan
[android.text.style.ScaleXSpan](https://developer.android.com/reference/android/text/style/ScaleXSpan.html)

ScaleXSpan позволяет растянуть текст вдоль оси x. Влияет на отдельно взятый символ.

```java
/*
public ScaleXSpan(float proportion)
-proportion: a proportion of actual text scale x
*/

//scale x 2 times bigger 
span = new ScaleXSpan(2.0f);
```

![ScaleXSpan]({{ "/assets/images/spans/scalex.jpg" }}){:style="width: 70% !important; margin: auto;"}

#### MaskFilterSpan
[android.text.style.MaskFilterSpan](https://developer.android.com/reference/android/text/style/MaskFilterSpan.html)

MaskFilterSpan оказывает влияение на каждый символ по-отдельности. Позволяет устанавливать [android.graphics.MaskFilter](https://developer.android.com/reference/android/graphics/MaskFilter.html) для символов.

```java
/*
public MaskFilterSpan(MaskFilter filter)
-filter: a filter to apply
*/

// blur a character
span = new MaskFilterSpan(new BlurMaskFilter(toPx(4), BlurMaskFilter.Blur.NORMAL));
// emboss a character
span = new MaskFilterSpan(new EmbossMaskFilter(new float[] { 1, 1, 1 }, 0.4f, 6, 3.5f));
```

- BlurMaskFilter

**Важно:** эта маска [не поддерживает аппартаное ускорение](https://stackoverflow.com/questions/11281404/android-blurmaskfilter-has-no-effect-in-canvas-drawoval-while-text-is-blurred).

- EmbossMaskFilter

![MaskFilterSpan]({{ "/assets/images/spans/maskfilter.jpg" }}){:style="width: 70% !important; margin: auto;"}

## Примеры работы со Spans
### Реализуем свой собственный простой Span

В этом разделе мы увидим способ создания пользовательского Span, открывающего интересные перспективы для настройки текста.

Во-первых, мы должны отнаследоваться от абстрактного класса [ReplacementSpan](https://developer.android.com/reference/android/text/style/ReplacementSpan.html).

**Внимение**:  __если вы хотите только отрисовать собственный фон, необходимо использовать [LineBackgroundSpan](https://developer.android.com/reference/android/text/style/LineBackgroundSpan.html), который находится на уровне абзаца.__

Необходимо реализовать следующие 2 метода:

- getSize: метод, возвращающий размер вашей замены.

__text__: текст, управляемый Span
__start__: начальный индекс текста
__end__: конечный индекс текста
__fm__: метрики шрифта (может быть нулем)

- draw: здесь вы можете рисовать с помощью Canvas.

__x__: x-координата, где рисовать текст
__top__: верх линии
__y__: базовый уровень (baseline)
__bottom__: нижняя граница линии

#### FrameSpan

```java
public class FrameSpan extends ReplacementSpan {

    private int width;

    @Override
    public int getSize(@NonNull Paint paint, CharSequence text, int start, int end, @Nullable Paint.FontMetricsInt fm) {
        return  (int) paint.measureText(text, start, end);
    }

    @Override
    public void draw(@NonNull Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, @NonNull Paint paint) {
        int color = paint.getColor();
        paint.setColor(Color.YELLOW);
        width = (int) paint.measureText(text, start, end);
        canvas.drawRect(x, top, x + width, bottom, paint);
        paint.setColor(color);
    }
}
```

#### RoundBackgroundSpan

```java
public class RoundBackgroundSpan implements LineBackgroundSpan {

    private final int padding = 0;
    private final RectF backgroundRect = new RectF();

    @Override
    public void drawBackground(Canvas c,
                               Paint p,
                               int left,
                               int right,
                               int top,
                               int baseline,
                               int bottom,
                               CharSequence text,
                               int start, int end, int lnum) {
        if(text == null) return;
        int textWidth = Math.round(p.measureText(text, start, end));
        int paintColor = p.getColor();
        backgroundRect.set(left - padding, top - lnum == 0 ?  padding / 2  :  -(padding / 2),
        left + textWidth + padding, bottom + (padding / 2));
        p.setColor(Color.CYAN);
        c.drawRoundRect(backgroundRect, toPx(4), toPx(4), p);
        p.setColor(paintColor);
    }
}
```

### Анимируем ForegoundColor

ForegroundColorSpan доступен только для чтения. 
Это означает, что вы не можете изменить цвет переднего плана после создания экзепляра ForegroundColorSpan.
Итак, первое, что нужно сделать, это написать MutableForegroundColorSpan.

#### MutableForegroundColorSpan

```java
public class MutableForegroundColorSpan extends ForegroundColorSpan {

    @ColorInt
    private int foregroundColor;

    public MutableForegroundColorSpan(@ColorInt int color) {
        super(color);
        foregroundColor = color;
    }

    public MutableForegroundColorSpan(Parcel src) {
        super(src);
        foregroundColor = src.readInt();
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        super.writeToParcel(dest, flags);
        dest.writeInt(foregroundColor);
    }

    @Override
    public void updateDrawState(TextPaint ds) {
        ds.setColor(getForegroundColor());
    }

    @Override
    public int getForegroundColor() {
        return foregroundColor;
    }

    public void setForegroundColor(@ColorInt int foregroundColor) {
        this.foregroundColor = foregroundColor;
    }
}
```

Создаем Property для Animator

```java
    private final Property<MutableForegroundColorSpan, Integer> MUTABLE_FOREGROUND_COLOR_PROPERTY =
            new Property<MutableForegroundColorSpan, Integer>(Integer.class, "MUTABLE_FOREGROUND_COLOR_PROPERTY") {
                @Override
                public Integer get(MutableForegroundColorSpan object) {
                    return object.getForegroundColor();
                }

                @Override
                public void set(MutableForegroundColorSpan object, Integer value) {
                    object.setForegroundColor(value);
                }
            };
```

```java
        TextView animatedTextView = new TextView(this);
        addContentView(animatedTextView, new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT));

        SpannableString text = new SpannableString(getString(R.string.text));
        MutableForegroundColorSpan span = new MutableForegroundColorSpan(Color.BLACK);
        text.setSpan(span, 0, text.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        animatedTextView.setText(text);

        ObjectAnimator animator = ObjectAnimator.ofInt(span, MUTABLE_FOREGROUND_COLOR_PROPERTY, Color.BLACK, Color.CYAN, Color.DKGRAY);
        animator.setEvaluator(new ArgbEvaluator());
        animator.addUpdateListener(animation -> {
            animatedTextView.invalidate();
        });
        animator.setDuration(3500L);
        animator.start();
```

### Вместо заключения

На мой взгляд, любое мобильное приложение на 90% состоит из текста и картинок, а подобные механизмы расширения имеющихся возможностей, почему-то не пользуются по-полной. Я постарался осветить аспекты работы со Spans, для того чтобы Вы смогли разнообразить свои приложения, имея в руках такой мощный инструмент.
