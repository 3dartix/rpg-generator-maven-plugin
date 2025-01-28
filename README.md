# RPG Generator Maven Plugin

Maven плагин для автоматизации генерации конфигурации BPMN процессов.

## Описание

RPG Generator - это Maven плагин, который автоматизирует процесс подготовки и конфигурации болванки BPMN процессов. Плагин анализирует BPMN схему, автоматически обрабатывает и конфигурирует workers и user tasks, генерируя готовую к деплою конфигурацию болванки процесса. Это значительно сокращает время на ручную настройку и минимизирует возможные ошибки при конфигурации.

### Основные возможности

- Автоматический анализ BPMN схем
- Конфигурация workers и user tasks с автоматической генерацией классов
- Генерация готовой к деплою конфигурации процесса
- Поддержка перевода русскоязычных названий (при подключении сервиса-переводчика)
- Автоматическая генерация JSON конфигурации для user tasks
- Автоматическое именование потоков и обработка ошибок

## Требования

- Java 17 или выше
- Maven 3.6.3 или выше
- BPMN схема, соответствующая контракту
- Camunda Modeler (рекомендуется для редактирования схем)

## Установка

Добавьте плагин в секцию `build/plugins` вашего `pom.xml`:

```xml
<plugin>
    <groupId>ru.pugart</groupId>
    <artifactId>rpg-generator</artifactId>
    <version>1.0.0</version>
</plugin>
```

## Использование

1. Подготовьте BPMN схему в соответствии с контрактом
2. Настройте конфигурацию плагина в вашем `pom.xml`
3. Запустите генерацию:
```bash
mvn rpg-generator:build
```

## Конфигурация

Пример конфигурации плагина в `pom.xml`:

```xml
<plugin>
    <groupId>ru.pugart</groupId>
    <artifactId>rpg-generator</artifactId>
    <version>1.0.0</version>

    <configuration>
        <bpmnDirectory>${project.basedir}/src/main/resources/bpmn</bpmnDirectory>
        <outputDirectory>${project.basedir}/src/main/resources/rpg-generate-output</outputDirectory>
        <translateServiceUrl>http://localhost:5000</translateServiceUrl>
        <classTemplatePath>${project.basedir}/src/main/resources/templates/template.mustache</classTemplatePath>
        <tabComponentsMap>
            <task>selectTariffRko</task>
            <company></company>
            <documents></documents>
            <products>selectTariffRko</products>
        </tabComponentsMap>
        <tabComponentsTitleMap>
            <task>Задача</task>
            <company>Компания</company>
            <documents>Документы</documents>
            <products>Продукты</products>
        </tabComponentsTitleMap>
    </configuration>

    <executions>
        <execution>
            <id>generate-bpmn-stubs</id>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Параметры конфигурации

| Параметр | Описание | По умолчанию |
|----------|----------|--------------|
| `bpmnDirectory` | Директория с исходными BPMN схемами | `${project.basedir}/src/main/resources/bpmn` |
| `outputDirectory` | Директория для сгенерированных файлов (схемы, классы, конфигурации) | `${project.basedir}/src/main/resources/rpg-generate-output` |
| `translateServiceUrl` | URL сервиса перевода (опционально) | Если не указан, выполняется транслитерация кириллицы в латиницу |
| `classTemplatePath` | Путь к шаблону Mustache для генерации классов воркеров | Встроенный шаблон по умолчанию |
| `tabComponentsMap` | Маппинг компонентов на вкладки интерфейса | Конфигурация блока `tabs` |
| `tabComponentsTitleMap` | Маппинг русскоязычных заголовков для вкладок | Конфигурация блока `tabs` |

### Описание маппингов

- `tabComponentsMap`: Определяет структуру и связи между компонентами на вкладках интерфейса
- `tabComponentsTitleMap`: Задает русскоязычные названия для компонентов интерфейса, которые будут отображаться в UI

### Выполнение

Плагин выполняется в фазе `build` и последовательно:
1. Анализирует BPMN схемы из указанной директории
2. Генерирует классы воркеров по шаблону
3. Создает конфигурации для user tasks
4. Формирует итоговую схему с правильными именами и связями

## Контракт BPMN схемы

### Sequence Flow и Default Flow
- В поле `general -> name` указывается строка в формате: `ключ=название`
- Пример: `notFound=Отказать`
- `default flow` выбирается автоматически из доступных потоков

### Service Task
- В поле `general -> name` указывается название сервисной задачи
- Генерируется класс воркера по шаблону: `название_процесса.ServiceTask.переведенное_название`
- Пример: `test-sub-process.ServiceTaskDataSearch`
- Блок `outputs` и исходящие flow автоматически получают суффикс `Result`
- Итог: `ServiceTaskDataSearchResult`

### User Task
- В поле `general -> name` указывается название пользовательской задачи
- Генерируется конфигурация по шаблону: `название_процесса.UserTask.переведенное_название`
- Пример: `test-sub-process.UserTaskCheckingTheDataFound`
- Блок `outputs` и исходящие flow получают суффикс `Result`
- Итог: `UserTaskCheckingTheDataFoundResult`
- Дополнительно создается JSON конфигурация для сервиса

### Error Events
- В поле `general -> name` указывается название ошибки
- Пример: `NOT_FOUND_ERROR`
- Автоматически заполняются блоки: `error -> global`, `name`, `code`

### Важное замечание
Если при деплое возникает ошибка валидации схемы:
1. Откройте сгенерированную схему в Camunda Modeler
2. Выделите всё (Ctrl+A)
3. Скопируйте (Ctrl+C)
4. Удалите содержимое (Ctrl+A, Delete)
5. Вставьте скопированное (Ctrl+V)
6. Сохраните (Ctrl+S)

Это заставит Modeler перерисовать схему и исправит возможные проблемы валидации.

## Зависимости

- Mustache (com.github.spullara.mustache.java:compiler:0.9.10)
- Maven Plugin API (org.apache.maven:maven-plugin-api:3.6.3)
- Maven Plugin Annotations (org.apache.maven.plugin-tools:maven-plugin-annotations:3.6.0)
- Commons IO (commons-io:commons-io:2.16.1)
- Jackson (com.fasterxml.jackson.core:jackson-databind:2.17.2)
- JDOM2 (org.jdom:jdom2:2.0.6)

## Простой сервис для перевода на python

Для использования функции перевода русскоязычных названий, необходимо настроить простой сервис-переводчик. Ниже приведена инструкция по его развертыванию с использованием Docker.

### Структура проекта

```
translate-service/
├── app.py
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

### Файлы сервиса

1. `app.py` - основной файл приложения:
```python
from flask import Flask, request
from googletrans import Translator

app = Flask(__name__)

@app.route('/')
def translate_query():
    text = request.args.get('text', '')
    translator = Translator()
    translation = translator.translate(text, src='ru', dest='en')
    return translation.text

def translate_russian_to_english(text):
    translator = Translator()
    translation = translator.translate(text, src='ru', dest='en')
    return translation.text

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

2. `Dockerfile`:
```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

3. `requirements.txt`:
```
Flask==2.1.1
Werkzeug==2.1.1
googletrans==4.0.0-rc1
```

4. `docker-compose.yml`:
```yaml
version: '3.9'

services:
  translate-service:
    build: .
    ports:
      - "5000:5000"
```

### Использование с RPG Generator

После запуска сервиса укажите его URL в конфигурации плагина:
```xml
<configuration>
    <translateServiceUrl>http://localhost:5000</translateServiceUrl>
</configuration>
``` 
