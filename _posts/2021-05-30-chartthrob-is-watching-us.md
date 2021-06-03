---
layout: post

### ЗАГОЛОВОК СТРАНИЦЫ ###

# Текст заголовка
title: ChartThrob шпионит за вам

# Текст подзаголовка
subtitle: "What are you looking for?"

# Скрыть текст заголовка на странице
title_hide: true

# Разместить заголовок в тексте.
# Если закомментировать, то заголовок размещается
# на заглавном изображении (feature-img)
title_place: in-text


### ТИТУЛЬНЫЕ ИЗОБРАЖЕНИЯ СТРАНИЦЫ ###

# Заглавное изображение на странице
feature-img: images/2021-05-30-chartthrob-is-watching-us/chartthrob-head-image.jpg

# Изображение-миниатюра для страницы, выводимое по центру,
# при генерации списка страниц блога
thumbnail: images/2021-05-30-chartthrob-is-watching-us/chartthrob-head-image.jpg

# Изображение-миниатюра для страницы, выводимое слева
#thumbnail-left: images/2021-05-30-chartthrob-is-watching-us/cover-front.jpg
# Изображение-миниатюра для страницы, выводимое справа
#thumbnail-right: images/2021-05-30-chartthrob-is-watching-us/cover-back.jpg

# Изображение, используемое в тегах open grapg для соц.сетей
seo-img: images/2021-05-30-chartthrob-is-watching-us/chartthrobmod-cover-social.jpg

# Описание, используемое в тегах open grapg для соц.сетей
#description: meta-description

permalink: /chartthrob-is-watching-us/
excerpt_separator: <!--more-->

# Список тегов
tags:
    - ChartThrob
    - фотография
    - сценарий
    - Photoshop
    - расследования

---

*ChartThrob* — jsx-cценарий для Adobe Photoshop, используемый для создания *цифровых негативов* при работе в альтернативных фотопроцессах, таких как платиновая печать, амбротипия, цианотипия, гумойл и др. Однако после детального ознакомления cо сценарием выяснилось, что во время работы *ChartThrob* передает личные данные пользователя на серверы Google, с целью сбора статистики. Причем, делает это скрыто, ни о чем не предупреждая.
<!--more-->

Если вы занимаетесь альтернативными фотопроцессами, наверняка вы слышали о *ChartThrob* и, может быть, пользовались им. С его помощью можно получить более качественный *цифровой негатив для контактной печати*[^1]. Сценарий сравнивает отпечаток шкалы с ее эталонным изображением, и автоматически строит компенсационную кривую прямо в Photoshop. В общем случае, последовательность действий может быть следующей:

  1. При помощи *ChartThrob* сгенерировать эталонную шкалу.
  2. Получить цифровой негатив (фото-шаблон) шкалы, путем печати изображения на специальной пленке-носителе.
  3. Получить отпечаток шкалы выбранным альтернативным фото-процессом и отсканировать его. Как правило, без предварительной настройки, снимок получается с высокой контрастностью. Для компенсации этого эффекта, применяют кривую при изготовлении цифрового негатива.
  4. При помощи *ChartThrob* проанализировать отсканированное изображение отпечатка шкалы и построить компенсационную кривую.
  5. Создать новый цифровой негатив с использованием полученной кривой. При необходимости, шаги 2-5 повторить.

{% include
    aligner.html
    images="images/2021-05-30-chartthrob-is-watching-us/chartthrob-pipeline-1.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrob-pipeline-2.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrob-pipeline-3.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrob-pipeline-4.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrob-pipeline-5.jpg,"
    alts="Последовательность действий при работе с ChartThrob"
    column = "auto"
%}

К слову, *ChartThrob* не является уникальным, в сети можно найти еще несколько решений с подобными возможностями. Однако *ChartThrob* здесь выделяет то, что нет необходимости хранить изображение эталонной шкалы. Сценарий строит свою собственную шкалу и анализирует ее отпечаток. *ChartThrob* полностью автономен и не требует вспомогательных файлов. Он бесплатен и доступен для свободного скачивания на сайте автора [botzilla.com](http://www.botzilla.com/blog/archives/000544.html) [^2] или [GitHub](https://github.com/joker-b/ChartThrob) [^3].

Однако за красотой и удобством скрывается один недостаток: *ChartThrob&nbsp;v1.15* **шпионит за вами**. Причем, делает это скрыто, никак не афишируя свои действия. Более того, в документации к сценарию нигде не указан данный факт, и узнать о слежке можно только через анализ исходного кода.

Если в этот момент ваш внутренний параноик напрягся, то для этого есть все основания, поскольку для идентификации пользователя, **сценарий передает в систему Google Analytics серийный номер вашей версии Photoshop**.

Чтобы в этом убедиться, откройте сценарий в редакторе и перейдите к строке 150:

{% highlight javascript %}
147  ...
148  GaEmitter.prototype.sendPayload = function(payload) {
149     'use strict';
150     this.payload = this.required + payload;
151     this.call = 'POST /collect HTTP/1.1\r\nHost: ...'
152  ...
{% endhighlight %}

Как видно, в строке 150 формируется тело запроса, а в строке 151 — выполнятся непосредственно сам запрос. Чтобы увидеть содержимое запроса, внесем небольшие изменения в код, а именно: вставим строку с функцией `alert();`, которой в качестве аргумента передадим свойство `this.payload` (см. ниже строку 151):

{% highlight javascript %}
147  ...
148  GaEmitter.prototype.sendPayload = function(payload) {
149     'use strict';
150     this.payload = this.required + payload;
151     alert(this.payload);  
152     // this.call = 'POST /collect HTTP/1.1\r\nHost: ...'
153  ...
{% endhighlight %}

Теперь после запуска сценария, появится модальное окно:

{% include
    aligner.html
    images="images/2021-05-30-chartthrob-is-watching-us/chartthrob-alert-serial-number.jpg"
    alts="Передача серийного номера"
    column = "auto"
%}

Из представленного скриншота видно, что в запросе передается серийный номер Photoshop (переменная `cid`). Более того, информация отправляется каждый раз при запуске сценария, создании и анализе шкалы, а также в случае появления ошибок.

Согласитесь, не очень приятно.

Что можно сделать в данной ситуации? Небольшие исправления в коде помогут решить проблему.

Отмените все предыдущие изменения и перейдите к строке 55. Здесь в сценарии предусмотрена переменная `gDoNotTrack`. Зададим ей значение `true`, чтобы запретить передачу данных:

{% highlight javascript %}
49  ...
50  // global values /////////
51
52  var gVersion = 1.15;
53  var gDate = '5 January 2019';
54  var gTitle = 'ChartThrob V'+gVersion;
55  var gDoNotTrack = true; // set to true to disable web analytics
56  ...
{% endhighlight %}

Теперь при работе со сценарием можно не опасаться, что личные данные будут передаваться третьим лицам, и ваш внутренний параноик может успокоиться. А вот внутреннему эстету это вряд ли пока удастся.

## ChartThrobMod

Факт слежки за пользователем — главный, но не единственный недостаток *ChartThrob*.
При работе со сценарием можно заметить, что графический интерфейс основного окна не сбалансирован. Например, панели, из которых состоит окно, отличаются по размеру, а в названии верхней из них используется имя текущего файла, и если оно будет слишком длинным, внешний вид окна вряд ли назовешь привлекательным:

{% include
    aligner.html
    images="images/2021-05-30-chartthrob-is-watching-us/chartthrob-1.png,images/2021-05-30-chartthrob-is-watching-us/chartthrob-2.jpg"
    alts="Не сбалансированный интерфейс окна ChartThrob"
    column = "auto"
%}

Так же во время работы были выявлены следующие недостатки:

* При нажатии кнопки `Help` и закрытии окна справки, приходится каждый раз заново запускать сценарий вручную.  
* При активации чекбокса `Excessive Messages` появляется более сотни модальных окон, каждое из которых необходимо закрыть, тоже вручную.
* При создании шкалы с числовыми значениями (для этого используется чекбокс `Numbers`), выполняется значительно бóльшее количество операций, и сценарий как бы "замирает" Чувство неопределенности переполняет вас, поскольку не ясно, то ли сценарий все еще работает, то ли Photosop уже завис.

Чтобы устранить описанные неудобства, на основе *ChartThrob* была создана его **модифицированная версия** — [ChartThrobMod](https://github.com/chainick/ChartThrobMod) [^4], в которой теперь:

* Основное окно сценария имеет фиксированную ширину.
* Названия элементов окна стали более лаконичными.
* После закрытия окна справки не требуется повторный запуск сценария.
* Для каждого этапа работы отображается прогресс-бар.
* При необходимости, вся служебная информация о работе сценария выводится в текстовом поле. Об атаке полутора сотен всплывающих окон теперь можно не беспокоиться.

Также для новой версии выполнен перевод интерфейса на русский язык. Однако по умолчанию используется язык интерфейса, выбранный для Photoshop. Чтобы явно активировать русский язык, откройте в редакторе файл `ChartThrobMod.jsx`, перейдите к строке 31, и для переменной `$.locale` задайте значение `'ru'`, т.е. `$.locale = 'ru'`. При желании локализацию можно расширить, добавив в код перевод для нужного языка.

{% include
    aligner.html
    images="images/2021-05-30-chartthrob-is-watching-us/chartthrobmod-1.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrobmod-2.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrobmod-3.jpg,images/2021-05-30-chartthrob-is-watching-us/chartthrobmod-4.jpg,"
    alts="Не сбалансированный интерфейс окна ChartThrob"
    column = "auto"
%}

[^1]: Цифровой негатив для контактной печати — негативное или позитивное изображение на прозрачной пленке, выполненное цифровым способом (чаще всего путем печати на принтере) Цифровой негатив также может называться фото-шаблоном.
[^2]: [http://www.botzilla.com/blog/archives/000544.html](http://www.botzilla.com/blog/archives/000544.html)
[^3]: [https://github.com/joker-b/ChartThrob](https://github.com/joker-b/ChartThrob)
[^4]: [https://github.com/chainick/ChartThrobMod](https://github.com/chainick/ChartThrobMod)