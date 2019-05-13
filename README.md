# Поиск: getElement*, querySelector*

Прямая навигация от родителя к потомку удобна, если элементы расположены рядом. А что если нет? Как получить произвольный элемент откуда-то из глубины документа?

Для этого в DOM есть дополнительные методы поиска.

## document.getElementById или просто id

Если элементу назначен специальный атрибут `id`, то можно получить его по переменной с искомым значением `id`.

Мы можем использовать его для немедленного доступа к элементу, где бы он не находился:

```html run
<div id="*!*elem*/!*">
  <div id="*!*elem-content*/!*">Элемент</div>
</div>

<script>
  alert(elem); // DOM-элемент с id="elem"
  alert(window.elem); // accessing global variable like this also works

  // for elem-content things are a bit more complex
  // that has a dash inside, so it can't be a variable name
  alert(window['elem-content']); // ...в имени дефис, поэтому через [...]
</script>
```

Это поведение соответствует [стандарту](http://www.whatwg.org/specs/web-apps/current-work/#dom-window-nameditem), но поддерживается в основном для совместимости, как осколок далёкого прошлого. Браузер пытается помочь нам, смешивая пространства имён JS и DOM, но при этом возможны конфликты, неочевидно откуда возьмется переменная. Also, when we look in JS and don't have HTML in view, it's not obvious where the variable comes from.

Если мы объявляем переменную с тем же именем, то она будет иметь приоритет:

```html run untrusted height=0
<div id="elem"></div>

<script>
  let elem = 5;

  alert(elem); //переменная переопределяет элемент
</script>
```

Лучшая альтернатива - использовать специальный метод `document.getElementById(id)`.

Например:

```html run
<div id="elem">
  <div id="elem-content">Element</div>
</div>

<script>
*!*
  let elem = document.getElementById('elem');
*/!*

  elem.style.background = 'red';
</script>
```
Далее в примерах мы часто будем использовать прямое обращение через `id`, но это только для краткости. В реальных проектах предпочтителен метод `document.getElementById`.

```smart header="Должен остаться только один"
Значение `id` должно быть уникальным. В документе может быть только один элемент с данным `id`.

Если в документе есть несколько элементов с одинаковым значением `id`, то поведение методов поиска непредсказуемо. Браузер может вернуть любой из них случайным образом. Поэтому, пожалуйста, придерживайтесь правила сохранения уникальности `id`.
```

```warn header="Only `document.getElementById`, not `anyNode.getElementById`"
The method `getElementById` that can be called only on `document` object. It looks for the given `id` in the whole document.
```

## querySelectorAll [#querySelectorAll]

Самый универсальный метод `elem.querySelectorAll(css)` возвращает все элементы внутри `elem` удовлетворяющие CSS-селектору.

Следующий запрос получает все элементы `<li>`, которые являются последними потомками в `<ul>`:

```html run
<ul>
  <li>Этот</li>
  <li>тест</li>
</ul>
<ul>
  <li>полностью</li>
  <li>пройден</li>
</ul>
<script>
*!*
  let elements = document.querySelectorAll('ul > li:last-child');
*/!*

  for (let elem of elements) {
    alert(elem.innerHTML); // "тест", "пройден"
  }
</script>
```

This method is indeed powerful, because any CSS selector can be used.

```smart header="Псевдо-классы тоже работают"
Псевдо-классы в CSS-селекторе, в частности `:hover` и `:active`, также поддерживаются. Например, `document.querySelectorAll(':hover')` вернёт коллекцию(в порядке вложенности) из текущих элементов под курсором мыши.
```

## querySelector [#querySelector]

Метод `elem.querySelector(css)` возвращает первый элемент,соответствующий данному CSS-селектору.

Иначе говоря, результат такой же, как при `elem.querySelectorAll(css)[0]`, но в последнем вызове сначала ищутся *все* элементы, а потом берётся первый, а `elem.querySelector` ищется только первый, то есть он эффективнее.

## matches
Предыдущие методы искали по DOM.

Метод [elem.matches(css)](http://dom.spec.whatwg.org/#dom-element-matches) ничего не ищет, а проверяет, удовлетворяет ли `elem` CSS-селектору, и возвращает `true` или `false`.

Этот метод удобен, когда мы перебираем элементы(например в массиве) и пытаемся отфильтровать те из них, которые нас интересуют.

Например:

```html run
<a href="http://example.com/file.zip">...</a>
<a href="http://ya.ru">...</a>

<script>
  // can be any collection instead of document.body.children
  for (let elem of document.body.children) {
*!*
    if (elem.matches('a[href$="zip"]')) {
*/!*
      alert("The archive reference: " + elem.href );
    }
  }
</script>
```

## closest

*Предками* элемента являются: родитель, родитель родителя, его родитель и так далее. Вместе предки образуют цепочку иерархии.

Метод `elem.closest(css)` ищет ближайший элемент по иерархии DOM, который соответствует CSS-селектору. Сам элемент тоже включается в поиск.

Другими словами, метод `closest`поднимается вверх от элемента и проверяет каждого из родителей. Если он соответствует селектору, поиск прекращается и предок возвращается.

Например:

```html run
<h1>Contents</h1>

<div class="contents">
  <ul class="book">
    <li class="chapter">Chapter 1</li>
    <li class="chapter">Chapter 1</li>
  </ul>
</div>

<script>
  let chapter = document.querySelector('.chapter'); // LI

  alert(chapter.closest('.book')); // UL
  alert(chapter.closest('.contents')); // DIV

  alert(chapter.closest('h1')); // null (because h1 is not an ancestor)
</script>
```

## getElementsBy*

Существуют также другие методы поиска узлов по тегу, классу, и так далее.

На данный момент, они скорее историческиthey are mostly history, так как `querySelector` более чем эффективен.

Здесь мы рассмотрим их для полноты картины, так же вы можете встретить их в ранних скриптах.

- `elem.getElementsByTagName(tag)` ищет элементы с данным тегом и возращает их коллекцию. Передав `"*"` вместо тега можно получить всех потомков.
- `elem.getElementsByClassName(className)` возвращает элементы, которые имеют данный CSS-класс. Элементы могут иметь и другие классы.
- `document.getElementsByName(name)` возвращает элементы с заданным `name` атрибута, для всего документа. Очень редко используется.

Например:
```js
// получить все элементы `div` в документе
let divs = document.getElementsByTagName('div');
```

Давайте найдем все `input` в таблице:

```html run height=50
<table id="table">
  <tr>
    <td>Ваш возраст:</td>

    <td>
      <label>
        <input type="radio" name="age" value="young" checked> младше 18
      </label>
      <label>
        <input type="radio" name="age" value="mature"> от 18 до 50
      </label>
      <label>
        <input type="radio" name="age" value="senior"> старше 60
      </label>
    </td>
  </tr>
</table>

<script>
*!*
  let inputs = table.getElementsByTagName('input');
*/!*

  for (let input of inputs) {
    alert( input.value + ': ' + input.checked );
  }
</script>
```

```warn header="Не забываем про букву `\"s\"`!"
Одна из самых частых ошибок начинающих разработчиков(впрочем, иногда и не только)  - это забыть букву `"s"`. То есть пробовать вызывать метод `getElementByTagName` вместо <code>getElement<b>s</b>ByTagName</code>.

Буква `"s"` отсутствует в названии метода `getElementById`, так как в данном случае вызывает один элемент. Но `getElementsByTagName` вернет список элементов, поэтому `"s"` обязательна.
```

````warn header="Возвращает коллекцию, а не элемент!"
Другая распространенная ошибка - написать:

```js
// не работает
document.getElementsByTagName('input').value = 5;
```

Это не сработает, так как вместо элемента присваивают значение *коллекции*.

Нужно перебрать коллекцию в цикле или получить элемент по номеру и уже ему присваивать значение, например так:

```js
// работает (if there's an input)
document.getElementsByTagName('input')[0].value = 5;
```
````

Ищем `.article` элементы:

```html run height=50
<form name="my-form">
  <div class="article">Article</div>
  <div class="long article">Long article</div>
</form>

<script>
  // ищем по имени атрибута
  let form = document.getElementsByName('my-form')[0];

  // ищем по классу внутри `form`
  let articles = form.getElementsByClassName('article');
  alert(articles.length); // 2, находим два элемента с классом "article"
</script>
```

## "Живые" коллекции

Все методы `"getElementsBy*"` возвращают *живую* коллекцию. Такие коллекции всегда отражают текущее состояние документа и автоматически обновляются при его изменении.

В приведенном ниже примере есть два скрипта.

1. Первый создает ссылку на коллекцию `<div>`. На этот момент ее длина равна `1`.
2. Второй скрипт запускается после того, как браузер встречает еще один `<div>`, теперь ее длина `2`.

```html run
<div>First div</div>

<script>
  let divs = document.getElementsByTagName('div');
  alert(divs.length); // 1
</script>

<div>Second div</div>

<script>
*!*
  alert(divs.length); // 2
*/!*
</script>
```

Напротив, `querySelectorAll` возвращает *статическую* коллекцию. Это как фиксированный массив элементовIt's like a fixed array of elements.

Если мы будем использовать его в примере вышеIf we use it instead, то оба скрипта вернут длину равную then both scripts output `1`:


```html run
<div>First div</div>

<script>
  let divs = document.querySelectorAll('div');
  alert(divs.length); // 1
</script>

<div>Second div</div>

<script>
*!*
  alert(divs.length); // 1
*/!*
</script>
```

Now we can easily see the difference. The static collection did not increase after the appearance of a new `div` in the document.

Here we used separate scripts to illustrate how the element addition affects the collection, but any DOM manipulations affect them. Soon we'll see more of them.

## Summary

Есть 6 основных методов поиска элементов в DOM:

<table>
<thead>
<tr>
<td>Метод</td>
<td>Ищет по...</td>
<td>Ищет внутри элемента?</td>
<td>Возвращает "живую" коллекцию?</td>
</tr>
</thead>
<tbody>
<tr>
<td><code>querySelector</code></td>
<td>CSS-selector</td>
<td>✔</td>
<td>-</td>
</tr>
<tr>
<td><code>querySelectorAll</code></td>
<td>CSS-selector</td>
<td>✔</td>
<td>-</td>
</tr>
<tr>
<td><code>getElementById</code></td>
<td><code>id</code></td>
<td>-</td>
<td>-</td>
</tr>
<tr>
<td><code>getElementsByName</code></td>
<td><code>name</code></td>
<td>-</td>
<td>✔</td>
</tr>
<tr>
<td><code>getElementsByTagName</code></td>
<td>tag or <code>'*'</code></td>
<td>✔</td>
<td>✔</td>
</tr>
<tr>
<td><code>getElementsByClassName</code></td>
<td>class</td>
<td>✔</td>
<td>✔</td>
</tr>
</tbody>
</table>
Безусловно, наиболее часто используемыми в настоящее время являются методы `querySelector` and `querySelectorAll`, но и методы  `getElementBy*` могут быть полезны в отдельных случаях, а также встречаются в ранних скриптах.

Кроме того:

- Есть метод `elem.matches(css)`, который проверяет, удовлетворяет ли элемент CSS-селектору.
- Метод `elem.closest(css)` ищет ближайшего по иерархии предка, соответствующему данному CSS-селектору. Сам элемент также включён в поиск.

And let's mention one more method here to check for the child-parent relationship:
-  `elemA.contains(elemB)` returns true if `elemB` is inside `elemA` (a descendant of `elemA`) or when `elemA==elemB`.
