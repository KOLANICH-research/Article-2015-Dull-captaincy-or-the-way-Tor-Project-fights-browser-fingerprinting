# Унылое капитанство или как Tor Project борется с browser fingerprinting

В [статье](http://cseweb.ucsd.edu/~hovav/dist/canvas.pdf) известного в определённых кругах [Hovav Shacam](http://cseweb.ucsd.edu/~hovav/papers.html) и [Keaton Mowery](http://cseweb.ucsd.edu/~kmowery/) от 2012 года был описан новый на тот момент метод [генерации идентификатора браузера и системы прямо из JavaScript](https://panopticlick.eff.org/browser-uniqueness.pdf) - Canvas Fingerprinting. Метод позволял различать, в том числе, оборудование.

# Статья

Суть метода в том, что на разных системах в разных браузерах по-разному рендерится текст (и не только текст), так как за это отвечает много различных компонентов на различных уровнях, которые могут иметь разные настройки для компонентов нижележащего уровня.

1.  Могут быть банально разные (с разным набором символов, с немного разными глифами, с разными лигатурами и кернингом…) шрифты.
2.  Разные параметры вызова библиотечных функций в разных браузерах.
3.  Разные версии [libfreetype](http://www.freetype.org/) и прочих библиотек рендеринга.
4.  Разные реализации в ОС и разные настройки ОС (например, [разные версии <acronym title="Технология субпиксельного сглаживания шрифта, из-за чего шрифты выглядят чётче">ClearType</acronym>](http://habrahabr.ru/post/247703/) и различное разрешение экрана).
5.  Разные графические драйвера.
6.  Разное графическое железо.

В статье описан способ с использованием <abbr title="Application Programming Interface">API</abbr> [getImageData](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D.getImageData), который возвращает картинку попиксельно. Также рассмотрено использование <acronym title="OpenGL в веб-страницах">WebGL</acronym> : на разных системах и 3D-сцены рендерятся по-разному.

В статье была дана рекомендация использовать чисто софтовый рендеринг, без использования компонентов ОС и прочего ПО, установленного на ПК, зашумление и прочие методики защиты. В Tor Project возможность создания отпечатков залатали, спрашивая разрешения на getImageData и заменяя случайные шрифты на fallback-шрифт (поэтому мы делаем замеры 10 раз).

# Defective by design

Есть другие API, текущие информацией, которые почему-то не защищены. Например, API измерения текста. Если текст рендерится по-разному, значит и размеры должны быть чуточку разные. Проверим эту гипотезу, используя API [measureText](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D.measureText) для того же холста и API [getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element.getBoundingClientRect) из DOM.

<acronym title="git clone https://KOLANICH@github.com/KOLANICH/Article-2015-Dull-captaincy-or-the-way-Tor-Project-fights-browser-fingerprinting">Склонируйте</acronym> [репозиторий](https://github.com/KOLANICH/Article-2015-Dull-captaincy-or-the-way-Tor-Project-fights-browser-fingerprinting), запустите у себя на компе последний (или не последний) [<abbr title="TOR Browser Bundle, набор программ для анонимизированного сёрфинга">TBB</acronym>](https://www.torproject.org/download/download.html.en), откройте в нём HTML-файл из репозитория, **отпостите результат в комментах или как pr**. Ещё можно зайти в [фиддл](http://jsfiddle.net/fyw4qmdg/5/) или [в полноэкранный фиддл](http://fiddle.jshell.net/fyw4qmdg/5/show/). В принципе можно было сделать не хеши, это дало бы больше простора для дата-майнинга, но даже в сжатом виде полный набор информации о каждом шрифте занимает очень много места. Поэтому обойдёмся хешами.

# Результаты

## Различные браузеры

Сначала запустим на Firefox стабильной и ночной версий.

### Firefox 35.0 и Firefox Nightly 37.0a1 @ Windows 8.1
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

Дают одинаковые отпечатки. Теперь проверем на семёрке

### Firefox 35.0 @ Windows 7
```json
{
	"Impact" : "2e32f50e863bc9a085666248",
	"Courier New" : "aff48e3be26cc0dc507cff3c",
	"Bookman Old Style" : "6e1b8cc3a60c648c763ccb43",
	"Consolas" : "156930f3a699c3d4744de1b5",
	"MS Gothic" : "4d3972114ab839313eb6b86f",
	"Constantia" : "b22efe989907bbb968cd028d",
	"Calibri" : "abb1c5868a80896c0c65ee13",
	"Cambria" : "f364822f7f8f4fd170f2e1e0",
	"Wingdings" : "66e3a81068bbba59e439235e",
	"Webdings" : "c22c781324f8365781b359e6",
	"Symbol" : "7073f323f33a6861b2368641",
	"Lucida Handwriting" : "7953cc9b88bac85bb5637189",
	"Georgia" : "8046fa72d901874f1d44c4f5",
	"serif" : {
		"d" : "4e9edc58576ac6e25485aeda",
		"fonts" : ["Times New Roman", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "0c5aeb3d64c4564e6cfe613f",
		"fonts" : ["Arial"]
	}
}
```

Как видно, отпечатки различаются.

А теперь поиграемся с параметрами. Игра с параметрами ClearType ничего не дала, как и включение DirectWrite, зато включение e10s изменило отпечаток и полностью порушило рендеринг (все страницы рендерились в чёрный прямоугольник).

### Firefox Nightly 37.0a1 c e10s включённым @ Windows 8.1
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


Для запуска в Хроме нужно изменить исходник - он не понимает ```type="application/javascript;version=1.7"```, да и в куки почему-то не записывается. В TBB наоборот не записывается в Storage, возможно оно отключено. Раз уж мы заговорили о совместимости, то скажу, что с совместимостью реализаций ES6 в браузерах просто беда: один не поддерживает генераторные функции со звездой, другой - без звезды, другому нужно написать ```"use strict";``` да ещё обернуть в анонимную функцию - иначе строгий режим в этом браузере не включится, а вместе с ним и ES6, третяя не поддерживает стрелочные функции (поэтому результатов для неё нет, браузер, не поддерживающий последние стандарты или хотя-бы их черновики не нужен). Лучше себя повёл Firefox, ему требуется только ```type="application/javascript;version=1.7"```.

### Chrome 39.0.2171.95 @ Windows 8.1
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

Также @starius [опубликовал свои результаты](#comment_8227810).

### Iceweasel @ Linux by starius
```json
{
	"Times New Roman" : "c2c91d5b3c4fecd9109afe0e",
	"Arial" : "4917211a76ddf69db033e125",
	"Courier New" : "eb211de3b75234ea90a50c3f",
	"Symbol" : "709ab9f882b1808b323e7d09",
	"Droid Sans" : "fbc25f5e038a28b94454fa13",
	"DejaVu Sans" : "c0bf2bce71e4313758d1aba8",
	"serif" : {
		"d" : "5daa940a38e3b137916aadcb",
		"fonts" : ["Impact", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Ubuntu Mono", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "46a9a2d351881662502ed793",
		"fonts" : []
	}
}
```

Не забудем Android

### Firefox @ CyanogenMod 11 Nightly
```json
{
	"serif" : {
		"d" : "56c4b3aba5c77853c4a1ed56",
		"fonts" : []
	},
	"sans-serif" : {
		"d" : "dec3163b8630ebcf6d7356b6",
		"fonts" : ["Times New Roman", "Arial", "Impact", "Courier New", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Symbol", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	}
}
```

# Tor Browser Bundle

Oтпечаток TBB версии A на Win 8.1 ЧАСТИЧНО совпадает с отпечатком Firefox Nightly с включенным e10s. В частности не совпадают отпечатки для Courier New, Georgia и System.

### TBB версия A на Win 8.1
```json
{
	"Impact" : "004c408dbbbdb9c9910c4352",
	"Courier New" : "c061e3c7731e6d18b2b1aee2",
	"Bookman Old Style" : "fe957ed9b9d3d352564b832d",
	"Consolas" : "74831d1303efc9733ab93cb1",
	"MS Gothic" : "ed7dbb5235608f4c2b38157f",
	"Constantia" : "c70150a0ed768f5e58196f96",
	"Calibri" : "c18f30a9cc711a1e67a5eb5e",
	"Cambria" : "0013175b346f8ff058a53e16",
	"Wingdings" : "547a5121752201d862bea0bc",
	"Webdings" : "a98b6f18a2464e8f4ac0c06b",
	"Symbol" : "be2745f60719b4350900e044",
	"Ubuntu Mono" : "7456b9713327556503f5f3df",
	"Inconsolata LGC" : "49b5231c4cf98df7b253feba",
	"Source Code Pro" : "12e997bcc449b894550266f9",
	"Lucida Handwriting" : "d293b8185d9fdb4060403a73",
	"Georgia" : "56acda38697db9e55593d004",
	"System" : "096e5dd0fd64151c7c45633c",
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

[Промежуточные результаты](#comment_8227810) @starius

### TBB версия ? @ Linux by starius
```json
{
	"it" : 1,
	"fonts" : {
		"Times New Roman" : "c2c91d5b3c4fecd9109afe0e",
		"Arial" : "4917211a76ddf69db033e125",
		"Courier New" : "eb211de3b75234ea90a50c3f"
	},
	"fontFingerprintingTotalTime" : 2404.465604000002,
	"serifHash" : "5daa940a38e3b137916aadcb",
	"sansHash" : "46a9a2d351881662502ed793"
}
```
Видно, что отпечатки для шрифтов совпадают с теми, что были для IceWeasel.

# Системы для анонимной работы

Как мы видим, в зависимости от окружения отпечатки разные. Один из способов решения этой и целого ряда других проблем - использовать дистрибутивы Linux, специально предназначенные для анонимной работы.

Рассмотрим [TAILS](https://tails.boum.org/) и [Whonix](https://www.whonix.org/).

Посмотрим, что выдаёт TBB на TAILS.

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

Теперь посмотрим, что выдаёт IceWeasel на Whonix

### IceWeasel Whonix
```json
{
	"DejaVu Sans" : "4fd7fb4babfa17ccd9755508",
	"serif" : {
		"d" : "6d0bcd365a1eaade9320ed12",
		"fonts" : ["Times New Roman", "Arial", "Impact";
			"Courier New", "Bookman Old Style", "Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Symbol", "Ubuntu Mono", "Droid Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "3a84298716a4bca4cb4f4d0f",
		"fonts" : []
	}
}
```

И наконец TBB на Whonix

### TBB Whonix
```json
{
	"serif" : {
		"d" : "6d0bcd365a1eaade9320ed12",
		"fonts" : ["Times New Roman", "Arial", "Impact", "Courier New", "Bookman Old Style";
			"Consolas", "MS Gothic", "Constantia", "Calibri", "Cambria", "Wingdings", "Webdings", "Symbol", "Ubuntu Mono", "Droid Sans", "DejaVu Sans", "Inconsolata", "Inconsolata LGC", "Source Code Pro", "Lucida Handwriting", "Georgia", "System", "vgaoem"]
	},
	"sans-serif" : {
		"d" : "3a84298716a4bca4cb4f4d0f",
		"fonts" : []
	}
}
```

Совпадает с [промежуточными результатами](#comment_8227810) @starius, что есть хорошо. Отпечатки TBB и IceWeasel почти совпадают: в IceWeasel доступно на 1 шрифт больше. Наблюдается различие между отпечатками TAILS, Whonix и TBB на разных системах, что есть плохо. То есть можно различать операционки.

# Linux и шрифты

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
Не совпадает с [результатами](#comment_8227810) @starius.


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


Установил Times New Roman. Изменилось всё… Inconsolata исчезла…

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

# In The Wild

Уже после публикации этой статьи пользователь gk с баг-трекера Tor Project [подсказал интересную статью](https://trac.torproject.org/projects/tor/ticket/14310#comment:3) [PriVaricator: Deceiving Fingerprinters with Little White Lies. Nikiforakis N., Joosen W., Livshits B.](http://research.microsoft.com/pubs/209989/tr1.pdf) о рандомизаци отпечатков. В ней упомянуто несколько отслеживающих скриптов, в частности как минимум [Bluecava BCAC5.js](http://ds.bluecava.com/v50/AC/BCAC5.js) и [tagv22.pkmin.js](http://tags.master-perf-tools.com/V20test/tagv22.pkmin.js) (реализации одинаковы) использует похожую технику: измеряют длину заданног сообщения для предопределённого списка шрифтов, из чего получают список шрифтов (фингерпринты на сервер не отправляются). Таблицу с прямыми адресами скриптов фингерпринтинга можно найти в [этой статье](https://www.cosic.esat.kuleuven.be/publications/article-2334.pdf).

# Немного оффтопа на тему рандомизации

В статье предлагается способ рандомизировать значения, поддающиеся фингерпринтингу, с целью предотвращения отслеживания, но мы уже на примере TBB видели, насколько успешна такая рандомизация. Я считаю, что рандомизация для каждого сайта бесполезна, так как её всегда можно обойти. Если, как предлагается в статье, используется кэш случайных значений (случайные значения сохраняются и при повторном запуске фингерпринтера рандомизированные значения будут теми же для этого сайта)

*   его опять можно обойти, например внедрив айфрейм с другого домена, но принадлежащего тому же сервису; в случае canvas можно нарисовать такую же картинку на другой картинке, шум будет другим, и его можно будет отделить.
*   его можно использовать как идентификатор;
*   его можно использовать для DoS-атаки, заставив браузер генерировать случайные значения для кеша и класть их туда до бесконечности. В качестве меры противодействия можно ограничить объём кеша, после чего дропнуть кеш и выдавать ложные значения странице как зловредной

Использовать кэш, единый для всех сайтов, и меняющийся только при новой сессии - один из вариантов. Это позволит отслеживать в пределах сессии, но не позволит создать идентификатор, сохраняющийся между сессиями.

# Выводы

*   Модификация метода не позволяет извлекать информацию о железе, но позволяет различать ОС, браузеры и наборы шрифтов (а вместе с ними - и софт, так как некоторые шрифты идут с софтом, например Calibri - с пакетом программ Microsoft Office), в том числе позволяет различить TBB, запущенный в разных окружениях и разные версии TBB, запущенные в одном окружении.
*   Один из шагов, который хотят предпринять разработчики - использовать шрифты, поставляемые вместе с TBB, но это, скорее всего, не поможет, так как в различных системах текст рендерятся по-разному, да и от версии браузера зависит.
*   JavaScript просто необходимо отключать, или хотя-бы отключить шрифты и наиболее опасные API через about:config. Ни то, ни другое не было сделано по-умолчанию ни в TBB, ни в TAILS, что наводит на определённые мысли, особенно если учесть [источники финансирования Tor Project](https://www.torproject.org/about/sponsors.htmml.en) и [рекомендацию их представителей](https://www.youtube.com/watch?v=pRrFWwA-47U&t=21m39s) делать ```bash curl https://ooni.torproject.org/install.sh | bash``` (никогда так не делайте, в любой момент по этому адресу может оказаться вредоносный скрипт). HTTP Public Key Pinning частично поможет (против смены ЦС, но ничего не мешает существующему ЦС перевыпустить сертификат), но он на момент написания статьи [не был включён](https://www.ssllabs.com/ssltest/analyze.html?d=torproject.org). Tails ещё простительно - отпечаток не зависит от оборудования, а система у всех одинакова (при одинаковой версии и без <abbr title="Сам Себе Злой Буратино">ССЗБ</abbr>, разумеется). Отключение фич может детектиться, поэтому отключать нужно у всех сразу. Все идеи постите в [баг-трекер Tor Project](https://trac.torproject.org/). Регистрация не нужна: есть официальный многопользовательский аккаунт с логином cypherpunks и паролем writecode.
*   Отключение JS не поможет, потому что есть ещё <abbr title="Cascade Style Sheet">CSS</abbr> (продвинутая технология настройки отображения элементов, [позволяющая даже реализовывать простенькие игры на ней без единой строчки JS-кода!](http://habrahabr.ru/post/203048/)), с помощью которого также можно извлекать информацию о компьютере, но это затруднит идентификацию, так как через CSS доступно меньше информации, чем через JS, и собрать и отправить на сервер её труднее. Даже если закрыть CSS, найдутся и другие лазейки. Так что не думаю, что отключение CSS вам сильно поможет.
*   Есть острая необходимость в [стандарте на поведение анонимных браузеров](https://trac.torproject.org/projects/tor/ticket/14310)
*   Отключение JS несовместимо с сегодняшним вебом, нацеленным в первую очередь на зарабатывание денег.
Бизнес есть бизнес, если для получения прибыли (или отсутствия смены руководства компании и отбытия его валить лес) необходимо прогнуться под спецслужбы, внестись в открытый или секретный реестр, отслеживать пользователей, анализировать их действия, и распространять вредоносное ПО, то это будет сделано.
Некоторые сайты умышлено сделаны нефункциональными без JS, на некоторых из них (говномыло и втентакле, например) явно написано, что без JS пользоваться сайтом нельзя (что, однако, иногда можно обойти).
**Вывод:** ожидать от владельцев сайтов реализацию работы без JS не приходится - не в их интересах.

# Дополнительные материалы

1.  [How Unique Is Your Web Browser? Peter Eckersley](https://panopticlick.eff.org/browser-uniqueness.pdf)
2.  [Pixel Perfect: Fingerprinting Canvas in HTML5. Keaton Mowery and Hovav Shacham](http://cseweb.ucsd.edu/~hovav/dist/canvas.pdf)
3.  [User Tracking on the Web via Cross-Browser Fingerprinting. K. Boda, Á. M. Földes, G. Gy. Gulyás, and S. Imre](http://pet-portal.eu/files/articles/2011/fingerprinting/cross-browser_fingerprinting.pdf)
4.  [FPDetective: Dusting the Web for Fingerprinters. G. Acar, M. Juarez, N. Nikiforakis, C. Diaz, S. Gürses, F. Piessens and B. Preneel.](https://www.cosic.esat.kuleuven.be/publications/article-2334.pdf)
5.  [PriVaricator: Deceiving Fingerprinters with Little White Lies. Nikiforakis N., Joosen W. and Livshits B.](http://research.microsoft.com/pubs/209989/tr1.pdf)
6.  [Тикет по этому вопросу](https://trac.torproject.org/projects/tor/ticket/13400)
7.  [google("getBoundingClientRect browser fingerprinting")](https://www.google.com/search?num=100&newwindow=1&q=getBoundingClientRect+browser+fingerprinting)
8.  [Fingerprinting defenses in the Tor Browser](https://www.torproject.org/projects/torbrowser/design/#fingerprinting-defenses)
9.  [PoC](https://KOLANICH@github.com/KOLANICH/Article-2015-Dull-captaincy-or-the-way-Tor-Project-fights-browser-fingerprinting)
10. [фиддл](http://jsfiddle.net/fyw4qmdg/5/)

# Обновления

1.  Добавил ссылки на фиддл.
2.  Добавил фингерпринты для Whonyx и Firefox @ android и результаты от @starius.
3.  Добавил ссылки на статьи и раздел In The Wild
4.  Добавлена английская версия, Папки с результатами и пофикшен текст.

