Объекты - это списки значений, почти как в массивах, за исключением того, что значениям соответствуют ключи, а не числа.

Например:

```js
var foodPreferences = {
  pizza: 'yum',
  salad: 'gross'
};
```

## Условие задачи:

Создайте файл `objects.js`.

В этом файле объявите следующим образом переменную `pizza`:

```js
var pizza = {
  toppings: ['cheese', 'sauce', 'pepperoni'],
  crust: 'deep dish',
  serves: 2
};
```

Используйте `console.log()` и введите в терминал объект `pizza`.

Чтобы удостовериться в правильности решения задачи, запустите в терминале следующую команду:

```bash
node bin/javascripting verify objects.js
```