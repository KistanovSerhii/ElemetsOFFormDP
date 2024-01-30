##### <a name="pageup"></a>

# ElemetsOfFormDynamicP

🗺️ 1С Elemets of Form with dynamic properties / Динамическая установка свойств элементам формы
Обработка позволяет программно задать значение любому свойству элемента формы.
(Ширина, Высота, Доступность, ТолькоПросмотр, Видимость, Шрифт и т.д.)

👀 ["Видео инструкиця"](https://youtu.be/bS7Gx8dLoow)

🚨 Если ваша цель именно в блокировки доступа тогда используйте более подходящий функционал БСП
["Запрет редактирования реквизитов объектов"](https://youtu.be/D5snZ2d2R7o)

📜 Область применения и конкретный кейс в видео я не озвучил, поэтому добавляю в описание:
Задачей было заполнение формы в строгой последовательности.
При открытии формы доступными были только определенные поля, которые необходимо было заполнить первыми.
Шрифт этих полей был выделен жирным и черным цветом. После ввода данных происходила автоматическая обработка,
и эти поля становились НЕдоступными, а их шрифт менялся на обычный с зеленым цветом.
Затем становился доступным новый набор полей с тем же форматированием (жирный, черный).
Все заполненные (пройденные) поля НЕдоступны, шрифт обычный, а цвет зеленый.
После заполнения всех полей появлялись команды "Принять заявку" или "Отказать"
в зависимости от введенных данных на форме.

# Разделы

+ [Почему не типовое решение или БСП](#step0); ➕
+ [Краткое описание вызова](#step1);  🟣
+ [Пример задачи](#step2); 🔘
+ [Внедрение](#step3); 🪚
+ [Если хотите вызвать без использования БСП](#step4); ➕
+ [Отдельно функция проверка групп доступа](#step5); ➕
+ [ВАЖНО! Требования к методу проверки группы доступа](#add0); 🔘

##### <a name="step0"></a> Почему не типовое решение или БСП [(начало)](#pageup);
❗ На youtube пример про доступность элементов формы, но для управления доступом лучше используйте БСП
["Запрет редактирования реквизитов объектов"](https://youtu.be/D5snZ2d2R7o)

⛔ Если продолжать говорить про доступность тогда "ВидимостьюПоРолям" или БСП функция "УстановитьСвойствоЭлементаФормы(ЭлементыФормы..."
управляют единичным элементом, а данная обработка добавляет немного удобства позволяя задать значения всем элементам формы
и отдельно еденичным, а также проверить вхождение в группу доступа
(НО повторюсь, для управления доступом используйте ["Запрет редактирования реквизитов объектов"](https://youtu.be/D5snZ2d2R7o) )
Как дополнение данную обработку можно применять в конфигурациях без БСП.

Вот метод БСП который выполняется для одного элемента:
 ```
 // ОбщегоНазначенияКлиентСервер.УстановитьСвойствоЭлементаФормы(ЭтаФорма.Элементы, "Наименование", "Доступность", Истина);
 Процедура УстановитьСвойствоЭлементаФормы(ЭлементыФормы, ИмяЭлемента, ИмяСвойства, Значение) Экспорт	
	ЭлементФормы = ЭлементыФормы.Найти(ИмяЭлемента);
	Если ЭлементФормы <> Неопределено И ЭлементФормы[ИмяСвойства] <> Значение Тогда
		ЭлементФормы[ИмяСвойства] = Значение;
	КонецЕсли;	
КонецПроцедуры
```

##### <a name="step1"></a> Краткое описание вызова [(начало)](#pageup);

	1. форма - СсылкаНаБлокируемуюформу,
	2. свойстваЭлементов - Массив ИЗ Структура (имя элемента, свойство и его значение которые будут установлены)
	3.1 свойстваЭлементовОбщее - Структура описывает одно свойство и знч которое будет задано ВСЕМ элементам формы!

Обработка.УстановитьСвойствоЭлементаФормы(форма, свойстваЭлементов, свойстваЭлементовОбщее, видыИТипы);

	3.2 свойстваЭлементовОбщее - например всем элементам устанавливаем "ТолькоПросмотр" Истина, а
	свойстваЭлементов - добавляем элемент "Наименование" и "Родитель" которым устанавливаем свойство "ТолькоПросмотр" в Ложь
	и таким образом мы запрещаем редактировать ВСЕ элементы кроме "Наименование" и "Родитель"!

 	4. видыИТипы - это Соответствие (не обязательный параметр), которое перечислят какие типы элементов формы обрабатывать.
  	Если не передавать тогда будет как видыИТипы = внешняяОбработкаОбъект.НаборВидовИТиповЭлементовФормы();
   	*Например: ВидПоляФормы.ПолеВвода, ВидКнопкиФормы.КнопкаКоманднойПанели, ВидПоляФормы.ПолеФлажка и т.д.

##### <a name="step2"></a> Пример задачи [(начало)](#pageup);

👥 Пользователи которые не входят в группуДоступа "РасширенныйДоступСправочникНоменклатура"
должны получить доступ на редактирование исключительно к реквизиту Наименование и команда "Записать и закрыть"
, а те пользователи которые входят в группу "РасширенныйДоступСправочникНоменклатура" к ним ограничения применятся не должны!

##### <a name="step3"></a> Внедрение [(начало)](#pageup);

(🕵 Рекомендую использовать с проверкой вхождения в ГруппыДоступа см. код Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы)

1. Открывает Модуль формы в основной конфигурации (если редактирование разрешено) или в расширения (если редактирование основной конфигурации запрещено)
2. Переходим в событие "ПриСозданииНаСервере"
3. Копируем (Ctrl + C -> Ctrl + V) код:

```
#Область НовыеЗнчДляСвойствЭлементовФормы

		#Область ПолучениеВнешнейОбработки_СвойствЭлементовФормы
		внешняяОбработкаИмя    = "ДинамическоеСвойствоЭлементаФормы";
		внешняяОбработкаСсылка = Справочники.ДополнительныеОтчетыИОбработки.НайтиПоРеквизиту("ИмяОбъекта", внешняяОбработкаИмя);
		внешняяОбработкаОбъект = ДополнительныеОтчетыИОбработки.ОбъектВнешнейОбработки(внешняяОбработкаСсылка);
		#КонецОбласти

		#Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы
		мГруппыДоступа = Новый Массив;
		мГруппыДоступа.Добавить("РасширенныйДоступСправочникНоменклатура");
		доступРазрешен = внешняяОбработкаОбъект.ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа);
		#КонецОбласти

		// Описываете каким элементам формы какое значение хотите установить +++
		свойстваЭлементовОбщее = Новый Структура("Свойство,Значение", "Доступность", Ложь);

		свойстваЭлементов = Новый Массив; 
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ФормаЗаписатьИЗакрыть", "Доступность", доступРазрешен) );
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "Наименование", "Доступность", Истина) );
		// Описываете каким элементам формы какое значение хотите установить +++

		#Область Выполнение_СвойствЭлементовФормы
		Если НЕ доступРазрешен Тогда
			внешняяОбработкаОбъект.УстановитьСвойствоЭлементаФормы(ЭтаФорма, свойстваЭлементов, свойстваЭлементовОбщее);
			// текстСообщения = НСтр("ru = 'Вам ограничен доступ см. группу доступа ""РасширенныйДоступСправочникНоменклатура"" '");
			// ОбщегоНазначения.СообщитьПользователю(текстСообщения);
   		КонецЕсли
		#КонецОбласти

#КонецОбласти
```

##### <a name="step4"></a> Если хотите вызвать без использования БСП [(начало)](#pageup);

![image](https://github.com/KistanovSerhii/ElemetsOfFormDynamicP/assets/28355711/901217c0-3499-430a-be13-163d2a5f5937)
➕ Можно добавить внешнюю обработку в ветку конфигурации к типу "Обработки" и использовать как:
внешняяОбработкаОбъект = Обработки.ДинамическоеСвойствоЭлементаФормы.Создать();

Тогда полный код будет таким:

```
#Область НовыеЗнчДляСвойствЭлементовФормы
  
		#Область ПолучениеВнешнейОбработки_СвойствЭлементовФормы
		внешняяОбработкаОбъект = Обработки.ДинамическоеСвойствоЭлементаФормы.Создать();
		#КонецОбласти

		#Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы
		мГруппыДоступа = Новый Массив;
		мГруппыДоступа.Добавить("РасширенныйДоступСправочникНоменклатура");
		доступРазрешен = внешняяОбработкаОбъект.ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа);
		#КонецОбласти

		// Описываете каким элементам формы какое значение хотите установить +++
		свойстваЭлементовОбщее = Новый Структура("Свойство,Значение", "Доступность", Ложь);

		свойстваЭлементов = Новый Массив; 
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ФормаЗаписатьИЗакрыть", "Доступность", Истина) );
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "Наименование", "Доступность", Истина) );
		// Описываете каким элементам формы какое значение хотите установить +++

		#Область Выполнение_СвойствЭлементовФормы
		Если НЕ доступРазрешен Тогда
			внешняяОбработкаОбъект.УстановитьСвойствоЭлементаФормы(ЭтаФорма, свойстваЭлементов, свойстваЭлементовОбщее);
   		КонецЕсли
		#КонецОбласти

#КонецОбласти
```

##### <a name="step5"></a> Отдельно функция проверка групп доступа [(начало)](#pageup);

🤔 Если вы не собираетесь использовать данную обработку для проверки вхождения в группу доступа
тогда иожете использовать функцию проверки отдельно, код ниже:

```

// мГруппыДоступа - Массив ИЗ Строка - Наименование ГруппыДоступа
//
// Возврат Булево
&НаСервере
Функция ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа) Экспорт

УстановитьПривилегированныйРежим(Истина);
Запрос = Новый Запрос;
Запрос.Текст = 
"Выбрать 
|
Пользователь 
|ИЗ 
|
Справочник.ГруппыДоступа.пользователи
|Где
|
Пользователь = &ТекущийПользователь
|
И Ссылка.Профиль.Наименование В (&ИмяПрофиля)";
Запрос.УстановитьПараметр("ТекущийПользователь",
ПараметрыСеанса.ТекущийПользователь);
Запрос.УстановитьПараметр("ИмяПрофиля", мГруппыДоступа);
Выборка = Запрос.Выполнить().Выбрать();
Возврат (Выборка.Следующий() И Выборка.Пользователь <> "");
КонецФункции

// Используем так:
мГруппыДоступа = Новый Массив;
мГруппыДоступа.Добавить("ИмяСозданнойВамиГруппыДоступа");
доступРазрешен = ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа);

```

# ВАЖНО:

##### <a name="add0"></a> ВАЖНО [(начало)](#pageup); 

 🔘 Если будешь использовать метод проверки вхождения предоставленный текущей обработкой и в Твоей
конфигурации есть пользователи без прав чтения спр. "ГруппыДоступа" Тогда создай роль которая
дает права на чтения справочника "ГруппыДоступа" и создай ГруппуДоступа+Профиль которому назначь эту роль
после чего добавь всех пользователей в эту группу (что бы они могли выполнять чтение спр. "ГруппыДоступа")
![image](https://github.com/KistanovSerhii/ElemetsOfFormDynamicP/assets/28355711/94f0b5cf-d524-4650-8e38-5c1a267c7237)


 🔘 При использовании через расширение - обязательно снимите флаг "Безопасный режим, имя профиля безопасности"
![image](https://github.com/KistanovSerhii/ElemetsOfFormDynamicP/assets/28355711/7a0d51e4-fb60-4885-857a-61993c5aa62b)


# Дополнительная информация

❗ В расширении нельзя использовать привилегированный режим см. https://its.1c.ru/db/pubextensions/content/48/hdoc
(в расширении не выполняется как флаг общего модуля так и глобальный метод УстановитьПривилегированныйРежим)
 
