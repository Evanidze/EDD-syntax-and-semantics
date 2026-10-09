# Примеры диаграмм

## Пример 1

Ниже показана диаграмма с одной кривой, обновляющаяся каждую секунду

```c
MENU measuring_values {
    LABEL "Measuring Values";
    STYLE DIALOG;
    ITEMS {
        primary_value_view
    }
}

MENU primary_value_view {
    LABEL "Primary Measuring Value";
    STYLE PAGE;
    ITEMS {
        primary_value_chart
    }
}

VARIABLE primary_value {
    LABEL "Primary Value";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT UNIT "Bar";
}

SOURCE primary_value_source {
    MEMBERS {
        PRIM_VAL, primary_value;
    }
}

CHART primary_value_chart {
    LABEL "Primary Value";
    MEMBERS {
        CHART1, primary_value_source;
    }
}
```

## Пример 2

Ниже приведен пример EDD, показывающий объединение двух источников. На рисунке показано, как приложение EDD может отображать диаграмму.

```c
VARIABLE primary_value {
    LABEL "Primary Value";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT "pH";
}

VARIABLE primary_value_undamped {
    LABEL "Undamped PV";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT "pH";
}

AXIS axis_0_14 {
    LABEL "axis 0-14";
    MIN_VALUE 0;
    MAX_VALUE 14;
}

SOURCE primary_value_stack1 {
    MEMBERS {
        PRIM_VAL_1, primary_value;
    }
    Y_AXIS axis_0_14;
    LINE_COLOR 0x0000FF; /*BLUE*/
}

SOURCE primary_value_stack2 {
    MEMBERS {
        PRIM_VAL_2, primary_value_undamped;
    }
    Y_AXIS axis_0_14;
    LINE_COLOR 0xFF0000; /*RED*/
}

CHART primary_value_stack_chart {
    LABEL "Primary Value";
    MEMBERS {
        CHART1, primary_value_stack1, "ph1";
        CHART2, primary_value_stack2, "ph2";
    }
    TYPE VERTICAL_BAR;
}
```

<figure><img src="../../.gitbook/assets/image (2).png" alt="" width="254"><figcaption></figcaption></figure>

## Пример 3

Ниже приведен пример EDD, на котором показано объединение трех горизонтальных источников. На рисунке показано, как это может отображаться в приложении EDD.

```c
VARIABLE primary_value {
    LABEL "Primary Value";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT "pH";
}

VARIABLE primary_value_undamped {
    LABEL "Undamped PV";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT "pH";
}

VARIABLE primary_value_non_temperature_compensated {
    LABEL "PV non-TC";
    CLASS DYNAMIC;
    TYPE FLOAT;
    CONSTANT_UNIT "pH";
}

AXIS axis_0_14 {
    LABEL "axis 0-14";
    MIN_VALUE 0;
    MAX_VALUE 14;
}

AXIS axis_0_7 {
    LABEL "axis 0-7";
    MIN_VALUE 0;
    MAX_VALUE 7;
}
SOURCE primary_value_stack1 {
    MEMBERS {
        PRIM_VAL_1, primary_value;
    }
    Y_AXIS axis_0_14;
    LINE_COLOR 0x0000FF; /*BLUE*/
}

SOURCE primary_value_stack2 {
    MEMBERS {
        PRIM_VAL_2, primary_value_undamped;
    }
    Y_AXIS axis_0_14;
    LINE_COLOR 0xFF0000; /*RED*/
}

SOURCE primary_value_single {
    MEMBERS {
        PRIM_VAL_3, primary_value_non_temperature_compensated;
    }
    Y_AXIS axis_0_7;
    LINE_COLOR 0x008000; /*GREEN*/
}

CHART primary_value_stack_chart {
    LABEL "Primary Value";
    MEMBERS {
        CHART1, primary_value_stack1, "pH1";
        CHART2, primary_value_single, "pH non-TC";
        CHART3, primary_value_stack2, "pH2";
    }
    TYPE HORIZONTAL_BAR;
}
```

<figure><img src="../../.gitbook/assets/image (3).png" alt="" width="451"><figcaption></figcaption></figure>

В **SOURCE** можно определить необязательные **INIT\_ACTIONS** и/или **REFRESH\_ACTIONS**.\
Один и тот же метод может быть использован как в списке **INIT\_ACTIONS**, так и в **REFRESH\_ACTIONS**.

Ниже приведён пример использования методов в графике. В данном примере показан график в диалоговом окне.

```c
MENU measuring_values {
    LABEL "Measuring Values";
    STYLE DIALOG;
    ITEMS {
        primary_value_view
    }
}

MENU primary_value_view {
    LABEL "Primary Measuring Value";
    STYLE PAGE;
    ITEMS {
        primary_value_chart
    }
}

VARIABLE primary_value {
    LABEL "Primary Value";
    CLASS DYNAMIC;
    TYPE FLOAT;
}

SOURCE primary_value_source {
    LABEL "Primary"; // this label is used in the legend
    LINE_TYPE DATA1;
    EMPHASIS TRUE;
    Y_AXIS measuring_values_axis;
    MEMBERS {
        PRIM_VAL, primary_value;
    }
}
METHOD CalculationMethod {
    LABEL "";
    DEFINITION {
        calculated_value = primary_value * 0.12345;
    }
}

AXIS measuring_values_axis {
    LABEL "measurement value";
    MIN_VALUE 0;
    MAX_VALUE 100;
}

VARIABLE primary_value_unit {
    LABEL "Primary Value Unit";
    CLASS CONTAINED;
    TYPE ENUMERATED(1) {
        { 32, [degC], [degC help] },
        { 33, [degF], [degF_help] },
        { 35, [Kelvin], [Kelvin_help] }
    }
}

UNIT {
    primary_value_unit:
    primary_value,
    calculated_value,
    measuring_values_axis
}
SOURCE calculated_value_source {
    LABEL "calculated";
    LINE_TYPE DATA2;
    Y_AXIS measuring_values_axis;
    MEMBERS {
        CALC_VAL, calculated_value;
    }
    INIT_ACTIONS { CalculationMethod }
    REFRESH_ACTIONS { CalculationMethod }
}

CHART primary_value_chart {
    LABEL "Primary Value";
    LENGTH 600000;
    TYPE SCOPE;
    WIDTH_LARGE;
    HEIGHT SMALL;
    CYCLE_TIME 5000; /*Source value refresh of 5000 msec*/
    MEMBERS {
        GRAPH1, primary_value_source;
        GRAPH2, calculated_value_source;
    }
}
```
