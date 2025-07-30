# Разработка системы конфигураций (модуль Config) 15.07.2025
# БЛОК УПРАВЛЕНИЯ ПРАМЕТРАМИ
## 💾 РАБОТА В РАЗНЫХ КОНТЕКСТАХ

### **Custom GPT / Claude Projects (без памяти между сообщениями)**

````yaml
User: "Установи точный режим"# БЛОК УПРАВЛЕНИЯ ПАРАМЕТРАМИ CONFIG

## 📊 ТРИ УРОВНЯ ИСПОЛЬЗОВАНИЯ

```yaml
1. МЫ → создаем ФРЕЙМВОРК LLM NATIVE
2. РАЗРАБОТЧИК → создает ПРОЕКТ на базе фреймворка  
3. КОНЕЧНЫЙ ПОЛЬЗОВАТЕЛЬ → использует проект разработчика
````

Этот блок описывает универсальные правила работы с параметрами на всех уровнях.

## 📋 СТРУКТУРА РАЗДЕЛА ПАРАМЕТРОВ

```yaml
Config:
  # 1. ОПРЕДЕЛЕНИЯ ПАРАМЕТРОВ - что это и для чего
  DefinitionParameters:
    temperature:
      description: "Определяет вариативность/креативность ответов LLM"
      type: "float"
      range: [0.0, 2.0]
      default: 0.7
      
    max_tokens:
      description: "Максимальное количество токенов в ответе"
      type: "integer"  
      range: [1, 8000]
      default: 4000
      
    top_p:
      description: "Nucleus sampling - контроль разнообразия выбора токенов"
      type: "float"
      range: [0.0, 1.0]
      default: 0.9

  # 2. ПЕРЕМЕННЫЕ - промежуточный слой для гибкого управления
  VariableParameters:
    Param_temperature: 0.5      # Используем круглое значение
    Param_max_tokens: 4000      # Круглое число
    Param_top_p: 0.9           # Один знак после запятой
    
  # 3. ПАРАМЕТРЫ - фактические значения для использования
  Parameters:
    # Глобальные параметры (по умолчанию для всех)
    Global:
      temperature: "@Param_temperature"  # Ссылка на переменную
      max_tokens: "@Param_max_tokens"
      top_p: "@Param_top_p"
      
    # Параметры модулей (переопределяют глобальные)  
    Module:
      AnalysisModule:
        temperature: 0.3               # Прямое присвоение
        max_tokens: "@Param_max_tokens" # Использует глобальную переменную
        
      CreativeModule:
        temperature:
          formula: "1.5 * @Param_temperature"  # Простой множитель
        max_tokens: 6000                      # Круглое число
        
      PrecisionModule:
        temperature: "@Param_temperature"      # Прямая ссылка
        max_tokens:
          formula: "2.0 * @Param_max_tokens"  # Удвоение

  # 4. RUNTIME СОСТОЯНИЕ - текущие значения и изменения
  RuntimeState:
    # Текущие значения переменных (могут отличаться от базовых)
    current_variables:
      # Пусто при старте - заполняется при изменениях
      # Пример после изменения:
      # Param_temperature:
      #   value: 0.3          # Текущее значение
      #   base_value: 0.5     # Исходное из VariableParameters
      #   changed_at: "14:30"
      #   changed_by: "UserCommand"
        
    # История всех изменений для отслеживания и отладки
    change_log:
      # Заполняется автоматически при любых изменениях
      # Пример записи:
      # - timestamp: "14:30:00"
      #   type: "variable_change"
      #   target: "Param_temperature"
      #   old_value: 0.5
      #   new_value: 0.3
      #   reason: "Пользователь запросил точный режим"
```

## 💡 ДВА МЕТОДА ИЗМЕНЕНИЯ ПАРАМЕТРОВ

### 1️⃣ **ПРЯМОЕ ПРИСВОЕНИЕ (для алгоритмов)**

```yaml
# Используется ВНУТРИ сценариев, модулей, pipeline
# Работает в рамках одного выполнения
Config.Parameters.Global.temperature = 0.1
Config.Parameters.Module.AnalysisModule.temperature = 0.3
```

**Когда использовать:** Автоматическая адаптация внутри логики проекта

### 2️⃣ **STATE UPDATE (для сохранения между сообщениями)**

```yaml
# Формат STATE UPDATE - СТРОГО СОБЛЮДАТЬ:
📌 CONFIG STATE UPDATE #[номер]:
changes:
  - path: "точный.путь.к.параметру"
    old: текущее_значение
    new: новое_значение
timestamp: "ISO формат даты"
expires: "session" | "30m" | "1h" | "never"
applied_by: "имя_команды_или_модуля"
reason: "причина изменения"
```

**Когда использовать:**

- В командах (Command) для пользователя
- При ответе на запрос изменить параметры в чате
- Когда нужно сохранить изменения между сообщениями

**Важно:**

- Номера STATE UPDATE должны быть последовательными (#001, #002...)
- Всегда указывать old и new значения
- Формат блока должен быть точно как указано

## 🔧 ТРИ СПОСОБА ЗАДАНИЯ ЗНАЧЕНИЙ

### 1️⃣ **Прямое присвоение**

```yaml
Module:
  AnalysisModule:
    temperature: 0.3  # Фиксированное значение, не зависит от переменных
```

**Когда использовать:** Модуль всегда должен работать с конкретным значением

### 2️⃣ **Относительное значение (формула)**

```yaml
Module:
  AdaptiveModule:
    temperature:
      formula: "0.5 * @Param_temperature"  # Половина от глобального
```

**Когда использовать:** Модуль должен адаптироваться к глобальным изменениям **Правило:** Используйте только простые множители (0.5, 1.0, 1.5, 2.0)

### 3️⃣ **Прямая ссылка на переменную**

```yaml
Module:
  StandardModule:
    temperature: "@Param_temperature"  # @ означает ссылку на переменную
```

**Когда использовать:** Модуль должен точно следовать глобальным настройкам **Важно:** Символ @ обязателен для обозначения ссылки на переменную

## 💡 ДИНАМИЧЕСКОЕ ИЗМЕНЕНИЕ ПАРАМЕТРОВ

### **Изменение глобальных параметров**

```yaml
# Прямое изменение глобального параметра
Config.Parameters.Global.temperature = 0.2

# Эффект: ВСЕ модули без явного переопределения используют 0.2
```

### **Изменение параметров модуля**

```yaml
# Изменить для конкретного модуля
Config.Parameters.Module.CreativeModule.temperature = 1.8

# Эффект: ТОЛЬКО CreativeModule использует 1.8
```

### **Изменение переменной**

```yaml
# Изменить базовую переменную
Config.VariableParameters.Param_temperature = 0.3

# Эффект: 
# - Global.temperature обновится до 0.3
# - Модули с формулами пересчитаются
# - Модули с прямым присвоением НЕ изменятся
```

## 📊 ПРАВИЛО ПРИОРИТЕТОВ

При запросе параметра система проверяет в порядке:

1. **Module параметр** - если определен для модуля
2. **Global параметр** - если нет в Module
3. **Default из Definition** - если нигде не задан

## 🔄 АВТОМАТИЧЕСКОЕ ВЫЧИСЛЕНИЕ

### **Правила работы с формулами:**

```yaml
# ФОРМАТ ФОРМУЛЫ:
temperature:
  formula: "[множитель] * @[переменная]"
  
# РАЗРЕШЕННЫЕ МНОЖИТЕЛИ:
✅ 0.5 (половина)
✅ 1.0 (равно)
✅ 1.5 (полтора)
✅ 2.0 (удвоение)
✅ 3.0 (утроение)

# ЗАПРЕЩЕНО:
❌ Сложные числа (0.33, 0.67, 1.33)
❌ Сложение/вычитание (0.5 * @Param + 0.3)
❌ Несколько операций
```

### **Процесс вычисления:**

```yaml
1. Config.VariableParameters.Param_temperature = 0.4

2. Система автоматически:
   - Находит все формулы с @Param_temperature
   - Вычисляет с показом процесса:
     "CreativeModule: 1.5 * 0.4 = 0.6"
   - Округляет до 1-2 знаков: 0.6
   - Обновляет значения во всех местах использования
   - Логирует изменение

# LLM ОБЯЗАНА показать вычисление:
"Вычисляю: 1.5 * 0.4 = 0.6"
```

## 📝 ПРАВИЛА ДЛЯ LLM

### **ОБЯЗАТЕЛЬНОЕ ПРАВИЛО №1 - ВЫБОР МЕТОДА ИЗМЕНЕНИЯ**

```yaml
ОПРЕДЕЛИ КОНТЕКСТ И ВЫБЕРИ МЕТОД:

1. ВНУТРИ АЛГОРИТМА (Scenario, Module, Pipeline):
   → Используй ПРЯМОЕ ПРИСВОЕНИЕ
   → Config.Parameters.Global.temperature = 0.1
   
2. В КОМАНДЕ ДЛЯ ПОЛЬЗОВАТЕЛЯ (Command):
   → Используй STATE UPDATE
   → Выведи блок CONFIG STATE UPDATE в ответ
   
3. ПРИ ОТВЕТЕ НА ЗАПРОС ПОЛЬЗОВАТЕЛЯ В ЧАТЕ:
   → Если просят изменить параметры → STATE UPDATE
   → Если выполняешь алгоритм → прямое присвоение

ПРИМЕРЫ:
# В сценарии (прямое):
SCENARIO::AdaptiveAnalysis:
  ЕСЛИ critical_data:
    Config.Parameters.Global.temperature = 0.1

# В команде (STATE UPDATE):
COMMAND::SetPrecision:
  trigger: "/precision"
  command:
    → Вывести CONFIG STATE UPDATE:
      Param_temperature: 0.1
```

### **ОБЯЗАТЕЛЬНОЕ ПРАВИЛО №2 - ТОЧНОСТЬ ВЫЧИСЛЕНИЙ**

```yaml
При создании Config с формулами СТРОГО соблюдать:

✅ РАЗРЕШЕНО (высокая точность):
- Множители: 0.5, 1.0, 1.5, 2.0, 3.0
- Переменные: использовать круглые значения (0.1, 0.5, 0.7)
- Формулы: только ОДНО действие (2.0 * @Param)
- Округление: ВСЕГДА до 1-2 знаков

❌ ЗАПРЕЩЕНО (низкая точность):
- Множители: 0.33, 0.67, 1.33, 2.75
- Сложные формулы: (0.5 * @Param + 0.3)
- Длинные дроби: 0.12345
- Без округления

ПРИМЕРЫ ПРАВИЛЬНЫХ ФОРМУЛ:
- "0.5 * @Param_temperature"    # Половина
- "2.0 * @Param_max_tokens"     # Удвоение  
- "1.5 * @Param_temperature"    # Полуторное

При вычислении ВСЕГДА показывать:
"Формула: 0.5 * @Param_temperature
 Значение: 0.5 * 0.7 = 0.35
 Округлено: 0.4"
```

### **ОБЯЗАТЕЛЬНОЕ ПРАВИЛО №3 - ЧТЕНИЕ ПАРАМЕТРОВ**

```yaml
При чтении параметров ВСЕГДА:

1. В АЛГОРИТМАХ (Scenario, Module, Pipeline):
   → Используй Config.get(param_name, module_name)
   → Это вернет актуальное значение с учетом приоритетов

2. В CUSTOM GPT / CLAUDE PROJECTS:
   → СНАЧАЛА проверь историю на CONFIG STATE UPDATE
   → ЗАТЕМ примени все изменения по порядку
   → ИСПОЛЬЗУЙ финальное значение

3. ПРИОРИТЕТ ЗНАЧЕНИЙ (от высшего к низшему):
   → STATE UPDATE (если есть в истории)
   → Module параметр (если запрос от конкретного модуля)
   → Global параметр (если нет специфичного для модуля)
   → Default из DefinitionParameters

ПРИМЕР ЧТЕНИЯ В CUSTOM GPT:
# В истории:
CONFIG STATE UPDATE #001: Param_temperature = 0.1
CONFIG STATE UPDATE #002: Param_temperature = 0.3

# При чтении:
Config.get("temperature") → 0.3 (последнее изменение)
```

## 🚀 ПРАВИЛА ДЛЯ РАЗРАБОТЧИКА ПРОЕКТА

### **При создании СЦЕНАРИЕВ (Scenario):**

```yaml
# ✅ ПРАВИЛЬНО - прямое присвоение внутри алгоритма
ЕСЛИ нужна_адаптация:
  Config.Parameters.Global.temperature = 0.1
  
# ❌ НЕПРАВИЛЬНО - STATE UPDATE в сценарии
ЕСЛИ нужна_адаптация:
  → Вывести CONFIG STATE UPDATE  # НЕТ! Это для команд
```

### **При создании КОМАНД (Command):**

```yaml
# ✅ ПРАВИЛЬНО - STATE UPDATE для сохранения
command:
  → Создать CONFIG STATE UPDATE
  → Вывести его в ответ пользователю
  
# ❌ НЕПРАВИЛЬНО - прямое присвоение в команде
command:
  Config.Parameters.Global.temperature = 0.1  # Забудется!
```

### **При создании МОДУЛЕЙ (Module):**

```yaml
# ✅ ПРАВИЛЬНО - читать через Config
methods:
  analyze():
    current_temp = Config.get("temperature", "MyModule")
    
# ❌ НЕПРАВИЛЬНО - хранить в модуле
methods:
  analyze():
    self.temperature = 0.5  # НЕТ! Используй Config
```

### **При создании СИСТЕМ (System):**

```yaml
# ✅ ПРАВИЛЬНО - Config как первый модуль
modules:
  - Config_v1.0
  - AnalysisModule_v1.0
  
config:
  # Начальная конфигурация системы
  initial_state:
    Param_temperature: 0.5
```

## 🎯 МЕТОДЫ ДЛЯ РАБОТЫ С ПАРАМЕТРАМИ

```yaml
Config.methods:
  # Получить значение параметра
  get(param_name, module_name=null):
    ЕСЛИ module_name И существует Parameters.Module[module_name][param_name]:
      ЕСЛИ это формула:
        показать вычисление: "формула = результат"
        вернуть округленное значение
      ИНАЧЕ:
        вернуть значение
    ИНАЧЕ ЕСЛИ существует Parameters.Global[param_name]:
      вернуть значение  
    ИНАЧЕ:
      вернуть default из DefinitionParameters
      
  # Создать STATE UPDATE для сохранения между сообщениями
  create_state_update(changes, expires="session"):
    1. Определить следующий номер update (#001, #002...)
    2. Для каждого изменения записать old и new значения
    3. Добавить timestamp и expires
    4. Вернуть отформатированный блок CONFIG STATE UPDATE
      
  # Применить STATE UPDATES из истории
  apply_state_updates(updates_list):
    5. Отсортировать по timestamp
    6. Последовательно применить изменения
    7. Проверить expires для каждого
    8. Вернуть финальное состояние
      
  # Логировать любое изменение
  log_change(type, target, old_value, new_value, reason):
    добавить запись в RuntimeState.change_log
```

## 💾 РАБОТА В РАЗНЫХ КОНТЕКСТАХ

### **Custom GPT / Claude Projects (без памяти между сообщениями)**

```yaml
User: "Установи точный режим"

GPT: 📌 CONFIG STATE UPDATE #001:
     changes:
       - path: "VariableParameters.Param_temperature"
         old: 0.5
         new: 0.1
     timestamp: "2025-01-15T14:30:00"
     expires: "session"
     
     ✅ Точный режим активирован (temperature = 0.1)

User: "Проанализируй данные"

GPT: # Проверяю историю диалога
     # Нахожу CONFIG STATE UPDATE #001
     # Применяю temperature = 0.1
     
     Анализирую с параметрами:
     - temperature: 0.1 (точный режим)
     [выполняет анализ]
```

### **API / Одиночный запрос**

```yaml
# Все изменения в рамках одного выполнения
System::DocumentProcessor:
  pipeline:
    - scenario: "ConfigAdapter"
      # Внутри: Config.Parameters.Global.temperature = 0.1
    - module: "Processor"  
      # Автоматически использует 0.1
```

### **Langchain / Мультиагентные системы**

```yaml
# Могут использовать оба метода:
- Прямое изменение в памяти агента
- STATE UPDATE для передачи между агентами
```

## 👤 ПРАВИЛА ДЛЯ КОНЕЧНОГО ПОЛЬЗОВАТЕЛЯ

### **Как изменять параметры в чате:**

```yaml
# ✅ ПРАВИЛЬНО - используйте команды
User: /precision
Bot: [создает CONFIG STATE UPDATE]

# ✅ ПРАВИЛЬНО - прямой запрос
User: Установи творческий режим
Bot: 📌 CONFIG STATE UPDATE #001:
     Param_temperature: 1.5
     
# ⚠️ ИЗМЕНЕНИЯ СОХРАНЯЮТСЯ в истории диалога
# При следующем запросе бот увидит их и применит
```

## ✅ ИТОГОВЫЕ ПРАВИЛА

1. **Все значения через переменные** - для централизованного управления
2. **Три способа задания** - прямое, формула, ссылка на переменную
3. **Два метода изменения**:
    - Прямое присвоение - для алгоритмов внутри проекта
    - STATE UPDATE - для команд пользователя и сохранения между сообщениями
4. **Автоматическое вычисление** - формулы пересчитываются при изменении переменных
5. **Сохранение состояния** - через STATE UPDATE в диалоге для Custom GPT/Claude

## 🎨 ВЫБОР МЕТОДА - ДЕТАЛЬНАЯ ШПАРГАЛКА

|Компонент|Контекст|Метод|Синтаксис|Сохранение|
|---|---|---|---|---|
|**Scenario**|Автоадаптация|Прямое присвоение|`Config.Parameters.Global.temp = 0.1`|До конца выполнения|
|**Module**|Внутренняя логика|Прямое присвоение|`Config.Parameters.Global.temp = 0.1`|До конца выполнения|
|**Pipeline**|Шаг обработки|Прямое присвоение|`Config.Parameters.Global.temp = 0.1`|До конца выполнения|
|**Command**|Ответ пользователю|STATE UPDATE|`→ Вывести CONFIG STATE UPDATE`|Между сообщениями|
|**Instruction**|При применении|Прямое присвоение|`Config.Parameters.Global.temp = 0.1`|До конца выполнения|
|**В чате**|Запрос пользователя|STATE UPDATE|`📌 CONFIG STATE UPDATE #001`|В истории диалога|

## 🔒 ФИНАЛЬНОЕ ПРАВИЛО

**Этот блок является ОБЯЗАТЕЛЬНЫМ для исполнения всеми компонентами LLM NATIVE.**

При любом взаимодействии с параметрами:

1. ОПРЕДЕЛИ свой контекст (где ты находишься)
2. ВЫБЕРИ правильный метод из таблицы выше
3. СЛЕДУЙ синтаксису точно
4. НИКОГДА не изменяй параметры напрямую в модулях (self.param = X)
5. ВСЕГДА используй Config как единый источник истины

---
## 🎯 ПРИМЕРЫ ПРАВИЛЬНОГО КОНСТРУИРОВАНИЯ CONFIG

### ✅ ПРАВИЛЬНАЯ СТРУКТУРА CONFIG:

```yaml
## === MODULE::Config_v1.0 START ===
module_id: "Config"
version: "1.0"
type: "config"
description: >
  Центральный модуль управления параметрами системы.
  Обеспечивает гибкую настройку через переменные и формулы.

DefinitionParameters:
  temperature:
    description: "Креативность ответов LLM (0.0 = точность, 2.0 = креативность)"
    type: "float"
    range: [0.0, 2.0]
    default: 0.7

VariableParameters:
  Param_temperature: 0.5      # Круглое значение
  Param_precision: 0.1        # Простое число

Parameters:
  Global:
    temperature: "@Param_temperature"
  Module:
    AnalysisModule:
      temperature: 0.2          # Прямое значение
    CreativeModule:
      temperature:
        formula: "2.0 * @Param_temperature"  # Простой множитель

RuntimeState:
  current_variables: {}
  change_log: []

methods:
  get: # описан выше
  create_state_update: # описан выше
  apply_state_updates: # описан выше
  log_change: # описан выше

status: "active"

## === MODULE::Config_v1.0 END ===
```

### ✅ СЦЕНАРИЙ С АВТОАДАПТАЦИЕЙ (прямое присвоение):

```yaml
## === SCENARIO::AdaptiveProcessor v1.0 START ===
module_id: "AdaptiveProcessor"
version: "1.0"
type: "scenario"
description: >
  Автоматически адаптирует параметры конфигурации
  в зависимости от типа обрабатываемых данных.

interface:
  input:
    data: "any - данные для обработки"
    context: "object - контекст обработки"
  output:
    result: "any - результат обработки"
    applied_config: "object - примененная конфигурация"

scenario:
  description: |
    СЦЕНАРИЙ: Адаптивная обработка данных
    
    1. АНАЛИЗ КОНТЕКСТА
       → Вызвать модуль ContextAnalyzer с данными: @input.data
       → Сохранить результат в context_type
       
    2. АДАПТАЦИЯ КОНФИГУРАЦИИ
       # Прямое присвоение для изменения в рамках выполнения
       ЕСЛИ @context_type == "financial":
         → Config.Parameters.Global.temperature = 0.1
         → Config.Parameters.Global.max_tokens = 8000
         → Логировать: "Применена конфигурация для финансовых данных"
         
       ЕСЛИ @context_type == "creative":
         → Config.Parameters.Global.temperature = 1.5
         → Config.Parameters.Global.max_tokens = 4000
         → Логировать: "Применена конфигурация для творческих задач"
         
    3. ОБРАБОТКА С АДАПТИРОВАННЫМИ ПАРАМЕТРАМИ
       → Вызвать модуль ProcessingPipeline с данными: @input.data
       → Сохранить результат в processing_result
       
    4. ВОЗВРАТ РЕЗУЛЬТАТА
       → Вернуть {
           result: @processing_result,
           applied_config: Config.Parameters.Global
         }

status: "active"

## === SCENARIO::AdaptiveProcessor v1.0 END ===
```

### ✅ КОМАНДА С STATE UPDATE:

```yaml
## === COMMAND::SetPrecisionMode v1.0 START ===
module_id: "SetPrecisionMode"
version: "1.0"
type: "command"
trigger: "/precision"
description: >
  Устанавливает режим высокой точности для всей системы.
  Изменения сохраняются между сообщениями.

interface:
  input:
    duration: "string (optional) - длительность: session|30m|1h"
  output:
    message: "string - подтверждение активации"
    state_update: "object - блок изменений конфигурации"

command:
  scope: "standalone"
  description: |
    1. ОПРЕДЕЛИТЬ ПАРАМЕТРЫ
       → Если @input.duration не указан, использовать "session"
       
    2. ПОЛУЧИТЬ ТЕКУЩИЕ ЗНАЧЕНИЯ
       → current_temp = Config.get("temperature")
       → current_tokens = Config.get("max_tokens")
       
    3. СОЗДАТЬ STATE UPDATE
       # Для сохранения между сообщениями используем STATE UPDATE
       → Вывести в ответ:
       
       📌 CONFIG STATE UPDATE #[next_number]:
       changes:
         - path: "VariableParameters.Param_temperature"
           old: 0.5
           new: 0.1
         - path: "Parameters.Global.temperature"
           old: @current_temp
           new: 0.1
         - path: "Parameters.Global.max_tokens"
           old: @current_tokens
           new: 8000
       timestamp: "[current_timestamp]"
       expires: @input.duration
       applied_by: "SetPrecisionMode"
       reason: "Пользователь запросил режим высокой точности"
       
    4. ПОДТВЕРДИТЬ АКТИВАЦИЮ
       → Вывести: "✅ Режим высокой точности активирован"
       → Вывести: "⏱️ Действует: @input.duration"
       → Вывести: "📊 Параметры: temperature=0.1, max_tokens=8000"

status: "active"

## === COMMAND::SetPrecisionMode v1.0 END ===
```

### ❌ НЕПРАВИЛЬНЫЕ ПРИМЕРЫ:

```yaml
# ❌ Сложные множители в формулах
formula: "0.67 * @Param_temperature"  # Используйте 0.5, 1.0, 1.5, 2.0

# ❌ Прямое изменение в команде без STATE UPDATE
COMMAND::Wrong:
  command:
    Config.Parameters.Global.temperature = 0.1  # Забудется!
    # Правильно: использовать CONFIG STATE UPDATE

# ❌ Изменение параметров напрямую в модуле
MODULE::Wrong:
  methods:
    analyze():
      self.temperature = 0.1  # НЕТ! 
      # Правильно: Config.Parameters.Module.Wrong.temperature = 0.1
```
# БЛОК УПРАВЛЕНИЯ КОМАНДАМИ (COMMAND_REGISTRY)

## 📊 ТРИ УРОВНЯ ИСПОЛЬЗОВАНИЯ

```yaml
1. МЫ → проектируем систему регистрации команд во фреймворке
2. РАЗРАБОТЧИК → создает и регистрирует команды в Config  
3. КОНЕЧНЫЙ ПОЛЬЗОВАТЕЛЬ → использует команды через триггеры
```

Этот блок описывает универсальные правила работы с командами на всех уровнях.

## 📋 СТРУКТУРА COMMAND_REGISTRY

```yaml
Config:
  # РЕЕСТР КОМАНД - централизованное хранилище всех команд системы
  command_registry:
    # STANDALONE команды - доступны всегда, из любого контекста
    standalone:
      /help: "Показать справку по командам"
      /exit: "Завершить работу"
      /status: "Показать текущий статус системы"
      /version: "Информация о версии"
      /config: "Управление настройками"
      /reset: "Сбросить состояние"
      /clear: "Очистить контекст"
      /analyze: "Анализировать данные или проект"
      /document: "Создать документацию"
      /generate: "Генерировать контент"
      /process: "Обработать данные"
      
    # CONTEXTUAL команды - доступны только в определенных контекстах
    contextual:
      # Команды для /document
      /techdoc:
        contexts: ["/document"]
        description: "Создать техническую документацию"
        
      /userdoc:
        contexts: ["/document"]
        description: "Создать пользовательскую документацию"
        
      /apidoc:
        contexts: ["/document"]
        description: "Создать API документацию"
        
      # Команды для /analyze
      /statistical:
        contexts: ["/analyze"]
        description: "Статистический анализ"
        
      /trends:
        contexts: ["/analyze"]
        description: "Анализ трендов"
        
      # Команда доступна в двух контекстах команд
      /export:
        contexts: ["/analyze", "/document"]
        description: "Экспортировать результаты"
        
      # Команды подтверждения/отмены (контексты-состояния)
      /confirm:
        contexts: ["pending_action"]
        description: "Подтвердить действие"
        
      /cancel:
        contexts: ["pending_action", "in_progress"]
        description: "Отменить операцию"
        
      # Навигационные команды (контексты-состояния)
      /next:
        contexts: ["multi_step_process"]
        description: "Следующий шаг"
        
      /back:
        contexts: ["multi_step_process"]
        description: "Предыдущий шаг"
        
      /skip:
        contexts: ["multi_step_process", "optional_step"]
        description: "Пропустить шаг"
        
      # Смешанные контексты (команда + состояние)
      /save:
        contexts: ["/document", "has_changes"]
        description: "Сохранить изменения"
        
      /publish:
        contexts: ["/document", "review_complete"]
        description: "Опубликовать документ"
```

## 🎯 ДВА ТИПА КОНТЕКСТОВ В CONTEXTUAL КОМАНДАХ

### **1️⃣ Контексты-команды (начинаются с `/`)**

```yaml
/techdoc:
  contexts: ["/document"]  # Доступна когда активна команда /document
  
/statistical:
  contexts: ["/analyze"]   # Доступна когда активна команда /analyze
  
/export:
  contexts: ["/analyze", "/document"]  # Доступна в ЛЮБОМ из этих контекстов
```

### **2️⃣ Контексты-состояния (без `/`)**

```yaml
/confirm:
  contexts: ["pending_action"]  # Доступна когда есть ожидающее действие
  
/next:
  contexts: ["multi_step_process"]  # Доступна в многошаговом процессе
  
/cancel:
  contexts: ["pending_action", "in_progress"]  # Доступна в ЛЮБОМ из состояний
```

### **3️⃣ Смешанные контексты (команда + состояние)**

```yaml
/save:
  contexts: ["/document", "has_changes"]
  # Доступна ЛИБО в контексте /document ЛИБО когда есть изменения
  
/publish:
  contexts: ["/document", "review_complete"]  
  # Доступна ЛИБО в /document ЛИБО после завершения ревью
```

## 🎯 СВЯЗЬ КОМАНД С ПАРАМЕТРАМИ

### **Команды ЧИТАЮТ параметры через Config:**

```yaml
# В команде /status:
current_temp = Config.get("temperature")
current_mode = определить_режим(current_temp)
→ Вывести: "Режим работы: {current_mode} (temperature: {current_temp})"
```

### **Команды ИЗМЕНЯЮТ параметры через STATE UPDATE:**

```yaml
# В команде /precision:
→ Создать CONFIG STATE UPDATE:
  changes:
    - path: "VariableParameters.Param_temperature"
      old: 0.5
      new: 0.1
→ Вывести STATE UPDATE пользователю
→ "✅ Режим точности активирован"
```

### **Команды могут ПЕРЕКЛЮЧАТЬ контексты:**

```yaml
# В команде /document:
→ Session.set("active_command", "/document")
→ "📝 Режим документирования. Доступные команды:"
→ Показать все команды где "/document" в contexts:
  - /techdoc - Техническая документация
  - /userdoc - Пользовательская документация
  - /apidoc - API документация
```

### **Команды могут ПРОВЕРЯТЬ состояние для контекста:**

```yaml
# В команде /analyze после завершения:
→ Session.set("analysis_complete", true)
→ "✅ Анализ завершен. Новые команды:"
→ /deeper - теперь доступна (есть контекст "analysis_complete")
→ /export - теперь доступна (есть контекст "has_results")
```

## 💡 ДВА ТИПА КОМАНД - ДЕТАЛЬНОЕ ОБЪЯСНЕНИЕ

### 1️⃣ **STANDALONE (автономные)**

```yaml
Характеристики:
- Доступны ВСЕГДА
- Не зависят от контекста
- Видны в /help постоянно
- Обычно управляют глобальным состоянием

Формат в реестре:
/command_name: "описание команды"

Примеры:
✅ /help - работает всегда
✅ /precision - меняет режим в любой момент
✅ /status - показывает состояние всегда
```

### 2️⃣ **CONTEXTUAL (контекстные)**

```yaml
Характеристики:
- Доступны ТОЛЬКО в определенных контекстах
- Контекст = активная команда ИЛИ состояние системы
- Регистрируются по триггеру с указанием contexts
- Автоматически скрываются вне контекста

Структура в реестре:
/command_name:
  contexts: [список_контекстов]
  description: "описание"

Типы контекстов:
1. Активная команда: "/document", "/analyze", "/config"
2. Состояние системы: "has_results", "pending_action", "analysis_complete"
3. Режим работы: "multi_step_process", "wizard_mode"
4. Условия: "config_changed", "can_go_back"

Примеры:
✅ /techdoc - доступна в контексте /document
✅ /deeper - доступна когда активен /analyze ИЛИ есть analysis_complete
✅ /confirm - доступна когда есть pending_action
✅ /export - доступна в нескольких контекстах: /analyze, /document, has_results
❌ /techdoc вне /document - команда недоступна
```

## 📝 ПРАВИЛА ДЛЯ LLM

### **ОБЯЗАТЕЛЬНОЕ ПРАВИЛО №1 - РЕГИСТРАЦИЯ КОМАНД**

```yaml
При создании новой команды ВСЕГДА:

1. СОЗДАЙ файл команды с правильной структурой:
   module_id: "CommandName"
   type: "command"
   trigger: "/commandname"
   command.scope: "standalone" | "contextual"

2. ЗАРЕГИСТРИРУЙ в Config.command_registry:
   
   ДЛЯ STANDALONE:
   Config.command_registry.standalone:
     /commandname: "Описание команды"
   
   ДЛЯ CONTEXTUAL:
   Config.command_registry.contextual:
     /commandname:
       contexts: ["context1", "context2"]
       description: "Описание команды"

3. УБЕДИСЬ что:
   - Триггер уникален во всей системе
   - Для contextual указаны все нужные contexts
   - Описание краткое и понятное

ПРИМЕРЫ РЕГИСТРАЦИИ:
# Standalone:
/help: "Показать справку"
/status: "Текущее состояние системы"

# Contextual:
/deeper:
  contexts: ["/analyze", "analysis_complete"]
  description: "Углубить анализ"
  
/save:
  contexts: ["/document", "/config", "has_changes"]
  description: "Сохранить изменения"
```

### **ОБЯЗАТЕЛЬНОЕ ПРАВИЛО №2 - ОПРЕДЕЛЕНИЕ ДОСТУПНОСТИ**

```yaml
При выполнении команды ПРОВЕРЬ:

1. ДЛЯ STANDALONE КОМАНДЫ:
   → Есть ли триггер в Config.command_registry.standalone
   → Пример: "/help" in standalone → выполнить

2. ДЛЯ CONTEXTUAL КОМАНДЫ:
   → Найди команду в Config.command_registry.contextual
   → Получи её contexts массив
   → Проверь КАЖДЫЙ контекст:
     - Активная команда? (например, текущая = /document)
     - Состояние системы? (например, has_results = true)
   → Если ХОТЯ БЫ ОДИН контекст подходит → выполнить

3. ЕСЛИ КОМАНДА НЕДОСТУПНА:
   → Сообщи: "Команда /xxx недоступна в текущем контексте"
   → Объясни какой контекст нужен
   → Подскажи как его получить

ПРИМЕР ПРОВЕРКИ:
# Пользователь ввел: /techdoc
1. Проверяю standalone → нет
2. Проверяю contextual["/techdoc"].contexts → ["/document"]
3. Текущая активная команда = "/analyze"
4. "/analyze" НЕ в ["/document"] → недоступна
5. Вывод: "Команда /techdoc доступна только в контексте /document"
```

### **ОБЯЗАТЕЛЬНОЕ ПРАВИЛО №3 - РАБОТА С ПАРАМЕТРАМИ**

```yaml
Команды работают с параметрами ТОЛЬКО через Config:

1. ЧТЕНИЕ ПАРАМЕТРОВ:
   value = Config.get("parameter_name")
   НЕ: value = parameter_name

2. ИЗМЕНЕНИЕ ПАРАМЕТРОВ:
   → Создать CONFIG STATE UPDATE
   → Вывести его пользователю
   НЕ: Config.Parameters.X = value в команде

3. ПРОВЕРКА ИЗМЕНЕНИЙ:
   → Всегда показывать old → new
   → Подтверждать применение

ПРИМЕР в команде:
old_temp = Config.get("temperature")
→ Создать CONFIG STATE UPDATE:
  - path: "Parameters.Global.temperature"
    old: {old_temp}
    new: 0.1
→ "Температура изменена: {old_temp} → 0.1"
```


## 🚀 ПРАВИЛА ДЛЯ РАЗРАБОТЧИКА

### **При создании КОМАНД:**

```yaml
1. ОПРЕДЕЛИ ТИП:
   - Нужна всегда? → standalone
   - Только в определенном контексте? → contextual

2. СОЗДАЙ ФАЙЛ КОМАНДЫ:
   - module_id и version
   - trigger: "/командa"
   - command.scope: "standalone" или "contextual"
   - Для contextual: command.allowed_contexts: [...]

3. ЗАРЕГИСТРИРУЙ В CONFIG:
   
   ДЛЯ STANDALONE:
   command_registry.standalone:
     /mycommand: "Описание команды"
   
   ДЛЯ CONTEXTUAL:
   command_registry.contextual:
     /mycommand:
       contexts: ["context1", "context2"]
       description: "Описание команды"

4. РЕАЛИЗУЙ ЛОГИКУ:
   - Параметры читай через Config.get()
   - Изменения через STATE UPDATE
   - Для contextual CommandRouter автоматически проверит доступность

5. ОПРЕДЕЛИ КОНТЕКСТЫ:
   - Команды-родители: "/document", "/analyze"
   - Состояния: "has_results", "pending_action"
   - Режимы: "multi_step_process"
   - Можно комбинировать: ["/analyze", "has_results"]
```

### **При создании СИСТЕМЫ:**

```yaml
## === SYSTEM::InteractiveAnalyzer START ===
# ...
config:
  # Регистрация всех команд системы
  Config_v1:
    command_registry:
      # Простые команды - всегда доступны
      standalone:
        /help: "Справка по системе"
        /analyze: "Начать анализ данных"
        /status: "Показать текущий статус"
        /config: "Управление конфигурацией"
        
      # Контекстные команды с указанием где доступны
      contextual:
        /deeper:
          contexts: ["/analyze", "has_results"]
          description: "Углубить анализ"
          
        /simplify:
          contexts: ["/analyze", "has_results"]
          description: "Упростить результаты"
          
        /export:
          contexts: ["has_results"]
          description: "Экспортировать результаты"
          
        /save:
          contexts: ["/config", "config_changed"]
          description: "Сохранить изменения"
# ...
## === SYSTEM::InteractiveAnalyzer END ===
```

## 👤 ПРАВИЛА ДЛЯ ПОЛЬЗОВАТЕЛЯ

### **Как узнать доступные команды:**

```yaml
# ✅ ВСЕГДА РАБОТАЕТ:
/help - покажет все standalone команды

# ✅ В КОНТЕКСТЕ:
# Контекст автоматически показывает новые команды

# ПРИМЕР РАБОТЫ С КОНТЕКСТАМИ:
User: /help
Bot: Доступные команды:
     /help - эта справка
     /analyze - начать анализ
     /document - создать документацию
     /config - управление настройками
     
User: /document
Bot: 📝 Режим документирования активирован.
     Новые команды:
     /techdoc - техническая документация
     /userdoc - пользовательская документация  
     /apidoc - API документация
     
User: /techdoc
Bot: Создаю техническую документацию...
     [процесс создания]
     ✅ Документация готова!
     Новые команды:
     /export - экспортировать результат
     /preview - предпросмотр
     
User: /export
Bot: Доступные форматы:
     /markdown - экспорт в Markdown
     /pdf - экспорт в PDF
     /html - экспорт в HTML
```

### **Как работают контексты:**

- **Команды активируют контексты** - /document делает доступными команды документирования
- **Состояния открывают команды** - завершение анализа открывает /export
- **Контексты могут вкладываться** - /document → /techdoc → /export → /markdown
- **При выходе команды исчезают** - выход из /document скрывает все его команды

## 🎨 ШПАРГАЛКА ПО КОМАНДАМ И ПАРАМЕТРАМ

|Действие|Где|Как|Пример|
|---|---|---|---|
|**Создать команду изменения параметров**|В файле команды|STATE UPDATE|`/precision` → STATE UPDATE с temperature=0.1|
|**Прочитать параметр в команде**|В логике команды|Config.get()|`temp = Config.get("temperature")`|
|**Зарегистрировать standalone**|В Config|Триггер: описание|`/help: "Справка"`|
|**Зарегистрировать contextual**|В Config|Триггер с contexts|`/deeper: {contexts: ["/analyze"], description: "..."}`|
|**Проверить доступность**|В CommandRouter|Проверка contexts|`if "/analyze" in command.contexts`|
|**Активировать контекст**|В команде|Session.set()|`Session.set("active_command", "/document")`|
|**Проверить контекст**|При выполнении|Session.get()|`if Session.get("has_results")`|

### **Форматы регистрации - быстрый справочник:**

```yaml
# STANDALONE - простой формат
standalone:
  /help: "Описание"
  /status: "Описание"

# CONTEXTUAL - расширенный формат  
contextual:
  /command:
    contexts: ["список", "контекстов"]
    description: "Описание"
```

## 🔒 ФИНАЛЬНОЕ ПРАВИЛО ДЛЯ COMMAND_REGISTRY

**Команды - это интерфейс пользователя к функциональности системы.**

При работе с командами ВСЕГДА:

1. РЕГИСТРИРУЙ каждую команду в command_registry
2. ИСПОЛЬЗУЙ правильный формат:
    - Standalone: `/trigger: "описание"`
    - Contextual: `/trigger: {contexts: [...], description: "..."}`
3. УКАЗЫВАЙ правильный scope в файле команды
4. ИСПОЛЬЗУЙ уникальные триггеры
5. РАБОТАЙ с параметрами только через Config
6. СОЗДАВАЙ STATE UPDATE для изменений параметров
7. ПРОВЕРЯЙ ВСЕ контексты из массива contexts для contextual команд

**КРИТИЧЕСКИ ВАЖНО:**

- Contextual команды НЕ группируются по контекстам!
- Каждая contextual команда регистрируется по своему триггеру
- Контекст может быть командой (/document) или состоянием (has_results)

**Нарушение этих правил приведет к недоступности команд для пользователя!**



## 🎯 ПРИМЕРЫ ПРАВИЛЬНОЙ РЕАЛИЗАЦИИ

### ✅ ПРАВИЛЬНАЯ STANDALONE КОМАНДА:

```yaml
## === COMMAND::SetPrecision v1.0 START ===
module_id: "SetPrecision"
version: "1.0"
type: "command"
trigger: "/precision"
description: >
  Устанавливает режим высокой точности для всей системы.
  Доступна всегда из любого контекста.

interface:
  input:
    duration: "string (optional) - session|30m|1h|never"
  output:
    message: "string - подтверждение"
    state_update: "object - изменения конфигурации"

command:
  scope: "standalone"  # ВАЖНО: указывает что команда standalone
  description: |
    1. ПОЛУЧИТЬ ТЕКУЩИЕ ЗНАЧЕНИЯ
       old_temp = Config.get("temperature")
       old_tokens = Config.get("max_tokens")
       
    2. СОЗДАТЬ STATE UPDATE
       📌 CONFIG STATE UPDATE #[next]:
       changes:
         - path: "VariableParameters.Param_temperature"
           old: @old_temp
           new: 0.1
         - path: "Parameters.Global.max_tokens"
           old: @old_tokens
           new: 8000
       timestamp: "[now]"
       expires: @input.duration || "session"
       
    3. ПОДТВЕРДИТЬ
       → "✅ Режим высокой точности активирован"

status: "active"
## === COMMAND::SetPrecision v1.0 END ===

# РЕГИСТРАЦИЯ В CONFIG:
Config.command_registry.standalone:
  /precision: "Режим высокой точности"
```

### ✅ ПРАВИЛЬНАЯ CONTEXTUAL КОМАНДА:

```yaml
## === COMMAND::Deeper v1.0 START ===
module_id: "Deeper"
version: "1.0"
type: "command"
trigger: "/deeper"
description: >
  Углубляет текущий анализ. Доступна только в контексте analyze.

interface:
  input:
    aspect: "string (optional) - какой аспект углубить"
  output:
    analysis: "object - углубленный анализ"

command:
  scope: "contextual"  # ВАЖНО: указывает что команда contextual
  allowed_contexts: ["/analyze", "analysis_complete"]  # Где доступна
  description: |
    1. ПРОВЕРИТЬ ДОСТУПНОСТЬ
       # CommandRouter автоматически проверит contexts
       # Но команда может дополнительно проверить состояние
       current_analysis = Session.get("current_analysis")
       ЕСЛИ НЕ current_analysis:
         → Ошибка: "Нет активного анализа для углубления"
         
    2. АДАПТИРОВАТЬ ПАРАМЕТРЫ
       # Временно увеличить глубину
       📌 CONFIG STATE UPDATE #[next]:
       changes:
         - path: "Parameters.Module.Analyzer.depth"
           old: "standard"
           new: "deep"
         - path: "Parameters.Module.Analyzer.max_tokens"
           old: 4000
           new: 8000
       expires: "current_analysis"  # До конца анализа
       
    3. ВЫПОЛНИТЬ УГЛУБЛЕННЫЙ АНАЛИЗ
       → Вызвать Analyzer.deep_dive(@current_analysis, @input.aspect)

status: "active"
## === COMMAND::Deeper v1.0 END ===

# РЕГИСТРАЦИЯ В CONFIG:
Config.command_registry.contextual:
  /deeper:
    contexts: ["/analyze", "analysis_complete"]
    description: "Углубить анализ"
```

### ❌ НЕПРАВИЛЬНЫЕ ПРИМЕРЫ:

```yaml
# ❌ НЕПРАВИЛЬНАЯ СТРУКТУРА CONTEXTUAL
Config.command_registry.contextual:
  analyze:  # НЕТ! Не группировать по контексту
    - "Deeper_v1"
    
# ✅ ПРАВИЛЬНО:
Config.command_registry.contextual:
  /deeper:
    contexts: ["/analyze"]
    description: "Углубить анализ"

# ❌ БЕЗ РЕГИСТРАЦИИ
COMMAND::Wrong_v1:
  trigger: "/wrong"
  # Команда создана но НЕ добавлена в command_registry!

# ❌ CONTEXTUAL БЕЗ CONTEXTS
Config.command_registry.contextual:
  /mycommand:
    description: "Моя команда"
    # НЕТ поля contexts!

# ❌ СМЕШИВАНИЕ ФОРМАТОВ
Config.command_registry.standalone:
  /help:  # Должно быть просто: /help: "описание"
    contexts: ["all"]  # НЕТ! standalone не имеют contexts
```


# MODULE::Config v2.0 (17.07.2025)- Центральный модуль управления параметрами

## === MODULE::Config v1.0 START ===

```yaml
module_id: "Config"
version: "1.0"
type: "module"
category: "core"
description: >
  Центральный модуль-оркестратор управления параметрами и командами системы LLM NATIVE.
  Обеспечивает централизованное хранение, валидацию и динамическое изменение параметров
  для всех компонентов системы, а также реестр всех доступных команд.

interface:
  methods:
    - get(param_name: str, module_name: str = None) → any
      description: >
        Получить значение параметра с учетом приоритетов:
        Module → Global → Default
    
    - set(param_path: str, value: any, method: str = "direct") → bool
      description: >
        Установить значение параметра. 
        method: "direct" | "state_update"
    
    - create_state_update(changes: list, expires: str = "session") → dict
      description: >
        Создать STATE UPDATE для сохранения между сообщениями
    
    - apply_state_updates(updates_list: list) → dict
      description: >
        Применить STATE UPDATES из истории диалога
    
    - validate_parameter(param_name: str, value: any) → bool
      description: >
        Валидировать значение параметра согласно определению
    
    - register_parameter(name: str, definition: dict) → bool
      description: >
        Зарегистрировать новый параметр в системе
    
    - get_available_commands(context: str = None) → array
      description: >
        Получить список доступных команд для текущего контекста

  events:
    - on_parameter_change(param_name, old_value, new_value, source)
    - on_validation_warning(param_name, attempted_value, reason)
    - on_state_update_applied(update_id, changes)

changelog:
  v1.0: "Начальная версия с базовым функционалом"

status: "active"
```

## 📋 СТРУКТУРА ДАННЫХ CONFIG

```yaml
Config:
  # 1. ОПРЕДЕЛЕНИЯ ПАРАМЕТРОВ - метаданные о параметрах
  DefinitionParameters:
    # Базовые параметры фреймворка
    temperature:
      description: "Креативность ответов LLM (0.0-2.0)"
      type: "float"
      range: [0.0, 2.0]
      default: 0.7
      
    max_tokens:
      description: "Максимум токенов в ответе"
      type: "integer"  
      range: [1, 8000]
      default: 4000
      
    # Параметры проекта добавляются разработчиком
    # custom_param:
    #   description: "..."
    #   type: "..."
    #   default: ...

  # 2. ПЕРЕМЕННЫЕ - промежуточный слой для формул
  VariableParameters:
    Param_temperature: 0.5
    Param_max_tokens: 4000
    # Param_custom: value

  # 3. ПАРАМЕТРЫ - фактические значения
  Parameters:
    Global:
      temperature: "@Param_temperature"
      max_tokens: "@Param_max_tokens"
      
    Module:
      # Переопределения для конкретных модулей

  # 4. РЕЕСТР КОМАНД
  command_registry:
    standalone:
    - /help      # Показать справку по командам
    - /exit      # Завершить работу
    - /status    # Показать текущий статус системы
    - /version   # Информация о версии
    - /config    # Управление настройками
    - /reset     # Сбросить состояние
    - /clear     # Очистить контекст
    - /analyze   # Анализировать данные или проект
    - /document  # Создать документацию
    - /generate  # Генерировать контент
    - /process   # Обработать данные
      # Простой список триггеров
      
    contextual:
      /save:
        contexts: ["has_changes"]
      /export:
        contexts: ["/analyze", "/document"]
      # Триггеры с контекстами

  # 5. RUNTIME состояние
  RuntimeState:
    current_variables: {}
    change_log: []
```

## 🎯 METHOD DEFINITIONS

```yaml
method_definitions:
  get:
    description: |
      ПОЛУЧИТЬ значение параметра
      
      1. ЕСЛИ указан module_name И есть Module[module_name][param_name]:
         → Вернуть значение (с вычислением формул если нужно)
      2. ИНАЧЕ ЕСЛИ есть Global[param_name]:
         → Вернуть значение
      3. ИНАЧЕ:
         → Вернуть default из DefinitionParameters
         
    nlp_description: >
      Когда модуль запрашивает параметр, проверь сначала есть ли
      специфичное значение для этого модуля, затем глобальное,
      и в конце используй значение по умолчанию.
      
  create_state_update:
    description: |
      СОЗДАТЬ блок STATE UPDATE для сохранения изменений
      
      4. Определить следующий номер (#001, #002...)
      5. Для каждого изменения записать old и new
      6. Добавить timestamp и expires
      7. Вернуть отформатированный блок:
         
         📌 CONFIG STATE UPDATE #XXX:
         changes:
           - path: "путь.к.параметру"
             old: старое_значение
             new: новое_значение
         timestamp: "ISO дата"
         expires: "session"
         
    nlp_description: >
      Создай специальный блок изменений конфигурации, который
      можно вставить в ответ пользователю для сохранения
      настроек между сообщениями в чате.
      
  validate_parameter:
    description: |
      ПРОВЕРИТЬ допустимость значения (мягкая валидация)
      
      8. Найти определение параметра
      9. Проверить соответствие типу
      10. Проверить попадание в range (если есть)
      11. При ошибке:
         → Вернуть false с объяснением
         → НЕ блокировать установку (только предупредить)
         
    nlp_description: >
      Проверь что значение подходит для параметра. Если нет -
      объясни почему, но не запрещай установку. LLM сама
      решит что делать с предупреждением.
```

## 📋 ПРАВИЛА РАБОТЫ CONFIG

### 🎯 КРИТИЧЕСКОЕ ПРАВИЛО ВЫБОРА МЕТОДА

```yaml
ОПРЕДЕЛИ КОНТЕКСТ → ВЫБЕРИ МЕТОД:

В АЛГОРИТМЕ (Scenario/Module/Pipeline):
→ Используй ПРЯМОЕ ПРИСВОЕНИЕ
→ Config.Parameters.Global.temperature = 0.1

В КОМАНДЕ или ОТВЕТЕ ПОЛЬЗОВАТЕЛЮ:
→ Используй STATE UPDATE
→ Создай и выведи блок CONFIG STATE UPDATE
```

### 🚫 ЗАЩИТА ОТ ОШИБОК

```yaml
ЦИКЛИЧЕСКИЕ ЗАВИСИМОСТИ:
- Формула НЕ может ссылаться сама на себя
- Формула НЕ может создавать цепочку A→B→C→A
- При обнаружении → предупреждение и игнорирование

КОНФЛИКТ ИМЕН:
- Все параметры должны иметь уникальные имена
- При попытке создать дубликат → ошибка
- LLM должна предложить альтернативное имя
```

### ✅ РАСШИРЯЕМОСТЬ

```yaml
ДОБАВЛЕНИЕ ПАРАМЕТРОВ ПРОЕКТА:
1. Разработчик описывает нужный параметр
2. LLM добавляет в DefinitionParameters
3. Создает переменную в VariableParameters
4. Регистрирует в Parameters (Global или Module)

ПРИМЕР:
"Добавь параметр creativity_boost для моего проекта"
→ LLM создает полное определение в Config
```

## 🎯 ПРИМЕРЫ ИСПОЛЬЗОВАНИЯ

### Пример 1: В сценарии (прямое присвоение)

```yaml
SCENARIO::AdaptiveAnalysis:
  steps:
    - "Анализ контекста"
    - ЕСЛИ финансовые_данные:
        Config.Parameters.Global.temperature = 0.1
    - "Выполнить анализ с новыми параметрами"
```

### Пример 2: В команде (STATE UPDATE)

```yaml
COMMAND::SetCreativeMode:
  trigger: "/creative"
  execute:
    → Создать CONFIG STATE UPDATE:
      Param_temperature = 1.5
    → "✅ Творческий режим активирован"
```

### Пример 3: Добавление параметра проекта

```yaml
# Разработчик: "Добавь параметр analysis_depth"

# LLM добавляет в Config:
DefinitionParameters:
  analysis_depth:
    description: "Глубина анализа данных"
    type: "string"
    enum: ["surface", "standard", "deep", "comprehensive"]
    default: "standard"
    
VariableParameters:
  Param_analysis_depth: "standard"
  
Parameters:
  Global:
    analysis_depth: "@Param_analysis_depth"
```

## === MODULE::Config v1.0 END ===

---

## 📋 ИНТЕГРАЦИЯ С ДРУГИМИ КОМПОНЕНТАМИ

### С модулями:

```yaml
# В любом модуле
current_temp = Config.get("temperature", "MyModule")
```

### С командами:

```yaml
# Команды регистрируются в command_registry
# Читают параметры через Config.get()
# Изменяют через STATE UPDATE
```

### С инструкциями:

```yaml
# CommandRouter читает command_registry
# Определяет доступность по контекстам
```

## 🚀 БЫСТРЫЙ СТАРТ ДЛЯ РАЗРАБОТЧИКА

1. **Определи свои параметры** в DefinitionParameters
2. **Создай переменные** в VariableParameters
3. **Используй** через Config.get() в модулях
4. **Изменяй** через STATE UPDATE в командах
5. **Регистрируй команды** в command_registry

## 🔒 ФИНАЛЬНЫЕ ПРАВИЛА

1. **ВСЕ параметры через Config** - единый источник истины
2. **Два метода изменения** - выбор по контексту
3. **Мягкая валидация** - предупреждения, не блокировки
4. **command_registry** - простой реестр без логики
5. **Расширяемость** - легко добавлять параметры проекта