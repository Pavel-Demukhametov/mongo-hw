# ДЗ 2

ФИО: Демухаметов Павел

## Окружение
- Windows 11 + Docker
- Образ: mongo:8.0.14

---

## Схема данных
- products: { _id, name, price }
- warehouses: { _id, product, warehouse, instock }
- Связь: warehouses.product = products.name

---

## Генерация данных

~~~javascript
use shop;

// Функция случайного числа в диапозоне 
function rnd(min, max) { return Math.floor(Math.random() * (max - min + 1)) + min; }

// Массив названий товаров
let items = [
  "almonds","pecans","cookies","bread","cashews","chocolate",
  "coffee","tea","milk","sugar","rice","pasta"
];

// Создание товаров по всем названиям
for (let i = 1; i <= items.length; i++) {
  db.products.insertOne({
    _id: i,
    name: items[i - 1],
    price: rnd(10, 300)
  });
}

// Создание случайного склада со случаным количеством товаров 
let codes = ["A","B","C"];
for (let wid = 1, limit = rnd(10, 20); wid <= limit; wid++) {
  db.warehouses.insertOne({
    _id: wid,
    product: items[rnd(0, items.length - 1)],
    warehouse: codes[rnd(0, codes.length - 1)],
    instock: rnd(0, 200)
  });
}
~~~

### Скрин значений в products
![Скрин](images/1.jpg)

### Скрин значений в warehouses
![Скрин](images/2.jpg)

## Aggregation framework
~~~javascript
db.products.aggregate([
  { $lookup: {
      from: "warehouses",
      localField: "name",
      foreignField: "product",
      as: "wh"
  }},
  { $set: {
      totalInstock: { $sum: "$wh.instock" }
  }},
  { $project: { _id: 0, name: 1, totalInstock: 1 } },
  { $sort: { name: 1 } }
]);
~~~
### Скрин результата
![Скрин](images/3.jpg)


## Map Reduce
~~~javascript

// Получаем название и количество товара
function mapStock() {
  if (this.product != null) emit(this.product, this.instock);
}
// Сумма количества товаров по названию
function reduceStock(key, values) {
  return Array.sum(values);
}

db.warehouses.mapReduce(mapStock, reduceStock, { out: { replace: "totals_by_product" } });

db.totals_by_product.find().sort({ _id: 1 });
~~~
### Скрин результата
![Скрин](images/4.jpg)