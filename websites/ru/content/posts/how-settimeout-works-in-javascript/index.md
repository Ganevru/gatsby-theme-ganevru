---
title: Как в JavaScript работает `setTimeout` и что происходит если он установлен на 0?
tags: ['статья']
date: 2020-02-14
cover: ./how-settimeout-works-in-javascript.jpg
---

Чтобы разобраться в том как работает `setTimeout` и функции которые вызываются в нем, нужно разобраться в том как вообще работает JavaScript. Разобравшись в этом, вы станете куда увереннее пользоваться джаваскриптом.

Простейший пример `setTimeout`:

```javascript
setTimeout(function() {
  console.log('какой-то обратный вызов');
}, 0);
```

**`setTimeout(коллбек, задержка)`** означает "вызови обратный вызов (коллбек, это может быть какая либо функция, в нашем случае это функция с `console.log("какой-то обратный вызов")`) после заданной задержки". Но что происходит если мы задаем задержку в **`0`**? Получается, он должен вызвать коллбек сразу же? Может показаться что да, но все сложнее.

Подумайте, что сделает этот код (к слову, подобные вопросы любят задавать на интервью):

```javascript
setTimeout(function() {
  console.log('setTimeout заданный в 0');
}, 0);
console.log('раз');
console.log('два');
console.log('три');
```

В какой последовательности будут отображены эти сообщения?

Отобразит он вот это:

```
раз
два
три
setTimeout заданный в 0
```

Почему-то JavaScript прописывает **`раз`**, **`два`**, **`три`**, и только после этого прописывает **`setTimeout заданный в 0`**.

Можно пойти дальше, и, скажем, написать такой код с несколькими циклами:

```javascript
setTimeout(function() {
  console.log('setTimeout заданный в 0');
}, 0);
for (i = 0; i < 1000; i++) {
  console.log('раз');
}
for (i = 0; i < 1000; i++) {
  console.log('два');
}
for (i = 0; i < 1000; i++) {
  console.log('три');
}
```

Но в результате **`setTimeout заданный в 0`** все равно окажется в конце:

```
тысяча сообщений раз
тысяча сообщений два
тысяча сообщений три
setTimeout заданный в 0
```

Дело не в задержке самой по себе, хоть мы и задали нулевую задержку, проблема в том _когда_ js запускает функцию внутри `setTimeout`, а делает он это _не в тот же самый момент_ когда запускается код, хотя, "интуитивно" кажется что должно же быть именно так.

## Как setTimeout работает в JavaScript

Важно понимать что JavaScript это однопоточный (single-threaded) язык программирования. Про это часто пишут и говорят, но зачем-то эту концепцию излишне усложняют, хотя, вообще-то, все очень просто:

> JavaScript все делает в одном потоке. Он последовательно берет задачи из специального списка, который называется **Call Stack**. Проще всего воспринимать этот список как TODO лист, задачи из которого JavaScript и выполняет.

Именно по таким правилам живут эти `console.log`:

```javascript
console.log('раз');
console.log('два');
console.log('три');
```

JavaScript получает задачу написать в консоли "раз", эта задача появляется вверху списка, он выполняет ее и убирает ее из списка.

Все становиться сложнее когда появляются асинхронные операции, к которым, собственно, и относиться `setTimeout`.

> А сейчас, страшная тайна, меня это в свое время удивило, но вообще-то `setTimeout` это не часть JavaScript движка! Это Web API, браузеры (и, скажем, nodejs), просто, по доброте душевной, позволяют нам использовать `setTimeout`, именно браузеры обрабатывают и запускают указанную в `setTimeout` задержку. JavaScript работает c этим API, но это просто часть его окружения, это не сам движок.

Так как же будет обрабатываться наш код? Когда мы вызываем `setTimeout`:

```javascript
setTimeout(function() {
  console.log('setTimeout заданный в 0');
}, 0);
```

Сообщение добавляется в очередь с указанным обратным вызовом. А остальная часть кода:

```javascript
console.log('раз');
console.log('два');
console.log('три');
```

Продолжает выполняться синхронно. Как только этот код полностью завершится, цикл обработки событий (event loop - эвент луп) начнет опрашивать очередь сообщений на наличие следующего сообщения. Он найдет сообщение с обратным вызовом (callback) `setTimeout`, после чего начнется его обработка (выполнение обратного вызова).

Так что же все таки происходит, если задержка установлена ​​на `0`? Новое сообщение будет немедленно добавлено в очередь, но обработано оно будет только после того как текущий исполняемый код будет завершен и все ранее добавленные сообщения будут обработаны.

Проще говоря, обратный вызов выполняется только после завершения текущего выполняемого кода.

Получается, затронув такой, как казалось, простой вопрос, почему `setTimeout` с задержкой заданной в **`0`** не исполняется сразу же, нам пришлось иметь дело и с обратными вызовами, и с внешним окружением (Web API), и с эвент лупом, и тд.

## Дальнейшее изучение

Изначально эта статья была переводом вопроса и ответа со stackoverflow: [What is setTimeout doing when set to 0 milliseconds?](https://stackoverflow.com/questions/33955650/what-is-settimeout-doing-when-set-to-0-milliseconds), но через какое-то время все переросло в отдельную статью. Но частично это все еще перевод этого вопроса и ответа.

https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop

http://blog.carbonfive.com/2013/10/27/the-javascript-event-loop-explained/

Крутое видео на YouTube про то как работает эвент луп: [What the heck is the event loop anyway? | Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

Визуализация того как работает JavaScript: http://latentflip.com/loupe

Еще один визуализатор для JavaScript, но визуализирующий контексты, замыкания и области видимости: https://tylermcginnis.com/javascript-visualizer/
