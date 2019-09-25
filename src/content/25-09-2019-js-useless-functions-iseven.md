---
title: "JS: Бесполезные функции - isEven()"
date: "2019-09-18"
draft: false
path: "/blog/25-09-2019-js-useless-functions-iseven"
---

В новой набирающей популярность рубрике __JS: Бесполезные функции__ я буду постить странные функции, вроде ***Функция которая определеяет является ли число четным** или **Функция вычисляющая количество вхождений определенного символа в строку**. Думаю идея понятна.

Сегодня разберемся с функцией isEven и проверкой числа на четность.

>Чётное число — целое число, которое делится на 2 без остатка: …, −4, −2, 0, 2, 4, 6, 8, …

Обратим внимание на то, что __0__ тоже является четным числом.

Рекурсивная реализация выглядит так:

```javascript
const isEven = (number) => {
  if (number === 0) {
    return true;
  }
  if (number === 1) {
    return false;
  }
  if (number < 0) {
    return isEven(-number)
  };
  
  return isEven(number - 2);
}
```

Если вы только недавно начали изучать программирование, то сейчас присядьте. Без рекурсии:

```javascript
const isEven = num => num % 2 === 0;
```

Потестировать обе функции:

```javascript
console.log(isEven(50));
console.log(isEven(75));
console.log(isEven(1));
console.log(isEven(-1));
console.log(isEven(0));
console.log(isEven(-4));
```

Вывод приводить не буду, если что сами разберетесь. :)