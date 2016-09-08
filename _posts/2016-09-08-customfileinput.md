---
title: Стилизация и кастомизация File Inputs
description: Небольшой туторила как кастомизировать тег input &lt;type="file"&gt; с учетом семантики и доступности, используя элемент label и немного JS.
---

# Стилизация и кастомизация File Inputs

## Правильный путь

Небольшой туторила как кастомизировать тег input &lt;type="file"&gt; с учетом семантики и доступности, используя элемент label и немного JS.

Изменение внешнего вида инпутов, как правило, не вызывает трудностей, но file input отличается от остальных.
Это связано с безопасностью и с тем, что каждый браузер по своему отображает этот элемент.
Проблема в том, чтои на это почти нельзя повлиять. Но если очень хочется...

## Demo
Для нетерпеливых сразу ссылка на [DEMO](/tutorials/custom-fileInputs/)

## Проблемы
С помощью JS мы не можем имитировать клик на file input. Вот что говорится об этом в описании спецификации к DOM:

	Simulate a mouse-click. For INPUT elements whose type attribute has one of the following values: “button”, “checkbox”,
	“radio”, “reset”, or “submit”.

	- No Parameters
	- No Return Value
	- No Exceptions

То есть методом click мы можем тригерить клик почти на всех типах инпутов, но не на input file. Это сделано чтобы обезопасить клиента: иначе вебстраница могла бы получать любые файлы с компьютера клиента. Хотя с другой стороны, по клику вызывается только окно выбора файла. Так или иначе, в MDN Firefox этот факт обозначен как баг.

Решение есть. И не одно.

Одно из распространенных решений: создается инпут с нулевой прозрачностью и ставится поверх кнопки или изображения, которые представляют собой кнопку выбора файла.

Основная трудность в следующем.  Мы не можем свободно влиять на размеры кнопок «обзор», чтобы подогнать input под размер перекрываемой картинки. В Firefox вообще не можем изменить внешний вид файл-инпута средствами CSS (кроме высоты). То есть задача заключается определении оптимального размера перекрываемой картинки, чтобы минимальное количество пикселей было не кликабельно, а пустые области не реагировали на клик.

Но есть способ лучше и правильнее, на мой взгляд. Это кастомизация тега label и тригеринг клика через него.

И так, минимальный HTML код для кастомизации file input тега:

```html
<input type="file" name="file" id="file" class="inputfile" />
<label for="file">Choose a file</label>
```

![](http://codropspz.tympanus.netdna-cdn.com/codrops/wp-content/uploads/2015/09/smart-custom-file-input-1.gif)

Как видите триггер на клик срабатывает через наш label, но мы видим так же наш тег input.

### Скрываем input

```css
.inputfile {
	width: 0.1px;
	height: 0.1px;
	opacity: 0;
	overflow: hidden;
	position: absolute;
	z-index: -1;
}
```

### Стилизуем label

```css
.inputfile + label {
    font-size: 1.25em;
    font-weight: 700;
    color: white;
    background-color: black;
    display: inline-block;
}

.inputfile:focus + label,
.inputfile + label:hover {
    background-color: red;
}
```

### Доступность
Чтобы пользовтель понимал что это активный элемент и именно его нужно кликнуть, добавим курсор "указатель":
```css
.inputfile + label {
	cursor: pointer; /* "hand" cursor */
}
```

**До**
![](http://codropspz.tympanus.netdna-cdn.com/codrops/wp-content/uploads/2015/09/smart-custom-file-input-2.gif)

**После**
![](http://codropspz.tympanus.netdna-cdn.com/codrops/wp-content/uploads/2015/09/smart-custom-file-input-3.gif)

### Навигация клавиатурой
Добавим возмрожность навигации клавиатурой:

```css
.inputfile:focus + label {
	outline: 1px dotted #000;
	outline: -webkit-focus-ring-color auto 5px;
}
```

-webkit-focus-ring-color auto 5px - это небольшой трюк для эмуляции дефолтного стиля выбранного элемента в браузерах Chrome, Opera и Safari. 

![](http://codropspz.tympanus.netdna-cdn.com/codrops/wp-content/uploads/2015/09/smart-custom-file-input-4.gif)

### Доступность в Touch устройствах

```html
<label for="file"><strong>Choose a file</strong></label>
```

```css
.inputfile + label * {
	pointer-events: none;
}
```

## Улучшаем с помощью JS

```html
<input type="file" name="file" id="file" class="inputfile" data-multiple-caption="{count} files selected" multiple />
```

Напишем небольшой контроллер, который будет менять представление в зависимости от режима.

```js
var inputs = document.querySelectorAll('.inputfile');
Array.prototype.forEach.call(inputs, function(input){
	var label	 = input.nextElementSibling,
		labelVal = label.innerHTML;

	input.addEventListener('change', function(e){
		var fileName = '';
		if( this.files && this.files.length > 1 )
			fileName = ( this.getAttribute( 'data-multiple-caption' ) || '' ).replace( '{count}', this.files.length );
		else
			fileName = e.target.value.split( '\\' ).pop();

		if( fileName )
			label.querySelector( 'span' ).innerHTML = fileName;
		else
			label.innerHTML = labelVal;
	});
});
```

![](http://codropspz.tympanus.netdna-cdn.com/codrops/wp-content/uploads/2015/09/smart-custom-file-input-5.gif)

### Фиксим баги в Firefox

```js
input.addEventListener( 'focus', function(){ input.classList.add( 'has-focus' ); });
input.addEventListener( 'blur', function(){ input.classList.remove( 'has-focus' ); });
```

```css
.inputfile:focus + label,
.inputfile.has-focus + label {
    outline: 1px dotted #000;
    outline: -webkit-focus-ring-color auto 5px;
}
```

[DEMO](/tutorials/custom-fileInputs/)


Вольный краткий перевод статьи [Styling & Customizing File Inputs the Smart Way](http://tympanus.net/codrops/2015/09/15/styling-customizing-file-inputs-smart-way/)
