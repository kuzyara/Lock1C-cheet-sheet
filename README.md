# Шпаргалка по блокировкам 1С

*Задача: наложение блокировок на регистр сведений*

см. также: [Прусаков Никита про Чтение и запись наборов записей.](Прусаков%20Никита%20про%201С.md)
* в репозитории 1С (https://github.com/1CEnterprise/MSQL-for-1C)
* в картинках (https://infostart.ru/public/629017/)
![alt text](https://github.com/kuzyara/Locks-cheet-sheet/blob/master/Блокировки.jpg)
Блокировки на **НаборЗаписей.Прочитать()** в 8.2 и в 8.3  работают по-разному:
* Например, в 8.2 у вас любое чтение будет ставить S-блокировку. Причем, чтение – это не просто Запрос.Выполнить(), но и Ссылка.Реквизит, Ссылка.ПолучитьОбъект() и т.д. 
* А в 8.3 без режима совместимости уже блокировок на чтение не будет. Таким образом, 8.3 безусловно выигрывает в плане параллельности работы.
* Ну а дальше начинается самое интересное – например, для конструкции НаборЗаписей.Прочитать() в управляемом режиме 8.2 у вас будет S-блокировка на сервере СУБД (это естественно). Но кроме этого также будет разделяемая блокировка на сервере 1С, причем это проявляется и в 8.2, и в 8.3. И главная проблема в том, что эта разделяемая блокировка у вас будет длиться до конца транзакции – пока транзакция не закончится, данные будут блокированы.
Поэтому рекомендация номер один – если набор записей вам нужен только для чтения, лучше использовать запрос, а не объектную модель. Тогда вы ничего блокировать не будете, а если и будете, то ненадолго.

Блокировки платформы:
* Р - РежимБлокировкиДанных.Разделяемый
* И - РежимБлокировкиДанных.Исключительный

Блокировки СУБД:
* Sch-S = Блокировка стабильности схемы. Гарантирует, что элемент схемы, такой как таблица или индекс, не будет удален до тех пор, пока сеанс связи удерживает блокировку стабильности схемы на данный элемент схемы;
* Sch-М = Блокировка изменения схемы. Должен поддерживаться любым сеансом связи, во время которого предполагается изменить схему данного ресурса. Гарантирует, что другие сеансы не имеют ссылок на обозначенный объект;
* S = Коллективная блокировка. Удерживающему сеансу предоставлен коллективный доступ к ресурсу;
* U = Блокировка обновления. Указывает блокировку обновления, полученную на ресурсы, которые со временем могут быть обновлены. Используется для предотвращения общей формы взаимоблокировки, которая возникает, когда множество сеансов блокируют ресурсы для потенциального обновления в последующее время;
* X = Монопольная блокировка. Удерживающему сеансу предоставлен исключительный доступ к ресурсу;
* IS = Блокировка с намерением коллективного доступа. Указывает намерение поместить S блокировки на некоторые подчиненные ресурсы в иерархии блокировок;
* IU = Блокировка с намерением обновления. Указывает намерение поместить U блокировки на некоторые подчиненные ресурсы в иерархии блокировок;
* IX = Блокировка с намерением монопольного доступа. Указывает намерение поместить X блокировки на некоторые подчиненные ресурсы в иерархии блокировок;
* SIU = Коллективная блокировка с намерением обновления. Указывает коллективный доступ к ресурсу с намерением получения блокировок обновления на подчиненные ресурсы в иерархии блокировок;
* SIX = Коллективная блокировка с намерением монопольного доступа. Указывает коллективный доступ к ресурсу с намерением получения монопольных блокировок на подчиненные ресурсы в иерархии блокировок;
* UIX = Блокировка обновления с намерением монопольного доступа. Указывает блокировку обновления ресурса с намерением получения монопольных блокировок на подчиненные ресурсы в иерархии блокировок;
* BU = Блокировка массового обновления. Используется для массовых операций;

![alt text](Избыточные%20блокировки.%20Подготовка%20к%20сдаче%20экзамена%201С_Эксперт%201-21%20screenshot.png)

### Объектные блокировки
--------------------------------------------------

> __Объектный способ__ (объектная техника) - доступ к данным посредством использования объектов __встроенного языка__. `ДокументСсылка.Товары`, `ПолучитьОбъект()`
>
> __Табличный доступ__ (табличная техника) - доступ к данным в «1С:Предприятии» с помощью __запросов к базе данных__, которые составляются на языке запросов. `Запрос.Выполнить()`
>
> _Подробнее см:  [Практическое пособие разработчика, занятие 13](https://github.com/kuzyara/Lock1C-cheet-sheet/blob/master/2024-04-17_15-55-33.png)_

#### 1. Метод `НаборЗаписей.Прочитать()` всегда устанавливает управляемую __разделяемую__ блокировку для следующих объектов:
 - набор записей регистра сведений,
 - набор записей регистра накопления,
 - набор записей регистра бухгалтерии,
 - набор записей регистра расчета,
 - набор записей перерасчета,
 - набор записей последовательности.<br />

 Пояснение: внутри метода `.Прочитать()` происходит неявная транзакция.
 
```bsl
Набор = РегистрыСведений.ВсеОбновленияНовостей.СоздатьНаборЗаписей();
Набор.Отбор.ВидОбновления.Установить("Получение новостей");
Набор.Прочитать(); // <- в ТЖ будет событие TLOCK
```

домашнее задание: 
> самостоятельно проверить устанавливают ли блокировки `.Остатки()`, `.СрезПоследних()`

#### 2. Платформа всегда устанавливает _объектную_ (не путать с транзакционными) оптимистическую блокировку при вызове метода `ПолучитьОбъект()`
_Пояснение: объектная оптимистическая блокировка - это когда текущий реквизит ВерсияДанных не совпадает с прочитанным, подробнее см. [Какие бывают блокировки](#какие-бывают-блокировки)_

```bsl
ТекОбъект = ДокументСсылка.ПолучитьОбъект(); // <- чтение свойства ВерсияДанных - "AAAAAACBPh8="
ТекОбъект.Записать(); // <- сравнение текущей версии данных из БД с прочитанной ранее
```
домашнее задание: 
> прочитать:
> - [Коротко о главном: Пессимистическая блокировка](https://its.1c.ru/db/pubessence/content/148/hdoc)
> - [Коротко о главном: Доступ к версии объекта во встроенном языке](https://its.1c.ru/db/pubessence#content:149:hdoc)

#### 3. Платформа НЕ устанавливает _управляемые_ блокировки при объектном чтении для следующих объектов:
 - констант и наборов констант;
 - справочников (например, используя методы `Справочники.ИмяСправочника.Выбрать()`, `СправочникВыборка.Следующий()`, `СправочникВыборка.ПолучитьОбъект()`, `СправочникСсылка.ПолучитьОбъект()` и т.д.);
 - документов (аналогично справочникам);
 - других объектов (планов видов характеристик, планов счетов и т.д.

Чтение в объектной технике обладает еще одной особенностью: оно в ряде случаев выполняется в неявной транзакции.

домашнее задание:
> 1) ответить на вопрос - какие блокировки устанавливает __СУБД__ при чтении данных запросом?
> 2) проверить в технологическом журнале - сколькими запросами платформа делает объектное чтение элемента справочника с табличной частью? почему их то 2, то 3?

#### 4. Глава 3.15. Особенности чтения в объектной модели 

У чтения в объектной модели есть три особенности.
1. Иногда ставятся управляемые блокировки.
2. Иногда чтение происходит в неявной транзакции.
3. Считываются все данные.

Разберем их по очереди

> ### 1. Иногда ставятся управляемые блокировки.
>    
> Платформой устанавливаются управляемые __разделяемые__ блокировки при чтении в объектной технике следующих видов объектов:
> - набор записей регистра сведений,
> - набор записей регистра накопления,
> - набор записей регистра бухгалтерии,
> - набор записей регистра расчета,
> - набор записей перерасчета,
> - набор записей последовательности.
>
> ### 2. Иногда чтение происходит в неявной транзакции.
> Чтение в объектной технике объектов, имеющих табличные части\*, будет выполняться в неявной транзакции. Например, запрос
> `ДокументПроведен = ДокументСсылка.Проведен;` выполнится в транзакции, если для документа данного типа в конфигураторе определена
> табличная часть, но этот же запрос будет выполняться вне транзакции, если табличных частей у документа нет.
> 
> \* _Анонсирован отказ от использования транзакций для этих целей в будущем, но есть ожидания,
> что на многих продуктивных системах еще на какое-то время сохранится существующее
> положение дел._
> 
> ### 3. Считываются все данные.
> Если посмотреть с помощью замера на отладчике, как выполняется следующий код, то можно увидеть, что основное время
> всегда тратится на выполнение первой строки:
> ```
> ДокументПроведен = ДокументСсылка.Проведен; // <- запрос к БД будет здесь
> ДокументДата = ДокументСсылка.Дата;
> ДокументНомер = ДокументСсылка.Номер;
> ```
> Если пройти код пошагово отладчиком и при этом посмотреть трассировку в профайлере SQL Server, можно увидеть, что запрос (если есть еще и табличная
> часть, то несколько запросов в транзакции) выполняется только на первой строке, при этом тексты запросов содержат имена всех реквизитов и все табличные части.
> На второй и на третьей строке запросы к базе уже не выполняются.
> 
> _Источник: Филлипов Е.В., Настольная книга Эксперта по технологическим вопросам, 2-е издание, стр. 56-57_

В версиях платформы до 8.3.7 (и в соответствующих режимах совместимости), чтение через объектную модель объектов, имеющих табличные части, осуществлялось в неявной транзакции.

Поэтому, если в СУБД предполагалось устоановка блокировки (автоматический режим управления блокировками или, например платформа в режиме совместимости 8.2.16 и MS SQL Server) - в формах списка документов могут возинкать конфликты блокировок, если в  событиях ПриПолученииСтроки() и в ПриВыводеСтроки() есть запросы через объектную модель, а время проведения документов превышает установленный для информационной базы лимит времени ожидания на блокировке (по умолчанию 20 сек).

В общем случае, разумеется, получение динамических списков при использовании обычных форм выполняется вне транзакции.


--------------------------------------------------
## Какие бывают блокировки
* Объектные блокировки: https://infostart.ru/public/543218/
    * Пессимистические (явные и неявные)
    * Оптимистические
      
### Объектная оптимистическая блокировка
Оптимистическая блокировка представляет собой проверку, которая выполняется перед записью объекта в базу данных. У объекта есть свойство «ВерсияДанных», которая вместе с объектом считывается из базы данных. Оптимистическая блокировка производит перед записью производит сравнение значения свойства «ВерсияДанных» объекта, который находится в оперативной памяти с значением свойства «ВерсияДанных» объекта находящийся в базе данных. Если значения свойства «ВерсияДанных» у объектов отличается, то оптимистическая блокировка запрещает запись объекта в базу данных и выдает сообщение об ошибке.

### Объектная пессимистическая блокировка (неявная, в форме)
Пессимистическая объектная блокировка предназначена для запрета изменений данных объекта, пока блокировка не будет снята. Система (с помощью соответствующих расширений формы объекта) автоматически устанавливает пессимистическую блокировку, в момент, когда пользователь пытается произвести изменение данных объекта. Если после этого другой пользователь, например, попытается выполнить редактирование того же объекта, ему будет выдано сообщение о том, что не удалось заблокировать объект. https://kostyanetsky.ru/notes/object-locks/

домашнее задание:
> открыть консоль кластера и проверить что при ~~открытии~~ измении формы появляется соответствующая запись о блокировке
### Объектная пессимистическая блокировка (явная, в коде)
* Без формы (обычные формы)
Методы «Заблокировать()», «Разблокировать()» и «Заблокирован()», используются для объектов базы данных, существуют только на сервере. Метод «Заблокирован()» не может быть использован для проверки, заблокирован ли вообще объект базы данных, например, другими пользователями. Для этого следует использовать метод «Заблокировать()» в попытке.
* Управляемы формы
Для работы с блокировками из управляемой формы без вызова сервера можено использовать методы: «ЗаблокироватьДанныеФормыДляРедактирования()» и «РазблокироватьДанныеФормыДляРедактирования()». Данные методы используются для блокировки или разблокировки данных основного реквизита формы. Контролируются флагом «Сохраняемые данные». Данный флаг определяет будет ли при интерактивном редактировании блокироваться данные основного реквизита, или нет.

Пессимистическая блокировка храниться в [сервисе блокировок](https://github.com/kuzyara/Lock1C-cheet-sheet/blob/master/images/lock-services-screen.png) DataEditLockService кластера 1С. Увидеть блокировку конкретного объекта можно в консоли кластера в разделе Кластреры -> Информационные базы -> Блокировки: 
<img src="https://github.com/kuzyara/Lock1C-cheet-sheet/blob/master/images/lock-object-screen.png" width=70% height=70%>

--------------------------------------------------
Какие бывают блокировки "На сервере 1С"?
* объектные - служба __DataEditLockService__
* транзакционные - служба __TransactionLockService__
--------------------------------------------------
Какие бывают "Объектные" блокировки:
* оптимистические
    * явные - `ДокументСсылка.ПолучитьОбъект().Записать()`
    * неявные! - `Ссылка.Товары.НайтиСтроки(Новый Структура("Склад", Склад))` https://infostart.ru/1c/articles/1461438/
* пессимистические  - `Объект.Заблокировать()`
--------------------------------------------------
Какие бывают "Транзакционные" блокировки:
* управляемые - на сервере 1С
* автоматические - на сервере СУБД
--------------------------------------------------
Какие бывают "Управляемые" блокировки:
* явные
```bsl
Блокировка = Новый БлокировкаДанных;
ЭлементБлокировки = Блокировка.Добавить("РегистрНакопления.ТоварыНаСкладах");
ЭлементБлокировки.УстановитьЗначение("Качество", Справочники.Качество.НайтиПоКоду("1"));
Блокировка.Заблокировать(); // <- для явных по-умолчанию устанавливается исключительная блокировка
```
* неявные
```bsl
Процедура ОбработкаПроведения()
    НаборЗаписей.Прочитать(); // <- тут наложится неявная разделяемая блокировка до конца транзакции
````
--------------------------------------------------
Какие бывают блокировки по режимам:
* на сервере 1с
     * управляемые - 8.3
* на сервере субд
    * автоматические - 8.2
--------------------------------------------------
 Какие бывают Управляемые (Транзакционные) блокировки:
* исключительные
    * явные - `ЭлементБлокировки.Режим = РежимБлокировкиДанных.Разделяемый;`
    * неявные - `Объект.Записать();`
* разделяемые
    * явные - `ЭлементБлокировки.Режим = РежимБлокировкиДанных.Исключительный;`
    * неявные - `НаборЗаписей.Прочитать();`
--------------------------------------------------
 Какие бывают Транзакционные блокировки:
* явные
    * на сервере 1с
    * на сервере СУБД
* неявные
    * на сервере 1с
    * на сервере СУБД

домашнее задание:
> привести пример для каждой из блокировок. *(гуглить track locks: MSSMS->Management->Extended Events)*
--------------------------------------------------
### Эскалация блокировок в ТЖ

* В mssql\
события Lock:Escalation. Укрупнение происходит в случае, если количество блокировок в определенном просмотре превышает 5000 http://www.gilev.ru/escalation/
* в платформе\
событие Locks без указания конкретных полей и значений. для 8.2 - более 20 тыс. на одно пространство, для 8.3 - более 100тыс. на одно пространство https://курсы-по-1с.рф/news/2017-12-22-optimization-1c-escalation-of-locks/
![alt text](Избыточные%20блокировки.%20Подготовка%20к%20сдаче%20экзамена%201С_Эксперт%207-37%20screenshot%20Блокировки%20СУБД%20Гранулярность.png)

### Ссылки
* https://www.youtube.com/watch?v=BsZAhSxUh9k - Избыточные блокировки. Подготовка к сдаче экзамена 1С:Эксперт
