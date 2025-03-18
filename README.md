<div>
<img src="https://www.syssoft.ru/upload/iblock/5be/po7970wzyp58ws9j9ikmgiznih9m0sz2.png"/>
</div>

## 1.Наполняем core-уровень КХД 
---

### Краткое изложение

Статья описывает процесс наполнения Core-уровня корпоративного хранилища данных (КХД) очищенными данными. КХД состоит из нескольких уровней: Stage (опционально), ETL-процессы, Core-уровень и Data Mart. Основное внимание уделяется Core-уровню, где хранятся данные, полученные в результате ETL-процессов.

#### Основные моменты:

1. **Уровни КХД**:
   - **Stage**: Хранение данных без трансформаций.
   - **ETL-процессы**: Трансформация данных для удобного сбора витрин/моделей.
   - **Core-уровень**: Хранение данных для дальнейшего использования.
   - **Data Mart**: Формирование витрин и моделей для BI и других потребителей.

2. **Совместимость данных**:
   - Данные на Core-уровне должны быть совместимы друг с другом, как кубики конструктора, чтобы сохранялась единая логика связей и можно было легко изменять модели данных.

3. **Структура таблиц**:
   - Таблицы делятся на таблицы показателей (факты) и справочники.
   - Таблицы показателей включают поля для расчетов и даты, а справочники — поля для измерений.

4. **Процесс выгрузки данных**:
   - Описан пример сценария выгрузки данных в Core-уровень КХД с использованием LGP-файла.
   - Важно соблюдать правила нейминга полей и форматов дат для оптимизации будущих моделей/витрин.

5. **Рекомендации**:
   - Расчетные показатели лучше брать из источника, если это возможно.
   - Чем больше показателей посчитано на уровне Core, тем проще их визуализировать.
   - Расчеты, требующие высокой интерактивности, лучше делать на стороне BI.

6. **Сохранение данных**:
   - Результат ETL-процесса сохраняется в Clickhouse.
   - Используются компоненты для аудита таблиц и создания/адаптации таблиц в Clickhouse.

**Полезные моменты:**

- Подробное описание уровней КХД и их ролей.
- Практические рекомендации по подготовке данных для Core-уровня.
- Описание процесса выгрузки данных и использования компонентов для аудита и создания таблиц.

**Менее полезные моменты:**

- Некоторые технические детали могут быть сложны для понимания без контекста или опыта работы с КХД.
- Описание процесса выгрузки данных может быть избыточным для пользователей, не знакомых с конкретными инструментами (например, Loginom).

--- 
## Материал
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

Сохраните настройки, запустите аудит повторно и откройте третий выходной порт. Вы должны увидеть отсутствие красных крестиков. Лампочка обозначает ситуацию, которая не является критической ошибкой, но влияет на то, как данные этой таблицы могут быть использованы в дальнейшем. Просто убедитесь, что пункты с лампочкой существуют в соответствие с вашей задумкой.

Проверьте также второй выходной порт — вы должны увидеть новые атрибуты в метаданных, которые появляются только при отсутствии проблем в аудите.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/output-port-13.png"/>
</div>

<h2 align="center">Второй выходной порт</h2>

После успешного аудита может быть активирован узел сохранения метаданных. Если в аудите есть критические ошибки, сохранение не сработает, и дальнейший процесс выгрузки данных не будет произведен. Метаданные сохраняются в папке библиотеки DMP в виде LGD-файлов.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/saving-node-14.png"/>
</div>

<h2 align="center">Узел сохранения метаданных</h2>

Чтобы данные могли быть выгружены в базу, в ней должна быть создана принимающая таблица. Компонент «CH: Создать / адаптировать MT таблицу» автоматически создает таблицу на основе полей данных.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/receiving-table-15.png"/>
</div>

<h2 align="center">Принимающая таблица</h2>

Данный компонент является универсальным и может использоваться для создания таблиц в Clickhouse для произвольных задач. Разберем настройки этого компонента в порту переменных.

Прежде всего, здесь нужно указать базу данных и таблицу, которая будет принимать данные. Если их не существует — они будут созданы автоматически. Т.к. компонент используется в рамках преднастроенной схемы, в нем уже указана принимающая БД (временная база для core-уровня) и название таблицы (на основе значения, указанного в аудите).

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/data-16.png"/>
</div>

<h2 align="center">Данные</h2>

Далее, идет набор специфичных для Clickhouse настроек. Краткая справка по основам хранения данных в Clickhouse:

У каждой таблицы есть движок. Движок определяет способ хранения данных и логику отработки запросов. Основной движок, использующий преимущества СУБД Clickhouse — MergeTree() и его разновидности. Он установлен в настройках по умолчанию.

Фишка Clickhouse — в быстрых выборках больших объемов данных. Однако, чтобы этот функционал работал более оптимально, для таблицы можно задать ряд дополнительных настроек:

Поле Партиции. Партиция — это поле, по которому идет разбивка на большие блоки данных с целью их быстрой переброски между разными таблицами. Имеет смысл использовать при наличии в партициях от 100 000 строк. Раньше имели большую роль в оптимизации чтения, но теперь эта роль сместилась в параметр Order By, указываемый при создании таблицы.

Поля сортировки Order By — основной способ ускорения чтения данных из MergeTree таблиц. Здесь обычно указывают набор полей, по которым происходит группировка данных при запросах из таблицы. При этом поля указываются в порядке возрастания частотности значений (количество уникальных значений или кардинальность).

Обычно это важно для таблиц, к которым идут запросы вида Group by (здесь перечисляются наиболее часто используемые поля в Group by), что довольно редко происходит для таблиц core-уровня, т.к. они используются для сборок витрин в виде материализованных таблиц на базе select-запросов. А вот к витринам уже идут group by запросы из BI. Таким образом, без особой необходимости для таблиц core-уровня здесь ничего можно не писать.

Однако, этот параметр является обязательным для создания таблицы. Поэтому оставим его пустым, таблица будет создана с сортировкой order by Tuple(), что эквивалентно сортировке по всем полям в порядке их создания. Для более оптимальной работы нужно, чтобы поля были расставлены в порядке возрастания частотности значений.

Чтобы не заниматься расстановкой порядка полей вручную (определяется порядком на входном порту), можно активировать опцию «Порядок полей по частотности при создании таблицы».

Тогда перед созданием таблицы будет проведен анализ частотности значений (может занять некоторое время), и порядок определится автоматически.

Поля первичного ключа — это не тот первичный ключ, о котором вы подумали (и он не имеет отношения к первичному ключу, указанному в аудите). В Clickhouse есть разновидность MergeTree движков, которые умеют агрегировать данные в фоновом режиме. Делают они это в разрезе полей, указанных в Order By. Например, в таблицу пишется информация из чековой ленты, разбитая по товарам внутри чека (т.е. по несколько строк на 1 чек). Но за счет указания Order By вида Город, Магазин, Дата, ИД чека и использования движка SummingMergeTree, данные суммируются на уровне чека.

Однако, такой Order By не является оптимальным с точки зрения использования group by запросов, т.к. в нем содержится слишком детализированное поле ИД чека. И вот для этих случаев используется параметр PrimaryKey — в него можно дописать часть полей, которые уже используются в Order By. Например, только Город, Магазин, Дата. Это сделает работу group by запросов более оптимальной.

### В целом для сохранения данных в core-уровень КХД вы можете держать все эти настройки пустыми. Они нужны прежде всего для специальных таблиц-витрин, к которым обращаются аналитические инструменты.

Параметр Nullable_mode (0 — все nullable, 1 — все не nullable, 2 — автоопределение) отвечает за то, могут ли принимать поля таблицы значения NULL или нет. При создании таблицы, для каждого поля может быть определена эта опция. Nullable-поля работают менее оптимально по сравнению с не-nullable с точки зрения скорости выполнения запросов. Но при этом нет смысла не использовать их, если такова логика данных (например, отсутствующее значение в ключевых полях все-таки имеет смысл хранить как null, чтобы не получить искусственный пустой ключ, на который будут завязаны все несвязанные данные).

Режим «2 — автоопределение» задает атрибут Nullable/не Nullable при создании таблицы в зависимости от наличия null-значений в текущем наборе данных.

Удалить и пересоздать. Если принимающая таблица уже существует в БД, то будет проведена проверка соответствия сохраняемой таблицы из Loginom и принимающей таблицы. При обнаружении расхождений будет выполнен запрос ALTER TABLE для того, чтобы принимающая таблица соответствовала по структуре сохраняемой. Возможные изменения: добавление полей, удаление полей, смена типов данных в поле.

Иногда возможны ситуации, когда изменение таблицы через запрос ALTER TABLE невозможно (например, вы переименовали все поля в сохраняемой таблицы, и теперь для актуализации нужно удалить все поля в принимающей таблице и создать новые с новыми именами, или происходит конвертация несовместимых типов данных). Тогда этот компонент будет возвращать ошибку и останавливать процесс сохранения данных. Чтобы исправить ситуацию, таблицу нужно пересоздать с нуля. Для этого можно активировать данную опцию и выполнить компонент. Не забудьте потом отключить ее, чтобы таблица не пересоздавалась при каждой активации компонента.

После активации компонента, на выходе окажутся:

1. Ваши данные, которые проходят насквозь без изменений;

2. Выполнение запроса DESCRIBE TABLE к целевой таблице показывает, что она реально существует в БД.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/execution-request-17.png"/>
</div>

<h2 align="center">Выполнение запроса</h2>

​​​​​3. Отчет о результатах активации компонента. Там в числе прочего можно увидеть действия, выполненные над таблицей и запрос (создание/изменение/ничего). А также шаблоны запросов Select с подстановкой алиасов и без — для быстрого использования в других местах.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/report-18.png"/>
</div>

<h2 align="center">Отчет</h2>

Далее отрабатывает стандартный узел экспорта в БД. В нем можно настроить способ обновления данных (частичная / полная перезапись).

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/export-19.png"/>
</div>

<h2 align="center">Экспорт в БД</h2>

За счет того, что в сопоставлении полей включена автосинхронизация, вам не нужно настраивать его вручную. Кроме того, при изменении структуры таблицы изменение сопоставления полей будет происходить автоматически.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/auto-sync-20.png"/>
</div>

<h2 align="center">Автосинхронизация</h2>

Финальный этап — публикация таблицы. Этот компонент мгновенно переносит таблицу из текущего положения в новое («Старая БД».«Старая таблица» > «Новая БД. Новая таблица»).

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/publication-table-21.png"/>
</div>

<h2 align="center">Публикация таблицы</h2>

В случае сценария выгрузки в КХД, это означает перенос из временного core-уровня в рабочий. Этот процесс необходим, чтобы во время длительных процессов записи данных не допустить чтения неполных данных из рабочей базы. Также у компонента есть опция — создать клон публикуемой таблицы в старом расположении. Таким образом, можно одновременно иметь 2 версии таблицы, тестовую и рабочую. При этом процесс публикации данных можно отключить в настройках компонента, чтобы данные обновлялись только в тестовой таблице.

Для эксперимента можете подать данные на выгрузку через параметры полей (данные подаются не из Sales plan, а через параметры полей. Набор полей меняется на входе в компонент и дальше автоматически меняется в БД, позволяя принять новую таблицу), и активировать выгрузку в КХД.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/field-options-22.png"/>
</div>

<h2 align="center">Параметры полей</h2>

1. Таким образом убедитесь, что изменение таблицы не потребовало от вас перенастройки сценария;
2. «CH: Создать / адаптировать MT таблицу» на сообщение о том, что выполнен запрос на изменение принимающей таблицы.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/publication-table-21.png"/>
</div>

<h2 align="center">Публикация таблицы</h2>

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/execution-request-23.png"/>
</div>

<h2 align="center">Выполнение запроса</h2>

### Переключите подачу данных на выгрузку обратно на LGD-файл и повторно выполните выгрузку.

Вернитесь на исходный сценарий и выполните всю цепочку обновления данных (там везде внутри все по аналогии с разобранным примером Sales plan).

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/data-update-24.png"/>
</div>

<h2 align="center">Обновление данных</h2>

### Проверяем результат работы
Добавьте в сценарий импорт из базы данных на основе готового подключения Clickhouse ADWH.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/import-database-25.png"/>
</div>

<h2 align="center">Импорт из базы данных</h2>

В настройках узла импорта подтвердите наличие девяти таблиц во временной и рабочей core-базах.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/node-setup-26.png"/>
</div>

<h2 align="center">Настройка узла</h2>

В производных компонентах, в разделе Data_Preparation_Kit, добавьте в сценарий компонент «3.1. Список таблиц (мета-справочник)».

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/adding-component-27.png"/>
</div>

## 2.Мониторинг никогда не спит

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/day3.png"/>
</div>

Прежде чем переходить к сборке слоя data-mart, разберем еще одну тему — мониторинг качества данных. В первом марафоне «Генеральная уборка данных» [![Loginom](https://img.shields.io/badge/Loginom-Visit%20Site-blue)](https://data-preprocessing.loginom.ru/)
 мы разбирали множество вариантов того, как недостаточно качественные данные могут исказить результаты анализа.

Вопрос в том, что у нас не всегда есть возможность постоянно контролировать отсутствие проблем и ошибок лично. И зачастую есть смысл узнавать о проблемах до того, как они всплывут в финальной аналитике.

<div>
<img src="https://loginom.ru/sites/default/files/global-files/textpage/2025/data-q.svg"/>
</div>

<h2 align="center">Качество данных</h2>

### Компонент для оповещений в Telegram

Откройте пакет прошлого занятия, зайдите в подмодель Sales_plan. Разместите там компонент «📁 Аудит качества данных» из Data_Control_Kit. Подайте ему на вход данные из импорта LGD-файла. [![Loginom](https://img.shields.io/badge/Loginom-Visit%20Site-blue)](https://help.loginom.ru/userguide/data-format/lgd-file.html)

Как вы можете видеть, «📁 Аудит качества данных» — это производный узел, а значит содержит внутри себя шаблон сценария, который мы будем настраивать.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/derived-node-1.png"/>
</div>

<h2 align="center">Производный узел</h2>

Но для начала задайте на входном порту переменных название проверки, которую мы будем проводить.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/check-3.png"/>
</div>

<h2 align="center">Проверка</h2>

Внутри нас ждет готовая схема.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/ready-made-scheme-2.png"/>
</div>

<h2 align="center">Готовая схема</h2>

Сэмплинг позволяет отобрать определенное количество строк из разных частей таблицы, что особенно актуально при большом объеме данных, когда проверка всех записей становится нецелесообразной.

В калькуляторе «DQ проверки» задаются условия для контроля значений. Давайте зайдем в него.

Суть контроля данных заключается в создании логических полей, которые принимают значение true, если в данных выявлена проблема или присутствует фактор, который требуется отслеживать.

Название такого поля должно начинаться с префикса DQ_, за которым следует название проверки. Рассмотрим пример проверки значения в поле Amount_plan >= 30000.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/checking-value-4.png"/>
</div>

<h2 align="center">Проверка значения в поле Amount_plan</h2>

Если требуется создать несколько проверок, достаточно клонировать поле проверки, задать ему новую метку и прописать новое условие.

Давайте создадим вторую проверку с условием Amount_plan <= 500.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/second-check-5.png"/>
</div>

<h2 align="center">Вторая проверка</h2>

​​​​Представим, что таким образом мы будем контролировать наличие подозрительно больших и маленьких объемов в планах продаж.

Компонент «Отчет DQ» формирует сводную информацию на основе созданных полей. Сводка включает:

- количество строк с инцидентами;
- процентное соотношение этих строк к общему количеству записей.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/component-6.png"/>
</div>

<h2 align="center">Компонент «Отчет DQ»</h2>

Одна из особенностей производных узлов — это возможность не только изменять настройки существующего сценария, но и добавлять в него собственные узлы.

Добавим узел Дубликаты и противоречия из стандартной библиотеки элементов между Калькулятором и «Отчетом DQ».

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/node-7.png"/>
</div>

<h2 align="center">Узел «Дубликаты и противоречия»</h2>

Давайте ради теста сделаем проверку дубликатов в поле Key_ERP_User_Sales_ID, указав его как входное поле.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/test-8.png"/>
</div>

<h2 align="center">Тест</h2>  

Очень удобно, что в результате проверки создаются логические поля, принимающие значение true для дубликатов. Необходимо лишь переименовать метку на выходном порте поля, например, в «DQ_Дубли ИД менеджера», чтобы узел «Отчет DQ» включил его статистику в сводку.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/verification-result-9.png"/>
</div>

<h2 align="center">Результат проверки</h2>  

Теперь статистика по дубликатам тоже формируется в сводке.

<div>
<img src="https://loginom.ru/sites/default/files/styles/image792/public/global-images/textpage/2025/statistics-duplicates-10.png"/>
</div>

<h2 align="center">Статистика по дубликатам</h2>

Как узнать, что в данных есть проблема, не заходя в Loginom?

Так как в первый день мы посвятили время созданию и настройке Telegram-бота, активация последнего компонента в сценарии отправит сообщение в технический чат через бота. Это позволит оперативно получать уведомления о выявленных проблемах.

<div>
<img src="https://loginom.ru/sites/default/files/global-images/textpage/2025/message-chat-11.png"/>
</div>

<h2 align="center">Сообщение в чат</h2>

