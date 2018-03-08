---
layout: post
title:  "Грокаем Spans."
published: true
author: "st235"
---

## Грокаем Spans.
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

![FontMetrics]({{ "/assets/images/fontmetrics.png" }})
