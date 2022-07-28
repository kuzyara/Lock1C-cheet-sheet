# Шпаргалка по блокировкам 1С

*Задача: наложение блокировок на регистр сведений*

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
--------------------------------------------------
Объектные блокировки: https://infostart.ru/public/543218/
* Пессимистические (явные, неявные)
* Оптимистические
