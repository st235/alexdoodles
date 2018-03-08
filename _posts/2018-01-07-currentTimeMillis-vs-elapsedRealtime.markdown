---
layout: post
title:  "System.currentTimeMillis() против SystemClock.elapsedRealtime()"
published: true
author: "st235"
---

Вы когда-нибудь пробовали рассчитать время между двумя событиями? Обычно для этого используют `System.currentTimeMillis()`, но я рекомендую вместо этого познакомиться с `SystemClock.elapsedRealtime()`.

`System.currentTimeMillis()` вернет стандартное "настенное" время, выраженное в милисекундах. Это время, к несчастью, может быть изменено через сеть или пользователем вручную [(смотри setCurrentTimeMillis(long))](https://developer.android.com/reference/android/os/SystemClock.html#setCurrentTimeMillis(long)), что приводит к видимым скачкам.

`SystemClock.elapsedRealtime()` возврщает время с момента запуска системы, включая режим глубокого сна. Эти часы гарантированно монотонны и продолжают идти даже когда процессор находится в режиме энергосбереженя. `elapsedRealtimeNanos()` можно использовать для получения `настоящего времени` в наносекундах. 

Например, необходимо посчитать время исполнения алгоритма

```
long startTime = System.currentTimeMillis();
// какой-то код
long endTime = System.currentTimeMillis();
```

для этого нужно засечь время в начале алгоритма, после его исполнения и вычислить разницу.

```
long duration = endTime - startTime; 
```

Что произойдет если мы попробуем установить время между вызовами? Ничего хорошего :-(


Используйте `SystemClock.elapsedRealtime()` вместо `System.currentTimeMillis()` чтобы не зависеть от методов, способных выставить время вручную. 

Модифицируем программу верно:

```
long startTime = SystemClock.elapsedRealtime();
// какой-то код
long endTime = SystemClock.elapsedRealtime();
long duration = endTime - startTime;
```
