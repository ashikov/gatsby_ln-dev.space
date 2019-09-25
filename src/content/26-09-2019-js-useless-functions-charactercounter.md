---
title: "JS: Бесполезные функции - characterCounter()"
date: "2019-09-26"
draft: false
path: "/blog/26-09-2019-js-useless-functions-charactercounter"
---

Данная функция иллюстрирует рекурсивный итеративный процесс.

Задача следующая. Необходима функция которая первым параметром принимает строку, вторым параметром - символ и возвращает количество вхождений данного символа в переданную строку.

Реализация с помощью итеративного процесса:

```javascript
const characterCounter = (string, searchCharacter) => {
  const iter = (index, acc) => {
    if (string[index] === undefined) {
      return acc;
    }

    const newAcc = string[index] === searchCharacter ? acc + 1 : acc;

    return iter(index + 1, newAcc);
  };

  return iter(0, 0);
};
```

Рекурсивно передаем в __iter()__ увеличивающийся на каждой итерации индекс положения символа в строке и накапливаем результат вхождений в __newAcc__. Увеличиваем __newAcc__ на 1 если [тернарник](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) возвращает __true__.

Тоже самое, только с использованием цикла [for](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for).

```javascript
const characterCounter = (string, searchCharacter) => {
  let counted = 0;
  for (let i = 0; i < string.length; i += 1) {
    if (string[i] == searchCharacter) {
      counted += 1;
    }
  }
  return counted;
};
```

Ну и для теста (хотя понятно и так, что все работает), что-то вроде:

```javascript
const string = 'Let The Force be with you!';
const searchCharacter = 'e';
const result = characterCounter(string, searchCharacter);

console.log(`Количество символов "${searchCharacter}" в строке "${string}" = ${result}`);
```

Да, на русском языке. [Unicode](https://en.wikipedia.org/wiki/Unicode) же. Почему бы и нет, если сегодня такое настроение? :)
