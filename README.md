## 1.Наполняем core-уровень КХД 

--- 

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/day-2.png"/>
</div>
Разберем выгрузку очищенных данных на Core-уровень КХД. Сюда сохраняются данные, которые потом используются для построения витрин и моделей на уровне Data Mart.

КХД состоит из нескольких условных уровней:

- Уровень Stage (опционально) — выгрузка данных из источника как есть — без трансформаций, чтобы дальнейшие процессы не дергали источники.

- ETL-процессы — трансформации данных со Stage уровня и из источников, чтобы получить таблицы, из которых удобно собирать витрины/модели

- Core уровень — хранение данных, полученных в ходе ETL-процессов.

- Datamart уровень — тут формируются витрины и модели из таблиц Core уровня, к которым подключаются BI и другие потребители.
<div>
<img src="https://loginom.ru/sites/default/files/global-files/textpage/2025/data-warehouse_2.svg"/>
</div>

---

<h2 align="center">Аналитическое хранилище</h2>

Как вы знаете, Loginom предоставляет большое количество возможностей по трансформации, очистке и обогащению данных («Генеральная уборка данных»). Однако, чтобы эти данные стали частью КХД, недостаточно просто взять и выгрузить их в базу. Нам нужно, чтобы таблицы на Core-уровне были совместимы друг с другом, буквально как кубики конструктора или части мозаики.

Такая «квадратная» совместимость важна для того, чтобы при росте количества таблиц сохранялась единая логика связей, и мы могли легко изменять модели данных. Более того, только такой подход является реально масштабируемым и позволяет не закапываться в бесконечных исправлениях «уникально созданных» моделей.
<div>
<img src="https://loginom.ru/sites/default/files/global-files/textpage/2025/data-model-collection.svg"/>
</div>

<h2 align="center">Сбор модели данных</h2>

### С чем будем работать

Для наполнения КХД мы приготовили набор таблиц следующей структуры (в таком виде они будут лежать на Core-уровне, до организации в витрину или «Звезду»).

<div>
<img src="https://loginom.ru/sites/default/files/global-files/textpage/2025/table-structure.svg"/>
</div>

<h2 align="center">Сбор модели данных</h2>

В него входят:

- Таблицы показателей (таблицы фактов):
  - Продажи — в разрезе дней, менеджеров, клиентов и товаров;
  - План продаж — в разрезе месяцев, товаров и менеджеров;
  - Прогноз продаж — прогноз в разрезе месяцев и категорий товаров.
- Справочники:
  - Товары — справочник товаров;
  - Менеджеры продаж — сотрудники, закрепленные за продажами. По ним же строится и план продаж;
  - Клиенты — справочник клиентов:
  - Менеджеры клиентов — сотрудники, закрепленные за клиентами;
  - Классификаторы A и B — дополнительные справочники для усложнения модели.