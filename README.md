# FastAPI Validation Service

Главной задачей данного сервиса является валидация моделью входных данных.

## Зависимости для установки

- [Python 3.11](https://www.python.org/downloads/)
- [Pip](https://pypi.org/project/pip/)
- [Docker](https://www.docker.com/products/docker-desktop)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Функционал

Приложение имеет единственный endpoint, который применяет модель к полученным входным данным, а именно к строке.
Для проверки работоспособности и подробного знакомства с контрактом endpoint можно воспользоваться Swagger-UI, доступным по ссылке http://localhost:8000/docs, по умолчанию или OpenApi файлом, расположенному по адресу http://localhost:8000/openapi.json, после запуска приложения.

## Запуск

Запуск может быть осуществлен двумя способами:

- Автоматизированный
- Ручной

### Автоматизированный

Автоматизированный запуск приложения происходит в Docker контейнере. Для этого введите в консоли:
```bash
docker-compose up
```
Данный вид запуска является приоритетным, поскольку зависимости устанавливаются в виртульной среде и мало зависят от платформы, на который вы запускаете приложение.

### Ручной

Для ручного запуска необходимо установить сначала менеджер настроек poetry:
```bash
pip install poetry
```
Далее для создания виртуального окружения необходимо из корня проекта выполнить:
```bash
poetry shell
```
Установить зависимости:
```bash
poetry install
```
Запустить приложение:
```bash
poetry run python ./src/main.py
```

## Настройки

В приложении есть возможность изменять некоторые дефолтные настройки, передавая их, как переменные среды. Ниже приведен перечень с этими настройками.

| Ключ       | Описание                                                                                                                                                                                                                                                                                                                                     | Стандартное значение | Возможные значения |
|:------------------:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------:|:------------------:|
| URL_PREFIX | Префикс, добавляемый после хоста.                                                                                                                                                                                                                                                                                                            |       /api/v1        |        Any         |
| LOG_LEVEL  | Уровень, с которого показываются логи. Для более подробного тестирования рекомендуется уровень DEBUG                                                                                                                                                                                                                                         |         INFO         |    NOTSET, DEBUG, INFO,WARNING, ERROR, CRITICAL    |
| PROFILE    | Для различных уровней работы с приложением возможно создавать свои наборы настроек. Для этого в корне проекта необходимо создать env файл в следующем формате: .{PROFILE}.env  Передавая PROFILE в приложении загружается соответствующий env файл. Важно: приоритет переменных в env файле ниже приоритета переменных, переданных напрямую. |          -           |        Any         |

## Решения

1. Приложение выполненно с использованием фреймворка FastApi по ряду причин: 
- встроенная поддержка валидации данных
- Dependency Injection
- количество и простота внедрения модулей фреймворка
- легковесность
- производительность
- встроенная поддержка OpenApi
2. Архитектура приложения проектировалась со ссылкой на принципы DDD и SOLID. В приложении в отличие от принятых 3 слоев, разделение на 2 слоя, т.к. главная и единственная функция приложения не имеет состояния, то и уровня для взаимодействия с доменной моделью нету. По итогу есть слой user_interface для взаимодействия с клиентами при помощи разных каналов связи и слой use_case для выполнения бизнес логики, которая не имеет состояния.
3. Для возможности быстрой замены способа валидации, слабой связи между слоями приложения и простоты тестирования используется инверсия контроля за счет инъекции зависимости ValidationUseCaseInterface. 
4. Настройка роутов выполнена в одном файле routes, т.к. не хотелось делать архитектуру приложения тяжелее. При необходимости можно создать папку routes и добавлять endpoint туда.
5. Для контроля над ошибками реализован глобальный обработчик ошибок, благодаря чему клиенты, использующие сервис, могут обрабатывать полученные ошибки. Также упрощается процесс поиска ошибок, работа приложения становится более контролируемой, и добавление обработчиков новых ошибок.
6. В приложении выбраны и оптимально написаны логи под разные уровни логгирования. На INFO уровне получаем только необходимую информацию о результате, на уровне DEBUG детализированная информация для разработки и тестирования. Общий формат логов заменен на более информативный, чем стандартный (время, уровень записи, источник).
7. Приложение задокументировано с помощью OpenApi для прозрачности контрактов взаимодействия.
8. Приложение собирается в docker image и уборачивается docker-compose для простоты развертывания и возможности автоматизации.
9. В проект добавлены и настроены инструменты упрощающие поддержку приложения:
- Flake8 (линтер)
- Black (форматер)
- Mypy (статический типизатор)
- Pytest (юнит-тесты и интеграционные)

## CI/CD

Инструменты, обеспечивающие CI/CD, было принято не включать в процесс запуска, поэтому ими можно воспользоваться после запуска приложения в ручном режиме.
Выполнять команды необходимо в среде Virtualenv, созданный Poetry.

### Линтер Flake8

Для запуска линтера необходимо:
```bash
poetry run flake8
```

### Форматтер black

Для запуска форматтера необходимо:
```bash
poetry run black ./src
```

### Статический типизатор mypy

Для запуска типизатора необходимо:
```bash
poetry run mypy ./src
```

### Pytest

Для запуска тестов необходимо:
```bash
poetry run pytest
```

## Возможности улучшения

- Создание pipeline для автоматизации CI/CD.
- Создание infrasture layer при появлении доменной модели.
