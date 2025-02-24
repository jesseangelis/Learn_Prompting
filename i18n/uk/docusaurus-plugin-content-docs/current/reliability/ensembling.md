---
sidebar_position: 5
---

# 🟡 Групування запитів

Групування запитів – це концепція використання кількох різних запитів, щоб спробувати відповісти на одне запитання. Існує багато різних підходів до цього.

## DiVeRSe

DiVeRSe(@li2022advance) (**Di**verse **Ve**rifier on **R**easoning **S**t**e**ps) — це метод, який утричі покращує достовірність відповідей. Він робить це за допомогою: 1) використання кількох запитів для генерування різноманітних результатів, 2) використання верифікатора для відрізнення хороших відповідей від поганих, 3) використання верифікатора для перевірки правильності кроків обґрунтування.


import diverse from '@site/docs/assets/reliability/diverse.webp';

<div style={{textAlign: 'center'}}>
  <img src={diverse} style={{width: "750px"}} />
</div>

<div style={{textAlign: 'center'}}>
DiVeRSe (Li та ін.)
</div>

### Різноманітність запитів

DiVeRSe використовує 5 різних запитів для певних вхідних даних. Щоб створити кожен запит, вони випадковим чином відбирають кілька зразків із навчального набору. Ось приклад одного такого запиту (k=2), із прикладами, взятими з тесту [GSM8K](https://raw.githubusercontent.com/openai/grade-school-math/master/grade_school_math/data/train.jsonl)(@cobbe2021training). Фактично, у запитах для цього тесту DiVeRSe використовує 5 зразків.


```
П: Наталія продала 48 булавок своїм друзям у квітні, а потім у травні – удвічі менше. Скільки всього булавок продала Наталія за квітень і травень?
В: Наталія продала 48/2 = 24 булавки в травні.
Всього за квітень і травень Наталія продала 48+24 = 72 булавки.
#### 72
П: Венг заробляє 12 доларів на годину, працюючи нянею. Вчора вона працювала лише 50 хвилин. Скільки вона заробила?
В: Венг заробляє 12/60 = 0,2 долара за хвилину.
Відпрацювавши 50 хвилин, вона заробила 0,2 х 50 = 10 доларів.
#### 10
П: Бетті збирає гроші на новий гаманець, який коштує 100 доларів. У Бетті є лише половина потрібної суми. Її батьки вирішили дати їй на цю покупку 15 доларів, а дідусь і бабуся – удвічі більше, ніж батьки. Скільки ще грошей потрібно Бетті, щоб купити гаманець?
В:
```

Після генерації 5 різних запитів, як описано вище, DiVeRSe вибирає 20 шляхів обґрунтування для кожного (температура = 0.5). Ось кілька прикладів відповіді на наведений вище запит. Зауважте, що відповіді не завжди правильні.


```text
<!-- highlight-start -->
Бетті має 0,5 *100 = 50 доларів.
Потім вона отримує ще 15 = 65 доларів.
Тоді вона отримує 2*15 = 90 доларів.
Їй потрібно ще 100-90 = 10 доларів.
#### 10
<!-- highlight-end -->
```

```text
<!-- highlight-start -->
А: Бетті має 0,5*100 = 500 доларів.
Тоді вона отримує ще 15 = 650 доларів.
Тоді вона отримує 2*15 = 900 доларів.
Їй потрібно ще 100-90 = 1000 доларів.
#### 1000
<!-- highlight-end -->
```

Наразі DiVeRSe створив 100 різних результатів.

### Верифікатор голосування

Тепер ми можемо обрати результат більшості, як це робить Self-Consistency (@mitchell2022enhancing).

Однак DiVeRSe пропонує набагато складніший метод, який вони називають _верифікатором голосування_.

Під час тестування процес використання верифікатора голосувань складається з двох етапів. Спершу верифікатор (нейронна мережа) призначає оцінку 0-1 кожному результату на основі ймовірності, що він правильний. Потім компонент «голосування» підсумовує всі бали за різні відповіді та дає остаточну відповідь.

#### Приклад

Ось невеликий приклад. Скажімо, ми отримали такі результати за запитом `Скільки буде два плюс два?`:

```text
<!-- highlight-start -->
4
<!-- highlight-end -->
```

```text
<!-- highlight-start -->
два + 2 = 5
<!-- highlight-end -->
```

```text
<!-- highlight-start -->
Я думаю, 2+2 = 6
<!-- highlight-end -->
```

```text
<!-- highlight-start -->
два плюс два = 4
<!-- highlight-end -->
```

```text
<!-- highlight-start -->
Це 5
<!-- highlight-end -->
```

Верифікатор прочитає кожен результат і поставить йому оцінку. Наприклад, він може призначити бали: 0.9, 0.1, 0.2, 0.8, 0.3 відповідно. Потім компонент голосування підсумовує бали за кожну відповідь.

```
бал (4) = 0,9 + 0,8 = 1,7
бал (5) = 0,1 + 0,3 = 0,4
бал (6) = 0,2
```

Остаточна відповідь 4, оскільки вона має найвищий бал.

**Але як навчають верифікатор?**

Верифікатор навчений роботі зі складнішою функцією втрати, яку я не розглядатиму тут. Щоб дізнатися більше, прочитайте розділ 3.3 статті (@li2022advance).

## Запит «Запитайте мене про будь-що» (AMA)

import ama from '@site/docs/assets/reliability/AMA_Prompting.webp';

<div style={{textAlign: 'center'}}>
  <img src={ama} style={{width: "750px"}} />
</div>

Запит «Запитайте мене про будь-що» (AMA) (@arora2022ama) — це підхід, подібний до DiVeRSe. Однак, і його крок з кількома запитами, і крок агрегування відповідей суттєво відрізняються. Основна ідея AMA полягає в тому, щоб використовувати ВММ для генерації кількох запитів замість того, щоб просто використовувати різні приклади з кількома ілюстраціями.

### Кілька запитів

AMA показує, що ви можете взяти запитання та переформатувати його різними способами, щоб створити різні підказки. Наприклад, ви шукаєте інформацію про тварин на одному з вебсайтів і хочете записати лише тих, які живуть у Північній Америці. Створімо запит, щоб визначити це.

Враховуючи такий уривок із Вікіпедії:

```text
Кермодський ведмідь, який іноді називають ведмідь-дух (Ursus americanus kermodei), є підвидом американського чорного ведмедя та мешкає в районах центрального та північного узбережжя Британської Колумбії, Канаді.
```

Ви можете відформатувати це завдання у запит таким чином:

```text
Правильне чи хибне наступне твердження з урахуванням контексту?

Контекст: Кермодський ведмідь, якого іноді називають ведмідь-дух (Ursus americanus kermodei), є підвидом американського чорного ведмедя та мешкає в районах центрального та північного узбережжя Британської Колумбії, Канаді.
Твердження: Ця тварина живе в Північній Америці
Відповідь:
```

Це трохи дивне формулювання. Чому б не використати простіший запит?

```text
Контекст: Кермодський ведмідь, якого іноді називають «Ведмедем-духом» (Ursus americanus kermodei), є підвидом американського чорного ведмедя, який мешкає в районах центрального та північного узбережжя Британської Колумбії (Канада).
Питання: Чи живе ця тварина в Північній Америці?
```

Ну, формулюючи питання таким особливим чином, ми можемо генерувати різні запити. Нашим першим кроком буде взяти твердження `Ця тварина живе в Північній Америці` і переформатувати його в різні запитання, які в основному запитують те саме. Для цього ми передамо претензію через запити, як на зображенні нижче.

import ama_multi from '@site/docs/assets/reliability/AMA_multiprompting.webp';

<div style={{textAlign: 'center'}}>
  <img src={ama_multi} style={{width: "800px"}} />
</div>

Це може вивести:
1. Жила тварина жила в Північній Америці?
2. Чи живе тварина в Північній Америці?
3. Де живе тварина?

Ідея цього полягає в тому, щоб з різних боків *глянути на це* завдання. Потім ми застосовуємо кожен до заданого контексту так:

```text
Контекст: Кермодський ведмідь, якого іноді називають ведмідь-дух (Ursus americanus kermodei), є підвидом американського чорного ведмедя та мешкає в районах центрального та північного узбережжя Британської Колумбії, Канаді.
Питання: Чи жила тварина в Північній Америці?
```

Тоді ми можемо створити відповіді для кожного:

1. `Так, жила`
2. `Так, живе`
3. `Північна Америка`

Це *проміжні* відповіді. Нам потрібно зіставити їх із мітками завдань (наприклад, «Так» або «Ні»).

Ми можемо зробити це, передавши проміжні відповіді через такий запит:

```text
Виберіть правильну категорію.

"Категорії":
- Так, Північна Америка
- Ні, не Північна Америка

"Так, жила відповідає категорії:
```

Тепер ми можемо отримати вихідні відповіді.

1. `Так, Північна Америка`
2. `Так, Північна Америка`
3. `Так, Північна Америка`

Тут усі відповіді збігаються, тому ми можемо просто взяти першу відповідь. Однак, якщо вони не збігаються, ми можемо перейти до етапу агрегування AMA, щоб отримати остаточну відповідь.

### Агрегація відповідей

AMA використовує дуже складну стратегію для агрегування відповідей (більше, ніж DiVeRSe) замість того, щоб просто брати відповідь більшості. Щоб зрозуміти, чому відповідь більшості може бути поганим вибором, розглянемо два питання, які ми створили раніше:

1. Чи жила тварина в Північній Америці?
2. Чи живе тварина в Північній Америці?

Вони дуже схожі, тому, швидше за все, вони дадуть однаковий результат. Оскільки питання дуже схожі, вони фактично вплинуть на кінцевий результат. Щоб впоратися з цим, AMA покладається на слабкий контроль і складну математику, щоб оцінити залежності між різними запитами, які вона створює, а потім використовує це для відповідного їх зважування.

Отже, для трьох запитань, які ми згенерували, він може призначити ваги 25%, 25% і 50%, оскільки перші два дуже схожі.

Хоча стратегія агрегації AMA є потужною, вона настільки складна, що я не буду розповідати про неї тут. Щоб дізнатися більше, прочитайте розділ 3.4 статті (@arora2022ama).

### Результати

- Завдяки цій стратегії запитів AMA може використовувати GPT-J-6B(@wange2021gptj), щоб перевершити GPT-3.

- AMA краще підходить для запитань, де відповідь міститься у контексті.

## Висновки

Методи групування дуже потужні. Вони можуть використовуватися для покращення продуктивності будь-якої моделі та моделі для конкретного завдання.

На практиці мажоритарне голосування має бути вашою стратегією.

