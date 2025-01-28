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
