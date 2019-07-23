# NooLite-PR1132-CLI
Форк [AlekseevAV NooLite-PR1132-CLI](https://github.com/AlekseevAV/NooLite-PR1132-CLI) для управления NooLite ethernet-шлюзом PR1132 в Homebridge через Http API. Для работы датчиков температуры необходимо поставить Homebridge плагин homebridge-temperature-cmd.

Пример добавление датчиков температуры:

"accessories": [
        {
            "accessory": "TemperatureCMD",
            "name": "Первый датчик температуры",
            "command": "sudo python /home/user/Homebridge-NooLite-PR1132-CLI/noolite_cli.py -hsns 0"
        },
         { "accessory": "TemperatureCMD", 
         "name": "Второй датчик температуры",
         "command": "sudo python /home/user/Homebridge-NooLite-PR1132-CLI/noolite_cli.py -hsns 0" }
]

### Описание

Аргументы CLI имеют такие же названия, как и в [API ethernet-шлюза PR1132](http://www.noo.com.by/assets/files/PDF/PR1132.pdf).
Добавлен два новый аргумента `sns` (json формат) и `hsns` (текстовый формат) для получения информации с датчиков. В его значении передается канал датчика информацию с которого нужно получить.

Аргумент   | Описание      | Расшифровка
---------- | ------------- | -----------
**-ch**    | Адрес канала  | Адрес канала, на который будет передаваться команда, значение от 0 до 31.
**-cmd**   | Команда       | Значение или номер команды
**-br**    | Абсолютная яркость в %, используется с командой=6 | Значение 0...100, используется только с командой 6. Для установки яркости на выбранном канале требуются только аргументы «ch», «cmd», «br». Наличия аргумента «fmt» при установке яркости из аргумента «br» не требуется.
**-fmt**   | Формат        | Значение 0...255. При передаче команды со значением 6 - значение «Формат»=1 (яркость – байт данных 0) или «Формат»=3 (яркость на каждый канал независимо - байт данных 0, 1, 2 *). Аргумент «fmt» необходим только при передаче данных вместе с аргументами («d0», «d1»,«d2»,«d3»).
**-d0**    | Байт данных 0 | Значение 0...255. При передаче команды со значением=6 и «Формат»=1 в данном байте содержится информация о яркости, которая будет установлена (значение в диапазоне 35...155). При значении 0 – свет выключится, при значении больше 155 – свет включится на максимальную яркость. При передаче команды со значением=6 и «Формат»=3 в данном байте содержится информация о яркости, которая будет установлена (значение в диапазоне 0...255) на канал R.
**-d1**    | Байт данных 1 | Значение 0...255. *При передаче команды со значением=6 и «Формат»=3 в данном байте содержится информация о яркости, которая будет установлена (значение в диапазоне 0...255) на канал G.
**-d2**    | Байт данных 2 | Значение 0...255. *При передаче команды со значением=6 и «Формат»=3 в данном байте содержится информация о яркости, которая будет установлена (значение в диапазоне 0...255) на канал B.
**-d3**    | Байт данных 3 | Значение 0...255, значение по умолчанию=0


### Установка

1. Склонировать репозиторий
2. Установить зависимости
3. Создать `conf_cli.yaml` на основе `conf_cli_sample.yaml`

```
$ git clone https://github.com/blog-razrabotchika/Homebridge-NooLite-PR1132-CLI.git
$ cd Homebridge-NooLite-PR1132-CLI
$ pip install -r requirements.txt
```

### Конфиг

При запуске CLI ищет файл `conf_cli.yaml` в директории расположения скрипта, откуда получает:
+ login - логин для авторизации в API
+ password - пароль для авторизации в API
+ api_url - полный адрес к API PR1132, к примеру: `http://192.168.2.200`


### Пример использования
```
# Включение силового блока, привязанного к 0 каналу
$ python noolite_cli.py -ch 0 -cmd 2
OK

# Выключение силового блока, привязанного к 0 каналу
$ python noolite_cli.py -ch 0 -cmd 0
OK

# Получение информации с датчика, привязанного к 0 каналу
# ответ приходит в json формате
$ python noolite_cli.py -sns 0
{"state": "Датчик привязан, ожидается обновление информации", "temperature": 21.1, "humidity": 56}

# Получение информации с датчика, привязанного к 0 каналу
# без лишнего текста, только числовое значение
$ python noolite_cli.py -hsns 0
21.1

# Задать RGB-контроллеру SD111-180, привязанному к 3 каналу соответствующую яркость 
# на каждый из цветовых каналов: d0 - красный, d1 - зеленый, d2 - синий
$ python noolite_cli.py -ch 3 -cmd 6 -fmt 3 -d0 247 -d1 255 -d2 247
```
