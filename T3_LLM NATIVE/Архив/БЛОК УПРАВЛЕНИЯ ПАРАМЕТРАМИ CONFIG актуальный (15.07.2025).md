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