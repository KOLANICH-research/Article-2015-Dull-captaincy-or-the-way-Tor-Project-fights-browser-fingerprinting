# Унылое капитанство или как Tor Project борется с browser fingerprinting

В [статье](http://cseweb.ucsd.edu/~hovav/dist/canvas.pdf) известного в определённых кругах [Hovav Shacam](http://cseweb.ucsd.edu/~hovav/papers.html) и [Keaton Mowery](http://cseweb.ucsd.edu/~kmowery/) от 2012 года был описан новый на тот момент метод [генерации идентификатора браузера и системы прямо из JavaScript](https://panopticlick.eff.org/browser-uniqueness.pdf) - Canvas Fingerprinting. Метод позволял различать, в том числе, оборудование ...

# Статья

Суть метода в том, что на разных системах в разных браузерах по-разному рендерится текст (и не только текст), так как за это отвечает много различных компонентов на различных уровнях, которые могут иметь разные настройки для компонентов нижележащего уровня. 

1.  Могут быть банально разные (с разным набором символов, с немного разными глифами, с разными лигатурами и кернингом...) шрифты.
2.  Разные параметры вызова библиотечных функций в разных браузерах.
3.  Разные версии [libfreetype](http://www.freetype.org/) и прочих библиотек рендеринга.
4.  Разные реализации в ОС и разные настройки ОС (например, [разные версии <acronym title="Технология субпиксельного сглаживания шрифта, из-за чего шрифты выглядят чётче">ClearType</acronym>](http://habrahabr.ru/post/247703/) и различное разрешение экрана).
5.  Разные графические драйвера.
6.  Разное графическое железо.

В статье описан способ с использованием <abbr title="Application Programming Interface">API</abbr> [getImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D.getImageData), который возвращает картинку попиксельно. Также рассмотрено использование <acronym title="OpenGL в веб-страницах">WebGL</acronym> : на разных системах и 3D-сцены рендерятся по-разному.

В статье была дана рекомендация использовать чисто софтовый рендеринг, без использования компонентов ОС и прочего ПО, установленного на ПК, зашумление и прочие методики защиты. В Tor Project возможность отпечаткинга залатали, спрашивая разрешения на getImageData и заменяя случайные шрифты на fallback-шрифт (поэтому мы делаем замеры 10 раз). 

# Defective by design

Есть другие API, текущие информацией. Например, API измерения текста. Если текст рендерится по-разному, значит и размеры должны быть чуточку разные. Проверим эту гипотезу, используя API [measureText](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D.measureText) для того же холста и API [getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element.getBoundingClientRect) из DOM.

<acronym title="git clone https://gist.github.com/KOLANICH/00b9145743d841cff4d7.git">Склонируйте</acronym> [gist](https://gist.github.com/KOLANICH/00b9145743d841cff4d7), запустите у себя на компе последний [<abbr title="TOR Browser Bundle, набор программ для анонимизированного сёрфинга">TBB</acronym>](https://www.torproject.org/download/download.html.en), откройте в нём HTML-файл из репозитория, отпостите результат в комментах. В принципе можно было сделать не хеши, это дало бы больше простора для дата-майнинга, но даже в сжатом виде полный набор информации о каждом шрифте занимает очень много места. Поэтому обойдёмся хешами. Кто хочет копнуть глубже, тот может раскомментировать несколько строчек в исходнике.

# Результаты

## Различные браузеры

Сначала запустим на Firefox стабильной и ночной версий.
### Firefox 35.0 и Firefox Nightly 37.0a1 на Windows 8.1
```json
{
	"Impact" : "16b207dfaab7643d19dfa45c",
	"Courier New" : "7cdc70fc5acdb4fdfbf91150",
	"Bookman Old Style" : "87bccbb4027b44c3ccec316e",
	"Consolas" : "2f15e176ec12eedab6a2964c",
	"MS Gothic" : "83bd03a90697218616b392ec",
	"Constantia" : "e520f8ba166cc561aafa1bfa",
	"Calibri" : "e7be39be54baeea86efe204d",
	"Cambria" : "364950a4a4e688d3a455a0a3",
	"Wingdings" : "f1ec609eb0ee165edcca0852",
	"Webdings" : "d6cb6d13744445a8052abf9f",
	"Symbol" : "472d3c8f99a96b8d182889b5",
	"Ubuntu Mono" : "dd9b8aa29b7744ad8d5a58f0",
	"Inconsolata LGC" : "bce92cecba3e4646c2e2720d",
	"Source Code Pro" : "baebc2fcd75bd7b72d6745b4",
	"Lucida Handwriting" : "d802c472637db0c3c35f7dc0",
	"Georgia" : "7eb28794ff6a47f9d4617ea5",
	"serif" : {
		"d" : "6f5646f292ff59c85e828f7d",
		"fonts" : ["Times New Roman", "Droid Sans", "DejaVu Sans", "Inconsolata", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "6b043ec519448e8dac79e4ee",
		"fonts" : ["Arial"]
	}
}
```

Дают одинаковые отпечатки.

А теперь поиграемся с параметрами. Игра с параметрами ClearType ничего не дала, как и включение <acronym title="API для рендеринга с аппаратным ускорением от Microsoft">DirectWrite</acronym>, зато включение <abbr title="project electrolysis - проект по использованию в Firefox нескольких процессов (как сделано в Chromium), что должно улучшить безопасность и стабильность">e10s</abbr> изменило отпечаток и полностью порушило рендеринг (все страницы рендерились в чёрный прямоугольник).
### Firefox Nightly 37.0a1 c e10s включённым
```json
{
	"Impact" : "004c408dbbbdb9c9910c4352",
	"Courier New" : "d1a0a33b5a8fc28faf6586f7",
	"Bookman Old Style" : "fe957ed9b9d3d352564b832d",
	"Consolas" : "74831d1303efc9733ab93cb1",
	"MS Gothic" : "ed7dbb5235608f4c2b38157f",
	"Constantia" : "c70150a0ed768f5e58196f96",
	"Calibri" : "94311fb530817035c6d0e67b",
	"Cambria" : "0013175b346f8ff058a53e16",
	"Wingdings" : "547a5121752201d862bea0bc",
	"Webdings" : "a98b6f18a2464e8f4ac0c06b",
	"Symbol" : "be2745f60719b4350900e044",
	"Ubuntu Mono" : "7456b9713327556503f5f3df",
	"Inconsolata LGC" : "49b5231c4cf98df7b253feba",
	"Source Code Pro" : "12e997bcc449b894550266f9",
	"Lucida Handwriting" : "d293b8185d9fdb4060403a73",
	"Georgia" : "5c7d4b20b601096c0f742e01",
	"System" : "3b0cd6cc6c88c08c7130c0c5",
	"serif" : {
		"d" : "c42ce0f2eadd307131c36cdb",
		"fonts" : ["Times New Roman", "Droid Sans", "DejaVu Sans", "Inconsolata", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "143b02c39843a1106a4b88ad",
		"fonts" : ["Arial"]
	}
}
```


Для запуска в Хроме нужно изменить исходник - он не понимает type="application/javascript;version=1.7", да и в куки почему-то не записывается. В TBB наоборот не записывается в Storage, возможно оно отключено. Раз уж мы заговорили о совместимости, то скажу, что с совместимостью реализаций ES6 в браузерах просто беда: один не поддерживает генераторные функции со звездой, другой - без звезды, другому нужно написать
```js
"use strict";
```
, да ещё обернуть в анонимную функцию - иначе строгий режим в этом браузере не включится, третяя не поддерживает стрелочные функции (поэтому результатов для неё нет, браузер, не поддерживающий последние стандарты или хотя-бы их черновики не нужен). Лучше себя повёл Firefox, ему требуется только `type="application/javascript;version=1.7"`.
### Chrome 39.0.2171.95
```json
{
	"Impact" : "ca3644b7c3b3df328e01cd65",
	"Courier New" : "1c6c9dd10a5d5bfc8c30ef13",
	"Bookman Old Style" : "f19f6085ebd1849e84df7a64",
	"Consolas" : "01c0b01b4b9cd5241a17e5bd",
	"MS Gothic" : "fbe20d8bed3884f64e27392e",
	"Constantia" : "f5fc044902baf27fe5dbf32c",
	"Calibri" : "563c352880c610c0b6d72e5d",
	"Cambria" : "23ba75fc6063e9b3c56361e2",
	"Wingdings" : "a65dba96053acacee4518c91",
	"Webdings" : "bacd52be46fac7bb0bb344e4",
	"Symbol" : "2323f362e066eaaa6f378187",
	"Ubuntu Mono" : "a6f4eb9ebd660f0fb218ccb7",
	"Inconsolata LGC" : "1ba935514f2187f7c4e8b944",
	"Source Code Pro" : "219562b8e6b7c0a9ef9ae154",
	"Lucida Handwriting" : "0a3ee60ebecbf1db68813fc5",
	"Georgia" : "1b20860fcc7ffa79ee0db0d8",
	"serif" : {
		"d" : "5fca6b682556b887fe5b91c2",
		"fonts" : ["Times New Roman", "Droid Sans", "DejaVu Sans", "Inconsolata", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "5cc93cfaa34d66e6afbf133c",
		"fonts" : ["Arial"]
	}
}
```


## Tor Browser Bundle

Oтпечаток TBB версии A на Win 8.1 совпадает с отпечатком Firefox <acronym title='"Ночная" сборка Firefox'>Nightly</acronym> с включенным e10s.

Перейдём на другой компьютер с Win XP, проверим, что выдаёт там.
### TBB версия A на Win XP
```json
{
	"Impact" : "570243d5a2a1ad9ecb5eeda5",
	"Courier New" : "9dca70bba9b272bab8f54e67",
	"Calibri" : "f5f4ca843390787ff5a58aa5",
	"Wingdings" : "29a0d5b485267d624068b451",
	"Webdings" : "ac90955ff27cddfd470630a7",
	"Symbol" : "dbf4640208e822cdce25b7a5",
	"serif" : {
		"d" : "eb8f5ca8f335bab71807f6c1",
		"fonts" : ["Times New Roman", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Cambria", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "4c574db8e9969363d70d0d25",
		"fonts" : ["Arial"]
	}
}
```


Попробуем другую версию TBB.
### TBB версия B на WinXP при browser.display.use_document_fonts=1 (по умолчанию)
```json
{
	"Impact" : "0a82add690d4353b2730d8ee",
	"Courier New" : "f2f88522221a2cd46c8e0897",
	"Calibri" : "166947a8f38924bff8c20df7",
	"Wingdings" : "71d1ca21e215374130468113",
	"Webdings" : "3507ade160e9610d45fbdfe3",
	"Symbol" : "c5b23ac865d492ae2596d17b",
	"serif" : {
		"d" : "1f0fec66cf90e8ef39df0209",
		"fonts" : ["Times New Roman", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Cambria", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "831ea900aa3285fa5747d31b",
		"fonts" : ["Arial"]
	}
}
```


Отключим шрифты.
### TBB версия B на WinXP при browser.display.use_document_fonts=0 (по умолчанию)
```json
{
	"serif" : {
		"d" : "1f0fec66cf90e8ef39df0209",
		"fonts" : ["Times New Roman", "Arial", "Impact", "Courier New", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Symbol", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "1f0fec66cf90e8ef39df0209",
		"fonts" : []
	}
}
```

Как видно, теперь можно извлечь информацию только о Fallback-шрифте.

Теперь посмотрим, что выдаёт TBB на [<acronym title="The Amnesic Incognito Live System, LiveCD для анонимной работы">TAILS</acronym>](https://tails.boum.org/).
### TBB 4.0.2 Tails (на железе и в виртуалке)
```json
{
	"Times New Roman" : "be2166d015ec4eeb45a0b798",
	"Arial" : "a6e6b634440edc0c369bc2e9",
	"Courier New" : "420ef97e187f6e740f55c365",
	"serif" : {
		"d" : "6b4685a844a9ba975363dba5",
		"fonts" : ["Impact", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Symbol", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "0873da0f46bd2f66345aba5b",
		"fonts" : []
	}
}
```


## Linux и шрифты

Проверим на одном из дистрибутивов Linux.
### IceWeasel 24.7.0 Linux
```json
{
	"Times New Roman" : "862dd27554770f4ac8aee536",
	"Arial" : "2c5b29ec3f4411b8f86f3068",
	"Courier New" : "070dbb80d339e0ccb2347141",
	"Symbol" : "ddc2d84cdddd41003741fe60",
	"Droid Sans" : "cab2b00f609fdfcea99d68d2",
	"DejaVu Sans" : "6d377d1282fdaca428a56943",
	"serif" : {
		"d" : "6a41b7981b6d6caf6087bfe4",
		"fonts" : ["Impact", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Ubuntu Mono", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "9963bcb1199ea5a24f06963a",
		"fonts" : []
	}
}
```


Скопировал несколько шрифтов с винды, устанавливаю и наблюдаю странные вещи: похоже шрифты зависят друг от друга.

Установил Consolas. Изменилось всё.
### IceWeasel 24.7.0 Linux + Consolas
```json
{
	"Times New Roman" : "46def84b497dc4e32cdc380f",
	"Arial" : "b4b350af8fe265ab3458e25c",
	"Courier New" : "b8df5885586d3f4d442c72db",
	"Consolas" : "dddaf0069eb3f16516ab3ecb",
	"Symbol" : "ed43eba4e2a1ff897a2f89c6",
	"Droid Sans" : "e59f70a6030a0531f1f32240",
	"DejaVu Sans" : "028b7d4cf2cc37de25453de0",
	"serif" : {
		"d" : "e97cbca6063cf04ff00f4867",
		"fonts" : ["Impact", "Bookman Old Style", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Ubuntu Mono", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "4f51ffbe1ef1cc3266e3cd19",
		"fonts" : []
	}
}
```


Установил Constantia.
### IceWeasel 24.7.0 Linux + Consolas + Constantia
```json

{
	"Times New Roman" : "46def84b497dc4e32cdc380f",
	"Arial" : "b4b350af8fe265ab3458e25c",
	"Courier New" : "b8df5885586d3f4d442c72db",
	"Consolas" : "dddaf0069eb3f16516ab3ecb",
	"Constantia" : "164bef6a78a302f7b453b555",
	"Symbol" : "ed43eba4e2a1ff897a2f89c6",
	"Droid Sans" : "e59f70a6030a0531f1f32240",
	"DejaVu Sans" : "028b7d4cf2cc37de25453de0",
	"serif" : {
		"d" : "e97cbca6063cf04ff00f4867",
		"fonts" : ["Impact", "Bookman Old Style", "MS Gothic", "Calibri", "Cambria", "Wingdings", "Webdings", "Ubuntu Mono", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "4f51ffbe1ef1cc3266e3cd19",
		"fonts" : []
	}
}
```


Установил кучу шрифтов (кроме Times New Roman и Arial). Изменилось всё, кроме Consolas. Каким-то чудом появилась Inconsolata, хотя я её не устанавливал.
### IceWeasel 24.7.0 Linux + Fonts
```json	
{
	"Times New Roman" : "037992474b8ee5aa4441582b",
	"Arial" : "b07d2897b62ea7bf41549bfe",
	"Impact" : "f2aae8524e25b402319d5e11",
	"Courier New" : "609d743a8ec951ef2526fb46",
	"Bookman Old Style" : "53c1a9e42f585b83b5cec59e",
	"Consolas" : "dddaf0069eb3f16516ab3ecb",
	"MS Gothic" : "272f6635e43dfa0e06c75fb2",
	"Constantia" : "5a1775f2f6a78c97c0f0e1be",
	"Calibri" : "12682e299c0bc5a45be15f8b",
	"Cambria" : "1b6188f0a37c832b916ea57a",
	"Wingdings" : "8e8c860e3d6402cc7395e1b3",
	"Webdings" : "6bf64e72d44f2621b1878cf0",
	"Symbol" : "890aebb687ec397c53ec9909",
	"Droid Sans" : "42a2976bd599ee4a51e89a9d",
	"DejaVu Sans" : "efd4920d05ca755afa95014a",
	"Inconsolata" : "0d614754d96d99f5da9b858d",
	"serif" : {
		"d" : "0e48d7e01ceba457769b6a65",
		"fonts" : ["Ubuntu Mono", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "8f6c32dac2b82d34e5a16954",
		"fonts" : []
	}
}
```


Установил Times New Roman. Изменилось всё...
### IceWeasel 24.7.0 Linux + Fonts + Times New Roman
```json
{
	"Times New Roman" : "1cc93820b8a55ba096c49210",
	"Arial" : "8ec0dbf49590e3e36e791c9f",
	"Impact" : "40da74b4288fb7b08c0b4524",
	"Courier New" : "185eefbb77239792247a60d7",
	"Bookman Old Style" : "ad3239f54fd2492e7291d443",
	"Consolas" : "e8770d7d209f02e63145cfa1",
	"MS Gothic" : "daaef28a5eb5ebc1f252dd98",
	"Constantia" : "b2f3cf465c7768970c5adaf1",
	"Calibri" : "d0a95ea4d0591a0846d34fdd",
	"Cambria" : "7c0669341bdc1a24186506c8",
	"Wingdings" : "20966ffc0d83cbf58f33258a",
	"Webdings" : "56e98312e05137ecca9593a1",
	"Symbol" : "a8b34f82e4198cda66833cba",
	"Droid Sans" : "aae7141159203f7a7a8ce672",
	"DejaVu Sans" : "4004c179a3fdecb0e3bdd120",
	"serif" : {
		"d" : "583e14d3bd28a2a66f7d8eda",
		"fonts" : ["Ubuntu Mono", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "7b9af04baaf04d2492fb7b23",
		"fonts" : []
	}
}
```

Установил Arial. Изменился только он и sans-serif.
### IceWeasel 24.7.0 Linux + Fonts + Times New Roman + Arial
```json
{
	"Times New Roman" : "1cc93820b8a55ba096c49210",
	"Arial" : "f13b1b10bee550cf918a68bd",
	"Impact" : "40da74b4288fb7b08c0b4524",
	"Courier New" : "185eefbb77239792247a60d7",
	"Bookman Old Style" : "ad3239f54fd2492e7291d443",
	"Consolas" : "e8770d7d209f02e63145cfa1",
	"MS Gothic" : "daaef28a5eb5ebc1f252dd98",
	"Constantia" : "b2f3cf465c7768970c5adaf1",
	"Calibri" : "d0a95ea4d0591a0846d34fdd",
	"Cambria" : "7c0669341bdc1a24186506c8",
	"Wingdings" : "20966ffc0d83cbf58f33258a",
	"Webdings" : "56e98312e05137ecca9593a1",
	"Symbol" : "a8b34f82e4198cda66833cba",
	"Droid Sans" : "aae7141159203f7a7a8ce672",
	"DejaVu Sans" : "4004c179a3fdecb0e3bdd120",
	"serif" : {
		"d" : "583e14d3bd28a2a66f7d8eda",
		"fonts" : ["Ubuntu Mono", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "1a5d44fe88e3b2aacc43c0c5",
		"fonts" : []
	}
}
```


Немного оффтопа: в ходе этой части исследования выяснилось, что многие шрифты содержат комментарий с рекомендациями по его использованию. Жаль, что в текстовых редакторах они не показываются по наведению.

# Выводы

*   Модификация метода не позволяет извлекать информацию о железе, но позволяет различать ОС, браузеры и наборы шрифтов (а вместе с ними - и софт, так как некоторые шрифты идут с софтом, например Calibri - с пакетом программ Microsoft Office), в том числе позволяет различить TBB, запущенный в разных окружениях и разные версии TBB, запущенные в одном окружении.
*   Один из шагов, который хотят предпринять разработчики - использовать шрифты, поставляемые вместе с TBB, но это, скорее всего, не поможет, так как в различных системах текст рендерятся по-разному, да и от версии браузера зависит.
*   JavaScript просто необходимо отключать, или хотя-бы отключить шрифты и наиболее опасные API через about:config.
Ни то, ни другое не было сделано по-умолчанию ни в TBB, ни в TAILS, что наводит на определённые мысли, особенно если учесть [источники финансирования Tor Project](https://www.torproject.org/about/sponsors.htmml.en) и [рекомендацию их представителей](https://www.youtube.com/watch?v=pRrFWwA-47U&t=21m39s) делать ```bash curl https://ooni.torproject.org/install.sh | bash``` (никогда так не делайте, в любой момент по этому адресу может оказаться вредоносный скрипт, про HTTPS забудьте - ЦС может выпустить новый сертификат). HTTP Public Key Pinning частично поможет (против смены ЦС, но ничего не мешает существующему ЦС перевыпустить сертификат), но он на момент написания статьи [не был включён](https://www.ssllabs.com/ssltest/analyze.html?d=torproject.org).
Tails ещё простительно - отпечаток не зависит от оборудования, а система у всех одинакова (при одинаковой версии и без <abbr title="Сам Себе Злой Буратино">ССЗБ</abbr>, разумеется).
Отключение фич может детектиться, поэтому отключать нужно у всех сразу. Все идеи постите в [баг-трекер Tor Project](https://trac.torproject.org/). Регистрация не нужна: есть официальный многопользовательский аккаунт с логином cypherpunks и паролем writecode.
*   Отключение JS не поможет, потому что есть ещё <abbr title="Cascade Style Sheet">CSS</abbr> (продвинутая технология настройки отображения элементов, [позволяющая даже реализовывать простенькие игры на ней без единой строчки JS-кода!](http://habrahabr.ru/post/203048/)), с помощью которого также можно извлекать информацию о компьютере, но это затруднит идентификацию, так как через CSS доступно меньше информации, чем через JS, и собрать и отправить на сервер её труднее. Даже если закрыть CSS, найдутся и другие лазейки. Но это ни в коей мере не значит, что его не надо отключать.
*   Отключение JS несовместимо с сегодняшним вебом, нацеленным в первую очередь на зарабатывание денег. Бизнес есть бизнес, если для получения прибыли (или отсутствия смены руководства компании и отбытия его валить лес) необходимо прогнуться под спецслужбы, внестись в открытый или секретный реестр, отслеживать пользователей, анализировать их действия, и распространять вредоносное ПO, то это будет сделано.
Некоторые сайты умышлено сделанны нефункциональными без JS, на некоторых из них (говномыло и втентакле, например) явно написано, что без JS пользоваться сайтом нельзя (что, однако, иногда можно обойти).
Вывод: ожидать от владельцев сайтов реализацию работы без JS не приходится - не в их интересах.

# Дополнительные материалы

1.  [How Unique Is Your Web Browser? Peter Eckersley](https://panopticlick.eff.org/browser-uniqueness.pdf)
2.  [Pixel Perfect: Fingerprinting Canvas in HTML5. Keaton Mowery and Hovav Shacham](http://cseweb.ucsd.edu/~hovav/dist/canvas.pdf)
3.  [Тикет по этому вопросу](https://trac.torproject.org/projects/tor/ticket/13400)
4.  [google("getBoundingClientRect browser fingerprinting")](https://www.google.com/search?num=100&newwindow=1&q=getBoundingClientRect+browser+fingerprinting)
5.  [Fingerprinting defenses in the Tor Browser](https://www.torproject.org/projects/torbrowser/design/#fingerprinting-defenses)
6.  [PoC](https://gist.github.com/KOLANICH/00b9145743d841cff4d7)