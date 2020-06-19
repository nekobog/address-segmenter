# Open Address Parser
Позволяет стандартизировать адреса, напечатанные от руки, и находить их соответствия в ФИАС. 
Поиск предлагает варианты адреса с ошибкой/отсутствием до 2 знаков, связано это с ограничениями elastic'а. 

## Требования для реализации
1.	60Гб дискового пространства
2.	Python 3 + pandas, simpledbf и elasticsearch
3.	Elasticsearch 7+
4.	Полная БД ФИАС в формате dbf (https://fias.nalog.ru/Updates.aspx)

## Установка
1. Скачать ФИАС (https://fias.nalog.ru/Updates.aspx) и распаковать

2. Запустить elastic и kibana

    docker-compose up 
3. Установить зависимости
    
    pip install -r requirements.txt
3. Загрузить ФИАС в elasticsearch. Это обычно занимает около 9 часов.

    python upload_fias.py --fiasdir Path/to/extracted/fias --delete
4. Теперь можно пользоваться методами из api.py. Главный метод там — standardize(string) и get_addr([strings])

    import api
   
    addr = "г. Москва, ул. Тверская, д.4"
    list_of_addrs = ["г. Москва, ул. Тверская, д.4", "Москва, Коровий вал 3]
    
    norm_addr = api.standardize(addr)
    list_norm_addrs = api.get_addr(list_of_addrs)

## Принцип работы
Задача: разбить одно поле (адреса) на несколько полей, таких как дом, улица, город итд
Так как все адреса принципиально не отличаются друг от друга, то набор этих полей уже заранее известен. Порядок их в строке может незначительно меняться, но обычно он один и тот же: Индекс, город, улица, дом, корпус, квартира. Каждое из этих полей заполняется совершенно разными значениями, которые практически никогда не пересекаются, поэтому словарный подход здесь будет как никогда актуален.
Вся идея свести задачу извлечения полей к задаче поиска адреса в базе данных.
При этом некоторые поля, такие как адрес и номер дома гораздо более надёжно извлекать эвристически, нежели поиском. 

Извлечение адресов происходит в следующие этапы:
1.	Детекция полей по шаблону
    *	Извлечение индекса
    *	Извлечение номера дома/корпуса
2.	Определение границ названия улицы/города
3.	Поиск улицы и города в БД ФИАС
4.	Поиск дома и корпуса для данной улицы в ФИАС

## Структура проекта
- api.py - методы для непосредственно использования проекта
- upload_fias.py - для первоначальной загрузки файлов в fias
- parsing.py - все методы для предобработки адреса, работают с самим текстом.
- tests.py - проверка качества
- ref/references.xlsx - референсная выборка для оценки качества

