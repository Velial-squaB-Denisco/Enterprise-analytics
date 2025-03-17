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

Разделение на таблицы показателей и справочников условно. Таблица показателей обычно включает поля для расчетов (меры, measures; например, «Выручка» в таблице продаж) и даты, к которым привязаны эти данные. В ней также есть ключевые поля, которые связывают данные со справочными таблицами.

Справочники содержат поля, которые на стороне аналитических систем становятся измерениями (еще их называют разрезы, аналитики, dimensions). По значениям этих полей на визуализациях разбиваются результаты агрегаций. Например, в диаграмме, на которой показывается разбивка выручки по клиентам, поле «Клиент» является измерением.

При этом таблицы показателей могут содержать свои собственные поля измерений (когда эти поля не целесообразно выносить в отдельный справочник, т.к. они используются только для этой таблицы). А в условном справочнике клиентов может появиться поле «Дата создания», и вот уже мы можем посчитать показатель «Количество новых клиентов», посчитав идентификаторы клиентов в разрезе дат их создания.

В нашем подходе на уровне КХД таблицы справочников и полей ничем технически не отличаются. Это просто общепринятое обозначение для прояснения основной роли таблицы.

### Пример сценария для выгрузки данных в Core-уровень КХД

Откройте сценарий второго дня в папке M3\Data Monetization Pack\Examples. Скачайте пакет в папку.

<h2 align="center">
<a href="https://downloads.loginom.ru/day2-saving-data-core-level.lgp" style="background-color: #4CAF50; color: white; padding: 10px 20px; text-align: center; text-decoration: none; display: inline-block; border-radius: 5px;">Скачать файл LGP</a>
</h2>

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/file-lgp-1.png"/>
</div>

<h2 align="center">LGP-файл</h2>

Откройте пакет.

Для справки: когда вы создаете сценарий выгрузки данных в КХД, в вашем пакете должны находиться ссылки на 3 набора компонентов:
 - Clickhouse Kit — для взаимодействия с СУБД Clickhouse;
  - Data Preparation Kit — для быстрого просмотра состояния хранилища;
  - Data Control Kit — для аудита данных и оповещениях об инцидентах.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/packet-2.png"/>
</div>

<h2 align="center">Пакет</h2>

В сценарии находится подмодель для очистки КХД. Можете запустить ее, если запутаетесь и захотите сбросить вашу базу к исходному состоянию, и проделать все операции с чистого листа.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/cleaning-dwh-3.png"/>
</div>

<h2 align="center">Очистка КХД</h2>

Обычно в ETL-сценариях для КХД подготовка каждой таблицы происходит внутри ее подмодели. Зайдем в подмодель Sales plan, чтобы увидеть детали процесса.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/submodel-sp-4.png"/>
</div>

<h2 align="center">Подмодель Sales plan</h2>

Импорт LGD-файла заменяет нам ETL-процесс, который происходит на этапах 1-2 схемы потока данных. В реальном сценарии тут скорее всего была бы подмодель с ETL-процессом.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/replacing-submodel-5.png"/>
</div>

<h2 align="center">Замена подмодели с ETL-процессом</h2>

Разберем, как должна выглядеть правильно подготовленная таблица для core-уровня КХД.

## Как правильно подготовить данные для Core-уровня КХД

Откроем предпросмотр данных из LGD-файла плана продаж.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/data-preview-6.png"/>
</div>

<h2 align="center">Предпросмотр данных</h2>

Технически, можно сохранить в БД любые данные. Но если мы хотим, чтобы в дальнейшем из них было легко собрать витрины/модели, в которые в будущем можно легко добавить другие таблицы, нужно соблюсти следующий набор правил.

## Нейминг полей — ключевые поля

Ключевые поля — это такие поля, по которым строится связь между таблицами, позволяя делать вычисления по полям одной таблицы в разрезе полей другой таблицы.

- Ключевые поля имеют свой префикс. В DMP (Data Monetization Pack) этот префикс определяется в файле настроек Data_prep.txt, по умолчанию key_;
- Ключевые поля называются одинаково во всех таблицах. Если идентификатор товара в справочнике товара называется key_Product_ID, он должен называться так во всех таблицах;
- Метка и имя ключевого поля должны быть одинаковыми.

Нейминг полей — поля данных:

- Не ключевое поле может иметь произвольную метку и имя;
- Для облегчения будущего сбора витрин, нужно заранее определить роли полей. Их может быть 3: мера, измерение, каноническая дата.

Меры определяются добавлением «_M» в конец метки поля. Каноническая дата определяется через добавление «_CD» в конец метки поля. Каноническая дата — термин из моделирования данных. Обозначает единую ось данных в модели/витрине, вокруг которой собираются показатели из разных таблиц, и их можно анализировать на одной временной оси (например, план и факт продаж). Одна таблица может содержать несколько канонических дат. Подробнее разберем эту тему позже.

Все поля, не являющиеся ключами, мерами или каноническими датами, становятся измерениями. Во время сборки витрины/модели, измерения могут быть связаны по ключевым полям с другими таблицами. Давая таким образом анализировать показатели одних таблиц в разрезе полей из других таблиц.

## Форматы дат:

- По возможности убирайте из полей дат значение времени (округляйте значение даты вниз до целого). Это оптимизирует работу будущих моделей/витрин;
- Если поле содержит не дату (например, день продаж), а обозначение периода (например, месяц плана) — такое значение должно быть приведено к первой дате периода (первый день месяца плана). Это позволит на одной временной оси связать данные разных разрядностей во времени и анализировать их совместно в более укрупненных группировках календаря (например, месяц, квартал, год).

## Общие рекомендации по полноте данных таблиц Core-уровня

Особенность потока превращения данных в аналитику в том, что преобразования данных могут происходить на каждом из этапов пути.

<div>
<img src="https://loginom.ru/sites/default/files/global-files/textpage/2025/system-nfe-1%20(1)_0.svg"/>
</div>

<h2 align="center">Поток данных</h2>

Можно посчитать все данные на стороне источника (особенно если это что-то вроде 1С), выгрузить это в BI и визуализировать простейшими формулами.

А можно взять самые кошмарные данные, загрузить их сразу в BI, написав там сложный ETL-процесс, трехэтажные формулы, и получить аналогичный результат.

А еще можно создавать новые аналитические признаки на этапе формирования модели/витрины. В общем, можно посчитать много что и много где. Как же в итоге лучше распределить на потоке преобразования вычисления, что и где лучше считать?

Вот некоторые рекомендации:

1. Если какой-то расчетный показатель может быть в готовом виде получен из источника — лучше взять его из источника (например, расчетные показатели из 1С вроде валовой прибыли);

2. Чем больше показателей будет посчитано на уровне Core, тем проще вам будет это визуализировать дальше, т.к. не придется полагаться на возможности выражений BI-системы. Ваши данные должны быть подготовлены на таком уровне, чтобы 90% показателей считалось максимум формулами вроде sum_if и ее аналогами.

Почему? Потому что чем больше специфических функций используется на стороне визуализации данных, тем менее универсальным является КХД, и тем больше вы привязаны к конкретному инструменту аналитики. Чем больше показателей и логики рассчитывается на core-уровне КХД, тем проще ее переиспользовать в любых инструментах. И к этому нам и нужно стремиться, но без фанатизма.

3. Расчеты, для которых требуется высокий уровень интерактивности, и которые нельзя заранее рассчитать на core-уровне, имеет смысл делать на стороне BI. Например, показатель «Средний чек», который считается как сумма продаж, деленная на количество продаж.

В аналитике этот показатель может требоваться выводить в самых разных разрезах с самыми разными фильтрами. Посчитать это на уровне таблицы данных заранее в таком количестве вариантов невозможно. Поэтому такие вещи смело считаем на стороне BI.

## Сохранение данных в core-уровень КХД

Результат ETL-процесса должен быть сохранены в Clickhouse, и за это отвечает компонент «📁 CH: Выгрузка в КХД».

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/replacing-submodel-5.png"/>
</div>

<h2 align="center">Замена подмодели с ETL-процессом</h2>

Этот компонент — часть библиотеки Clickhouse Kit.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/clickhouse-kit.png"/>
</div>

<h2 align="center">Clickhouse Kit</h2>

Т.к. библиотеки из Data Monetization Pack содержат множество компонентов, которые, в том числе, должны комбинироваться друг с другом для решения рабочих задач, для упрощения работы используются готовые шаблоны сценариев (отмечены значком папки). Такие компоненты добавляются в сценарий как производный узел, т.е. подмодель в которую можно зайти, увидеть готовую схему связей компонентов, и донастроить ее под ваш сценарий.

Зайдем в подмодель «📁 CH: Выгрузка в КХД».

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/submodel-8.png"/>
</div>

<h2 align="center">Подмодель</h2>

Активируем компонент «Аудит таблицы». Он используется для проверки, все ли учтено для сохранения таблицы в core-уровень.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/table-audit-9.png"/>
</div>

<h2 align="center">Компонент «Аудит таблицы</h2>

Первый порт пропускает входные данные без изменений.

Второй порт содержит метаданные, т.е. описание полей таблицы, которые будут использоваться для формирования интерактивной документации и автоматизаций КХД.

Часть метаданных формируется автоматически, а другая часть (в т.ч. произвольное описание полей) задается вами в настройках аудита (про них поговорим дальше).

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/metadata-10.png"/>
</div>

<h2 align="center">Метаданные</h2>

​​​​​Третий порт содержит результаты аудита. Если там есть строки с красным крестиком, значит не все было правильно подготовлено к сохранению в core-уровень. В данном случае, проблема в заполнении обязательных параметров.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/audit-results-11.png"/>
</div>

<h2 align="center">Результаты аудита</h2>

Откроем второй порт переменных на входе в аудит.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/table-audit-9%20%281%29.png"/>
</div>

<h2 align="center">Второй порт переменных</h2>

Заполните указанные параметры в соответствие с примером:

- Название таблицы — то, как будет называться таблица в базе core-уровня КХД. Рекомендуется писать английские буквы, без пробелов;
- Описание таблицы — краткий комментарий (произвольный текст) о том, что это за таблица;
- Первичный ключ — ключевое поле с префиксом key_, являющееся первичным ключом (если таковое есть в данных, иначе оставляем пустым. Пример с заполнением первичного ключа можно посмотреть в подмодели Products);
- Префикс имени/метки поля — аббревиатура, которая будет использована при построении витрин для идентификации полей, взятых из данной таблицы. Т.к. витрины соединяют в одной таблице поля из множества таблиц core-уровня, очень полезно понимать, из какой таблицы какое поле берется.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/received-data-12.png"/>
</div>

<h2 align="center">Полученные данные</h2>

