Примеры применения парсера
==========================
В данном разделе будут описываться практически реализованные примеры применения парсера.

Получение списка городов и населенных пунктов РФ
------------------------------------------------
Исходный xml-файл можно скачать отсюда http://gis-lab.info/qa/vmap0-settl-rus.html

**Задача** сконвертировать данные в формат JSON.

Особенностью структуры xml-файла является то, что все данные хранятся в атрибутах.

Итоговый код с применением xmlParser выглядит так

.. code-block:: python

	x=xmlParser('np.xml', 'xml')
	data=x.getData(['osm/node', 'osm/node/tag'], get_attrib=True)
	d={}
	f=open('a.txt','w')
	f.write('[\n')
	for i in data:
		if len(i[1])>2:
			if d:
				f.write(simplejson.dumps(d,  ensure_ascii=False, indent=4 * ' ').encode('utf-8')+',\n')
			d=dict(i[1])
		if len(i[1])==2:
			k,v=i[1].values()
			k=k.replace(':','_')
			d[k]=v
	f.write('\n]')
	f.close() 
