---
description: ДОБАВИТЬ КАРТИНКУ ПОСЛЕ РЕАЛИЗАЦИ
---

# Примеры диаграмм

## Пример 1

```c
VARIABLE x_value {
    LABEL "...";
    HELP "...";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT [degC];
}

ARRAY x_data {
    LABEL "x-data";
    NUMBER_OF_ELEMENTS 1000;
    TYPE x_value;
}

VARIABLE y_value {
    LABEL "...";
    HELP "...";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT [degC];
}

ARRAY y_data {
    LABEL "y-data";
    NUMBER_OF_ELEMENTS 1000;
    TYPE y_value;
}

VARIABLE x_min_value {
    LABEL "...";
    HELP "...";
    CLASS LOCAL;
    TYPE FLOAT;
}

AXIS x_axis_signature {
    LABEL "x axis";
    MIN_VALUE DEFAULT_X_MIN_VALUE;
    MAX_VALUE DEFAULT_X_MAX_VALUE;
}

AXIS y_axis_signature {
    LABEL "y axis";
    MIN_VALUE DEFAULT_Y_MIN_VALUE;
    MAX_VALUE DEFAULT_Y_MAX_VALUE;
}

WAVEFORM value_signature1 {
    TYPE XY {
        X_VALUES { x_data }
        Y_VALUES { y_data }
    }
    INIT_ACTIONS { read_first_signature }
    REFRESH_ACTIONS { read_signature }
    Y_AXIS y_axis_signature;
}

x_max_value LIKE x_min_value;
device_x_min_value LIKE x_min_value;
device_x_max_value LIKE x_min_value;
y_min_value LIKE x_min_value;
y_max_value LIKE x_min_value;
device_y_min_value LIKE x_min_value;
device_y_max_value LIKE x_min_value;

METHOD read_first_signature {
    DEFINITION {
        // Инициализация значений по умолчанию
        x_min_value = DEFAULT_X_MIN_VALUE;
        x_max_value = DEFAULT_X_MAX_VALUE;
        y_min_value = DEFAULT_Y_MIN_VALUE;
        y_max_value = DEFAULT_Y_MAX_VALUE;
        
        // Взаимодействие с устройством
        WriteCommand(write_data_area); // не для HART
        ReadCommand(read_data); // не для HART
        
        // Обновление осей на основе аппаратных значений
        x_axis_signature.MIN_VALUE = device_x_min_value;
        x_axis_signature.MAX_VALUE = device_x_max_value;
        y_axis_signature.MIN_VALUE = device_y_min_value;
        y_axis_signature.MAX_VALUE = device_y_max_value;
    }
}
METHOD read_signature {
    DEFINITION {
        // Чтение данных устройства в текущем диапазоне осей
        x_min_value = x_axis_signature.VIEW_MIN;
        x_max_value = x_axis_signature.VIEW_MAX;
        y_min_value = y_axis_signature.VIEW_MIN;
        y_max_value = y_axis_signature.VIEW_MAX;
        
        WriteCommand(write_data_area); // не для HART
        ReadCommand(read_data); // не для HART
        
        // Автомасштабирование при выходе за границы
        if (device_x_min_value < x_min_value)
            x_axis_signature.VIEW_MIN = device_x_min_value;
        if (device_x_max_value > x_max_value)
            x_axis_signature.VIEW_MAX = device_x_max_value;
        if (device_y_min_value < y_min_value)
            y_axis_signature.VIEW_MIN = device_y_min_value;
        if (device_y_max_value > y_max_value)
            y_axis_signature.VIEW_MAX = device_y_max_value;
    }
}

GRAPH valve_signature {
    MEMBERS {
        VAL_SIG, value_signature1;
    }
    X_AXIS x_axis_signature;
}
```

<figure><img src="https://i.postimg.cc/cH2ttw8V/Screenshot-5.png" alt=""><figcaption></figcaption></figure>

## Пример 2

Ниже продемонстрирован пример в EDD с общей осью для WAVEFORM w1 и w2 и отдельной осью для WAVEFORM w3

```c
AXIS x1 { }
AXIS y1 { }
AXIS y2 { }

GRAPH graph1 {
    MEMBERS {
        W1, w1;
        W2, w2;
        W3, w3;
    }
    X_AXIS x1;
}

WAVEFORM w1 {
    Y_AXIS y1;
    TYPE YT {
        Y_VALUES {w1_values}
        X_INCREMENT 1;
        X_INITIAL 0;
    }
}

WAVEFORM w2 {
    Y_AXIS y1;
    TYPE YT {
        Y_VALUES {w2_values}
        X_INCREMENT 1;
        X_INITIAL 0;
    }
}

WAVEFORM w3 {
    Y_AXIS y2;
    TYPE YT {
        Y_VALUES {w3_values}
        X_INCREMENT 1;
        X_INITIAL 0;
    }
}
```

<figure><img src="https://i.postimg.cc/x19dptG0/Screenshot-6.png" alt=""><figcaption></figcaption></figure>

## Пример 3

Приложение EDD должно поддерживать масштабирование и прокрутку без дополнительной поддержки в EDD. Приложение EDD показывает меньше или больше точек или данные из не отображаемой в настоящий момент части WAVEFORM в области отображения.

Кроме того, REFRESH\_ACTIONS в EDD могут поддерживать прокрутку и масштабирование. Приложение EDD вызывает REFRESH\_ACTIONS, если пользователь прокручивает в область, которая не сохранена в данных waveform. Оно также вызывает REFRESH\_ACTIONS, если пользователь масштабирует и должно быть показано больше точек. Информация о позиции и масштабировании может быть получена с оси с помощью атрибутов VIEW\_MIN и VIEW\_MAX.

Код ниже показывает пример EDD с поддержкой масштабирования и прокрутки устройством.

```c
VARIABLE y_value {
    LABEL "...";
    HELP "...";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT [degC];
}

ARRAY y_data {
    NUMBER OF ELEMENTS 1000;
    TYPE y_value;
}

COMMAND read_data {
    SLOT ...; INDEX ...;
    OPERATION READ;
    TRANSACTION {
        REPLY {
            device_t_min_value, device_t_max_value,
            device_y_min_value, device_y_max_value,
            y_data
        }
    }
}

COMMAND read_current_data {
    SLOT ...; INDEX ...;
    OPERATION READ;
    TRANSACTION {
    REPLY {
        device_t_min_value, device_t_max_value,
        device_y_min_value, device_y_max_value,
        y_data
    }
}
COMMAND write_data_area {
    SLOT ...; INDEX ...;
    OPERATION WRITE;
    TRANSACTION {
        REQUEST {
            t_min_value, t_max_value,
            y_min_value, y_max_value
        }
    }
}

VARIABLE y_increment {
    LABEL "..."; 
    HELP "...";
    CLASS LOCAL;
    TYPE TIME;
}

VARIABLE current_date_time1 {
    LABEL "..."; 
    HELP "...";
    CLASS LOCAL;
    TYPE DATE_AND_TIME;
}

VARIABLE t_min_value {
    LABEL "..."; 
    HELP "...";
    CLASS LOCAL;
    TYPE DATE_AND_TIME;
}

t_max_value LIKE t_min_value;
t_device_min_value LIKE t_min_value;
t_device_max_value LIKE t_min_value;

VARIABLE y_min_value {
    LABEL "..."; 
    HELP "...";
    CLASS LOCAL;
    TYPE FLOAT;
}

y_max_value LIKE y_min_value; // ?????
device_y_min_value LIKE y_min_value;
device_y_max_value LIKE y_min_value;

AXIS y_axis_signature {
    LABEL "...";
    HELP "...";
    MIN_VALUE y_min_value;
    MAX_VALUE y_max_value;
    SCALING LINEAR;
}

AXIS t_axis_signature {
    LABEL "...";
    HELP "...";
    MIN_VALUE t_min_value;
    MAX_VALUE t_max_value;
    SCALING LINEAR;
}

WAVEFORM value_signature1 {
    LABEL "...";
    HELP "...";
    HANDLING READ;  // альтернатива: READ & WRITE
    LINE_TYPE DATA1;
    EMPHASIS TRUE;
    TYPE XY {
        X_INITIAL current_date_time1;  // начальное время в мс
        X_INCREMENT y_increment;       // интервал между точками в мс
        Y_VALUES { y_data }            // массив Y-значений
    }
    Y_AXIS y_axis_signature;
    INIT_ACTIONS { init_signature }
    REFRESH_ACTIONS { refresh_signature }
}

METHOD init_signature {
    DEFINITION {
        // Чтение текущих данных и области хранения
        // t_device_min_value, t_device_max_value,
        // y_device_min_value, y_device_max_value = 
        read_command(read_current_data); // не для HART
        
        // Настройка параметров waveform
        value_signature1.X_INITIAL = t_device_max_value;
        value_signature1.X_INCREMENT = (t_device_max_value - t_device_min_value) / (y_data.NUMBER_OF_ELEMENTS - 1);
        
        // Установка видимой области согласно данным устройства
        t_axis_signature.MIN_VALUE = t_device_min_value;
        t_axis_signature.MAX_VALUE = t_device_max_value;
        y_axis_signature.MIN_VALUE = y_device_min_value;
        y_axis_signature.MAX_VALUE = y_device_max_value;
    }
}

METHOD refresh_signature {
    DEFINITION {
        // 1. Получение текущих границ осей
        t_min_value = t_axis_signature.MIN_VALUE;
        t_max_value = t_axis_signature.MAX_VALUE;
        y_min_value = y_axis_signature.MIN_VALUE;
        y_max_value = y_axis_signature.MAX_VALUE;
        
        // 2. Отправка запрошенного диапазона на устройство
        write_command(write_data_area); // не для HART
        
        // 3. Чтение данных и аппаратных границ
        read_command(read_data);  // не для HART // Получает:
                                 // t_device_min/max_value
                                 // y_device_min/max_value
                                 // y_data
        
        // 4. Корректировка видимой области
        if (t_device_min_value > t_min_value)
            t_axis_signature.MIN_VALUE = t_device_min_value;
        if (t_device_max_value < t_max_value)
            t_axis_signature.MAX_VALUE = t_device_max_value;
        if (y_device_min_value > y_min_value)
            y_axis_signature.MIN_VALUE = y_device_min_value;
        if (y_device_max_value < y_max_value)
            y_axis_signature.MAX_VALUE = y_device_max_value;
    }
}
```
