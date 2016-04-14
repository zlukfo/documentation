*********************
Парсинг базы вакансий
*********************

В данном разделе по шагам рассмотрен практический пример парсинга **большого xml файла**.

:Задача: Необходимо закачать в таблицу БД Postgresql Общероссийскую базу вакансий, сохраненную в формате xml и доступную по адресу http://data.gov.ru/opendata/7710538364-vacansii

:Исходные данные: В распакованнном виде файл xml занимает 842,3 Мб.

**Полученные результаты**

* Количество полей таблицы - 19
* Количество записей - 320 479
* Общее время парсинга и закачки данных в таблицу БД Postgresql - 29 мин.

Алгоритм
--------
1. Определяет структуру данных источника xml.

В большинстве своем архивные файлы xml имеют не сильно разветвленную структуру и ассоциативные имена тегов. Поэтому анализировать структуру данных в файлах рекомендуется графическим способом.

Поскольку наш файл имеет внушительный размер, очевидно, он содержит много записей одинаковой структуры. Поэтому анализировать файл целиком для построения структуры данных не имеет смысла. Выполним построение структуры данных на примере первых 1000 элементов

.. code-block:: python

    x=xmlParser('OBV_full.xml')
    x.getTree(count=1000)
    x.saveTree2svg(width=2000)


Получили вот такую структуру.___


Чтобы убедиться, что полученная нами структура является полной, повторим построение дерева, увеличив количество тегов до 10000


.. code-block:: python
    :emphasize-lines: 2
    
    x=xmlParser('OBV_full.xml')
    x.getTree(count=1000)
    x.saveTree2svg(width=2000)

Как видим, структура не изменилась. Зничит показанное на рисунке дерево полностью отражает структуру данных xml источника.

2. Указываем пути к данным

Теги элементов xml файла имеют как правило информативные имена, поэтому из рисунка легко понять какие данные хранит каждый элемент. Кроме того, по рисунку легко построить пути для каждого нужного элемента. В нашем примере мы будем извлекать данные из всех элементов

.. code-block:: python

    path2Data=['source/vacancies/vacancy/company/name',
			   'source/vacancies/vacancy/company/hr-agency',
			   'source/vacancies/vacancy/company/descrip',

			   'source/vacancies/vacancy/addresses/address/location',
			   'source/vacancies/vacancy/addresses/address/lng',
			   'source/vacancies/vacancy/addresses/address/lat',

			   'source/vacancies/vacancy/requirement/education',
			   'source/vacancies/vacancy/requirement/qualification',
			   'source/vacancies/vacancy/requirement/experience',

			   'source/vacancies/vacancy/creation-date',
			   'source/vacancies/vacancy/salary',
			   'source/vacancies/vacancy/currency',
			   'source/vacancies/vacancy/category/industry',
			   'source/vacancies/vacancy/job-name',
			   'source/vacancies/vacancy/employment',
			   'source/vacancies/vacancy/schedule',
			   'source/vacancies/vacancy/description',
			   'source/vacancies/vacancy/duty',
			]

3. Создаем генератор извлекаемых данных

.. code-block:: python

    result=x.getData(path2Data, type_response='json')

4. Сохраняем данные в таблицу БД

.. code-block:: python

        import psycopg2
        conn = psycopg2.connect("dbname=vacancy user=admin password=123456")
	conn.autocommit=True
	cur = conn.cursor()
	for rec in result:
		fields_name='('+', '.join(['"'+i.split('/')[-1]+'"' for i in rec.keys()])+')'
		templ='('+(ur'%s, '*len(rec))[:-2]+')'
		query= "INSERT INTO pub_simple %s VALUES %s" % (fields_name, templ)
		cur.execute(query, tuple(rec.values()))

Вот и все! Все данные в нашей таблице.