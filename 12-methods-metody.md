# 12 METHODs (Методы)

## 12.1 General structure

**METHOD** определяет подпрограмму, выполняемую в приложении EDD.

{% hint style="info" %}
**Примечание 1:**\
**METHOD** применяется для:

* Проверки согласованности данных
* Алгоритмов преобразования между переменными EDD
* Запросов подтверждения от пользователя
* Специфических взаимодействий с устройством (например, стандартные процедуры для:
* Технического обслуживания
* Калибровки
* Ввода в эксплуатацию
* Конфигурации
{% endhint %}

{% hint style="info" %}
**Примечание 2: Methods НЕ предназначены для:**

* Расширения функциональности устройства
* Генерации новых данных устройства
{% endhint %}

{% hint style="danger" %}
**Важно:**\
Methods, создающие процессные или статусные данные (которые должны предоставляться самим устройством), нарушают согласованное использование данных вне приложений EDD.
{% endhint %}

{% code title="Lexical structure" %}
```c
METHOD identifier [(parameter-declaration [, parameter-declaration]*)]? {
    LABEL string-value;
    HELP string-value;
    ACCESS [OFFLINE | ONLINE]; // недоступно в HART
    CLASS method-class-specifier [& method-class-specifier]*;
    DEFINITION {compound-statement}
    HANDLING handling-specifier;
    PRIVATE boolean-specifier;
    TYPE method-type-specifier;
    VALIDITY boolean-specifier;
    VISIBLITY boolean-specifier;
}
parameter-declaration := parameter-type [&]?identifier[[]]?
```
{% endcode %}

В таблице ниже описаны типы параметров методов.

<table><thead><tr><th width="148">Значение атрибута</th><th>Описание</th></tr></thead><tbody><tr><td><code>char</code></td><td>METHOD ожидает параметр типа <code>char</code></td></tr><tr><td><code>unsigned char</code></td><td>METHOD ожидает параметр типа <code>unsigned char</code></td></tr><tr><td><code>short</code></td><td>METHOD ожидает параметр типа <code>short</code></td></tr><tr><td><code>unsigned short</code></td><td>METHOD ожидает параметр типа <code>unsigned short</code></td></tr><tr><td><code>int</code></td><td>METHOD ожидает параметр типа <code>int</code></td></tr><tr><td><code>unsigned int</code></td><td>METHOD ожидает параметр типа <code>unsigned int</code></td></tr><tr><td><code>long</code></td><td>METHOD ожидает параметр типа <code>long</code></td></tr><tr><td><code>unsigned long</code></td><td>METHOD ожидает параметр типа <code>unsigned long</code></td></tr><tr><td><code>long long</code></td><td>METHOD ожидает параметр типа <code>long long</code></td></tr><tr><td><code>unsigned long long</code></td><td>METHOD ожидает параметр типа <code>unsigned long long</code></td></tr><tr><td><code>float</code></td><td>METHOD ожидает параметр типа <code>float</code></td></tr><tr><td><code>double</code></td><td>METHOD ожидает параметр типа <code>double</code></td></tr><tr><td><code>time_t</code></td><td>METHOD ожидает параметр типа <code>time_t</code></td></tr><tr><td><code>DD_STRING</code></td><td>METHOD ожидает параметр типа <code>DD_STRING</code></td></tr><tr><td><code>DD_ITEM</code></td><td>METHOD ожидает параметр типа <code>DD_ITEM</code></td></tr></tbody></table>

## 12.2 ACCESS

Недоступно в HART

## 12.3 CLASS

Атрибут **CLASS** определяет воздействие **METHOD** на устройство. Используется приложением **EDD** для реализации уровней доступа или организации отображения **METHODs.**

{% code title="Lexical structure" overflow="wrap" %}
```c
CLASS method-class-specifier [& method-class-specifier]* ;
method-class-specifier := [ALARM | ANALOG_OUTPUT | BACKGROUND | COMPUTATION | CONTAINED | CORRECTION | DEVICE | DIAGNOSTIC | DISCRETE | FREQUENCY | HART | INPUT | LOCAL_DISPLAY | OPERATE | OUTPUT | SERVICE | SPECIALIST | TUNE]
```
{% endcode %}

#### **Технический перевод таблицы атрибутов CLASS**

<table data-full-width="true"><thead><tr><th width="178">Атрибут</th><th width="570">Описание</th><th width="283">Профили связи</th></tr></thead><tbody><tr><td><strong>ALARM</strong></td><td>METHOD связан с аварией (например, установка аварийных пределов, индикация статуса аварии и т.д.)</td><td>CS, FF, GPE, ISA100, PB, PN</td></tr><tr><td><strong>ANALOG_OUTPUT</strong></td><td>METHOD является частью блока аналогового вывода</td><td>CS, GPE, HART, PB, PN</td></tr><tr><td><strong>BACKGROUND</strong></td><td><p>Класс BACKGROUND предназначен для улучшения симуляции устройства с использованием EDD. Во всех приложениях EDD, кроме симуляторов, этот класс должен игнорироваться. METHODS класса BACKGROUND не должны принадлежать другим CLASS.</p><p>CLASS BACKGROUND используется только в METHODS и указывает, что симулятор должен периодически выполнять METHOD. Все METHODS этого класса выполняются с периодом, указанным в VARIABLE background_period. </p><p>METHOD не выполняется во время обработки пакета (между получением запроса и отправкой ответа). Если период background_period истекает в этот интервал, METHOD выполняется сразу после генерации ответа.</p></td><td>HART</td></tr><tr><td><strong>COMPUTATION</strong></td><td>METHOD является частью вычислительного блока</td><td>CS, GPE, HART, PB, PN</td></tr><tr><td><strong>CONTAINED</strong></td><td>METHOD представляет физические характеристики устройства</td><td>CS, GPE, PB, PN</td></tr><tr><td><strong>CORRECTION</strong></td><td>METHOD поддерживает преобразователи в устройстве (установка пределов, демпфирование сигнала, калибровка и т.д.)</td><td>CS, GPE, HART, PB, PN</td></tr><tr><td><strong>DEVICE</strong></td><td>METHOD определяет физические характеристики устройства</td><td>CS, GPE, HART, PB, PN</td></tr><tr><td><strong>DIAGNOSTIC</strong></td><td>METHOD оценивает статус устройства</td><td>CS, FF, GPE, HART, ISA100, PB, PN</td></tr><tr><td><strong>DISCRETE</strong></td><td>METHOD поддерживает дискретные/цифровые входы-выходы устройства</td><td>CS, GPE, HART</td></tr><tr><td><strong>FREQUENCY</strong></td><td>METHOD поддерживает частотные входы-выходы устройства</td><td>CS, GPE, HART</td></tr><tr><td><strong>HART</strong></td><td>METHOD является частью блока HART, характеризующего интерфейс HART (только один блок HART)</td><td>CS, HART</td></tr><tr><td><strong>INPUT</strong></td><td>METHOD является частью входного блока (специальный вычислительный блок для преобразования единиц, масштабирования и демпфирования)</td><td>CS, FF, GPE, HART, ISA100, PB, PN</td></tr><tr><td><strong>LOCAL_DISPLAY</strong></td><td>METHOD является частью блока локального дисплея (переменные, связанные с локальным интерфейсом устройства)</td><td>CS, GPE, HART, PB, PN</td></tr><tr><td><strong>OPERATE</strong></td><td>METHOD используется для управления работой блока</td><td>CS, FF, GPE, PB, PN</td></tr><tr><td><strong>OUTPUT</strong></td><td>METHOD является частью выходного блока (значения параметров могут использоваться другими блоками)</td><td>CS, FF, GPE, ISA100, PB, PN</td></tr><tr><td><strong>SERVICE</strong></td><td>METHOD используется при проведении планового технического обслуживания</td><td>CS, FF, GPE, HART, ISA100, PB, PN</td></tr><tr><td><strong>SPECIALIST</strong></td><td><p>METHOD выполняется только специалистом. Для обычных пользователей METHOD с этим атрибутом недоступен. Если VALIDITY или VISIBILITY FALSE, METHOD не отображается.</p><p>Если атрибут не определен, METHOD выполняется.</p></td><td>CS, GPE, HART, ISA100, PB, PN</td></tr><tr><td><strong>TUNE</strong></td><td>METHOD используется для настройки алгоритма блока</td><td>CS, FF, GPE, ISA100, PB, PN</td></tr></tbody></table>

**Примечания по профилям связи:**

* **HART**: Рекомендуется использовать только SPECIALIST и BACKGROUND. Остальные классы поддерживаются для обратной совместимости.
* **FF, ISA100**: Должен быть ровно один класс INPUT или OUTPUT, плюс дополнительные поддерживаемые классы.

## 12.4 TYPE

**Атрибут TYPE** определяет тип возвращаемого значения. Ограничения использования для **HART:**

Атрибут **TYPE** для METHOD должен объявляться **только** для:

* METHOD, вызываемых исключительно из других METHODS
* **Запрещено:** ссылки из MENU на METHOD с объявленным TYPE

{% code title="Lexical structure" overflow="wrap" %}
```c
TYPE method-type-specifier;
method-type-specifier := [char | unsigned char | short | unsigned short | int | unsigned int | long | unsigned long | long long | unsigned long long | float | double | time_t | DD_STRING]
```
{% endcode %}

<table><thead><tr><th width="186">Атрибут</th><th>Описание</th></tr></thead><tbody><tr><td><code>char</code></td><td>METHOD возвращает значение типа <code>char</code></td></tr><tr><td><code>unsigned char</code></td><td>METHOD возвращает значение типа <code>unsigned char</code></td></tr><tr><td><code>short</code></td><td>METHOD возвращает значение типа <code>short</code></td></tr><tr><td><code>unsigned short</code></td><td>METHOD возвращает значение типа <code>unsigned short</code></td></tr><tr><td><code>int</code></td><td>METHOD возвращает значение типа <code>int</code></td></tr><tr><td><code>unsigned int</code></td><td>METHOD возвращает значение типа <code>unsigned int</code></td></tr><tr><td><code>long</code></td><td>METHOD возвращает значение типа <code>long</code></td></tr><tr><td><code>unsigned long</code></td><td>METHOD возвращает значение типа <code>unsigned long</code></td></tr><tr><td><code>long long</code></td><td>METHOD возвращает значение типа <code>long long</code></td></tr><tr><td><code>unsigned long long</code></td><td>METHOD возвращает значение типа <code>unsigned long long</code></td></tr><tr><td><code>float</code></td><td>METHOD возвращает значение типа <code>float</code></td></tr><tr><td><code>double</code></td><td>METHOD возвращает значение типа <code>double</code></td></tr><tr><td><code>time_t</code></td><td>METHOD возвращает значение типа <code>time_t</code></td></tr><tr><td><code>DD_STRING</code></td><td>METHOD возвращает значение типа <code>DD_STRING</code></td></tr></tbody></table>
