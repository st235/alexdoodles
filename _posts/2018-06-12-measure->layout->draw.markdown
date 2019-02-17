---
layout: post
title:  "Measure->Layout->Draw. Layouts & Views concepts."
published: false
author: "st235"
---

Очень часто, смотря какое-нибудь приложение невольно задумываешься о том, как реализованы компоненты на разметке или логика их расстановки, как этот компонент называет, пока в итоге не приходишь к выводу, что этот компонент - не системный и его логику реализовал автор приложения. В Android к нам на помощь приходит [View](https://developer.android.com/reference/android/view/View) и [View Group](https://developer.android.com/reference/android/view/ViewGroup), однако, с большой силой приходит и большая ответственность.
Давайте разберемся с правилами написания кастомных View-элементов.

Все примеры, которые я буду приводить в качестве референсных, реализованы в виде отдельного [Sample App](https://github.com/st235/LayoutPlusViews.).

## Основа 
View - это базовый элемент для отображения в Android.
ViewGroup - 
