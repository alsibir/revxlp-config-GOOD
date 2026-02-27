## 1 Общие правила чтения и применения привязок

В ZMK каждая ячейка в `bindings = < ... >;` описывается как «вызов поведения» (behavior binding) в форме:

* `&<поведение> <параметр 1> <параметр 2> ...`

Количество и смысл параметров определяется конкретным поведением (его `binding-cells`). Поведения и ключевые коды задаются через devicetree-синтаксис в `.keymap`. ([ZMK Firmware][1])

---

## 2 Атрибуты, начинающиеся с `&` (поведения)

### 2.1 `&kp` — отправка кода клавиши (key press)

Назначение: отправка указанного keycode (нажатие/отпускание) на хост. ([ZMK Firmware][2])
Форма применения:

* `&kp <keycode>`
* `&kp <модификаторная функция>(<keycode>)` (см. `LS()`, `LC()`, `LG()` ниже)

Примеры из слоёв:

* `&kp ESC`, `&kp Q`, `&kp PG_UP`, `&kp C_VOL_UP`
* `&kp LG(LS(N4))` — комбинация модификаторов через «modifier functions». ([ZMK Firmware][3])

### 2.2 `&mt` — mod-tap (удержание модификатора / tap кода)

Назначение: «удержание/тап»; при удержании срабатывает первый параметр (mod), при коротком нажатии — второй (tap). По умолчанию решение зависит от времени удержания и/или прерывания другой клавишей. ([ZMK Firmware][4])
Форма применения:

* `&mt <mod> <tap_keycode>`

Примеры из слоёв:

* `&mt LCTRL TAB`
* `&mt RALT GRAVE`

### 2.3 `&lt` — layer-tap (удержание слоя / tap кода)

Назначение: при удержании включает слой, при коротком нажатии отправляет keycode. ([ZMK Firmware][4])
Форма применения:

* `&lt <layer_id> <tap_keycode>`

Примеры из слоёв:

* `&lt MOU_L LGUI`
* `&lt NAV_L RET`

### 2.4 `&mo` — momentary layer (слой на удержании)

Назначение: включает слой на время удержания клавиши. ([ZMK Firmware][5])
Форма применения:

* `&mo <layer_id>`

Примеры из слоёв:

* `&mo NUM_L`, `&mo MED_L`, `&mo FUN_L`

### 2.5 `&to` — переход на слой (to layer)

Назначение: включает указанный слой и выключает прочие активные слои (кроме базового). ([ZMK Firmware][6])
Форма применения:

* `&to <layer_id>`

Пример из слоёв:

* `&to BASE_L`

### 2.6 `&trans` — прозрачная клавиша (transparent)

Назначение: игнорирует нажатие на текущем слое и «пропускает» событие на нижележащий активный слой. ([ZMK Firmware][7])
Форма применения:

* `&trans`

### 2.7 `&bt` — управление Bluetooth профилями

Назначение: выполнение Bluetooth-команд (выбор профиля, очистка привязки и т. п.). Команды задаются как `BT_*`, некоторые требуют числовой аргумент профиля (0-индексация). ([ZMK Firmware][8])
Форма применения:

* `&bt <BT_команда> [параметр]`

Примеры из слоёв:

* `&bt BT_CLR`
* `&bt BT_SEL 4`

### 2.8 `&out` — выбор канала вывода (USB/BLE)

Назначение: выбор/переключение предпочитаемого канала вывода (USB или BLE); команды `OUT_*` берутся из `dt-bindings/zmk/outputs.h`. ([ZMK Firmware][9])
Форма применения:

* `&out <OUT_команда>`

Пример из слоёв:

* `&out OUT_TOG`

### 2.9 `&sys_reset` и `&bootloader` — перезагрузка

Назначение:

* `&sys_reset` — сброс устройства
* `&bootloader` — перезапуск с переходом в режим загрузчика для прошивки ([ZMK Firmware][10])
  Форма применения:
* `&sys_reset`
* `&bootloader`

### 2.10 `&mkp`, `&mmv`, `&msc` — эмуляция мыши

Назначение: отправка событий кнопок мыши, движения курсора и прокрутки. Требуется включение `CONFIG_ZMK_POINTING=y` и использование `dt-bindings/zmk/pointing.h`. ([ZMK Firmware][11])
Форма применения:

* `&mkp <кнопка>` — кнопка мыши (например, `LCLK`, `RCLK`, `MCLK`) ([ZMK Firmware][11])
* `&mmv <направление>` — движение (например, `MOVE_LEFT`, `MOVE_UP`) ([ZMK Firmware][11])
* `&msc <направление>` — прокрутка (например, `SCRL_DOWN`, `SCRL_RIGHT`) ([ZMK Firmware][11])

### 2.11 `&shifty` — пользовательское поведение (tap-dance по метке)

Назначение: вызов пользовательского behavior по метке devicetree (node label). В предоставленной конфигурации `shifty` реализован как tap-dance, который выбирает действие по числу нажатий. ([ZMK Firmware][12])
Форма применения в слоях:

* `&shifty` (параметров нет, поскольку `#binding-cells = <0>` в определении поведения)

---

## 3 Атрибуты после `&` (параметры поведения и keycodes)

### 3.1 Идентификаторы слоёв (параметры для `&mo`, `&lt`, `&to`)

В качестве параметров используются числовые индексы слоёв; для удобства применяются `#define`-алиасы. ([ZMK Firmware][6])
Используемые в фрагменте:

* `BASE_L`, `NAV_L`, `NUM_L`, `MED_L`, `FUN_L`, `MOU_L`

Применение:

* `&mo NUM_L`
* `&lt NAV_L RET`
* `&to BASE_L`

### 3.2 Команды Bluetooth (параметры для `&bt`)

Команды определяются в `dt-bindings/zmk/bt.h`. ([ZMK Firmware][8])
Используемые в фрагменте:

* `BT_CLR` — очистка привязки текущего профиля
* `BT_SEL <n>` — выбор профиля по номеру (0-индексация), пример: `BT_SEL 4`

Применение:

* `&bt BT_CLR`
* `&bt BT_SEL 4`

### 3.3 Команды выбора канала вывода (параметры для `&out`)

Команды определяются в `dt-bindings/zmk/outputs.h`. ([ZMK Firmware][9])
Используемые в фрагменте:

* `OUT_TOG` — переключение между USB и BLE

Применение:

* `&out OUT_TOG`

### 3.4 Параметры эмуляции мыши (параметры для `&mkp`, `&mmv`, `&msc`)

Определяются в `dt-bindings/zmk/pointing.h`. ([ZMK Firmware][11])
Используемые в фрагменте:

* Для `&mkp`: `LCLK`, `MCLK`, `RCLK`
* Для `&mmv`: `MOVE_LEFT`, `MOVE_DOWN`, `MOVE_UP`, `MOVE_RIGHT`
* Для `&msc`: `SCRL_LEFT`, `SCRL_DOWN`, `SCRL_UP`, `SCRL_RIGHT`

### 3.5 Модификаторы и модификаторные функции (используются внутри `&kp`, а также как параметры `&mt`/`&lt`)

Модификаторы могут применяться:

1. как самостоятельные keycodes (`LCTRL`, `RALT`, `LGUI`, `LSHFT` и т. п.);
2. как «modifier functions» вида `XX(code)` с возможностью вложения. ([ZMK Firmware][3])

Используемые в фрагменте:

* Модификаторы как параметры:

  * `LCTRL` (в `&mt LCTRL TAB`)
  * `RALT` (в `&mt RALT GRAVE`)
  * `LGUI` (в `&lt MOU_L LGUI`)
* Модификаторные функции:

  * `LG(LS(N4))` — пример вложения функций: `LG()` + `LS()` для кода `N4` ([ZMK Firmware][3])
  * `LC(LEFT)` / `LC(RIGHT)` — добавление `LEFT_CONTROL` к коду направления

Примечание по унификации: в актуальной документации ZMK рекомендуется использовать формы `LEFT_ARROW`/`RIGHT_ARROW` (и аналогично для `UP_ARROW`/`DOWN_ARROW`), а алиасы `LEFT`/`RIGHT` целесообразно избегать при сопровождении. (Проверка ключевых кодов выполняется по справочнику keycodes.) ([ZMK Firmware][13])

### 3.6 Keycodes (коды клавиш), используемые в слоях

Keycodes применяются в `&kp`, а также как «tap-код» в `&mt`/`&lt`. Справочник и точные написания следует сверять со страницей «List of Keycodes», поскольку ошибка в написании приводит к трудно диагностируемым ошибкам парсинга. ([ZMK Firmware][13])

Наборы, используемые в фрагменте:

1. Базовые управляющие и редактирование:

* `ESC`, `TAB`, `RET`, `BSPC`, `SPACE`, `ENTER`

2. Буквы и цифры (основной ряд):

* `A…Z`, `N0…N9` (цифровой ряд), а также `N4` внутри `LG(LS(N4))`

3. Навигация и курсор:

* `PG_UP`, `PG_DN`, `HOME`, `END`, `UP_ARROW`, `DOWN_ARROW`, `LEFT_ARROW`, `RIGHT_ARROW`

4. Пунктуация и спецсимволы:

* `SEMI`, `SQT`, `COMMA`, `DOT`, `FSLH`, `BSLH`, `GRAVE`
* `EXCL`, `AT`, `HASH`, `DLLR`, `PRCNT`, `CARET`, `AMPS`, `LPAR`, `RPAR`, `COLON`, `UNDER`
* `LBRC`, `RBRC`, `LBKT`, `RBKT`

5. Keypad (отдельные коды цифровой клавиатуры):

* `KP_MULTIPLY`, `KP_PLUS`, `KP_MINUS`, `KP_EQUAL`, `KP_N1`, `KP_N2`

6. Media (consumer keycodes):

* `C_VOL_UP`, `C_VOL_DN`, `C_MUTE`, `C_PREV`, `C_NEXT`, `C_PLAY_PAUSE` ([ZMK Firmware][2])

---

## 4 Минимальный порядок применения при разработке и сопровождении

1. Для каждой ячейки `bindings` определить требуемое поведение (`&kp`, `&mo`, `&lt`, `&mt`, `&bt`, `&out`, и т. п.) и убедиться в корректном числе параметров по документации поведения. ([ZMK Firmware][4])
2. Для параметров:

* слои задавать через `#define`-алиасы и применять их в `&mo`/`&lt`/`&to`. ([ZMK Firmware][6])
* Bluetooth и Output команды использовать только из соответствующих `dt-bindings/*.h`. ([ZMK Firmware][8])
* keycodes сверять по «List of Keycodes», уделяя внимание точному написанию. ([ZMK Firmware][13])

3. При использовании эмуляции мыши обеспечить включение `CONFIG_ZMK_POINTING=y` и применение `pointing.h`, затем использовать `&mkp`/`&mmv`/`&msc` с параметрами `LCLK`/`MOVE_*`/`SCRL_*`. ([ZMK Firmware][11])

[1]: https://zmk.dev/docs/keymaps?utm_source=chatgpt.com "Keymaps & Behaviors"
[2]: https://zmk.dev/docs/keymaps/behaviors/key-press?utm_source=chatgpt.com "Key Press Behaviors"
[3]: https://zmk.dev/docs/keymaps/modifiers "Modifiers | ZMK Firmware"
[4]: https://zmk.dev/docs/keymaps/behaviors/hold-tap "Hold-Tap Behavior | ZMK Firmware"
[5]: https://zmk.dev/docs/keymaps/behaviors/layers?utm_source=chatgpt.com "Layer Behaviors"
[6]: https://zmk.dev/docs/keymaps/behaviors/layers "Layer Behaviors | ZMK Firmware"
[7]: https://zmk.dev/docs/keymaps/behaviors/misc?utm_source=chatgpt.com "Miscellaneous Behaviors"
[8]: https://zmk.dev/docs/keymaps/behaviors/bluetooth "Bluetooth Behavior | ZMK Firmware"
[9]: https://zmk.dev/docs/keymaps/behaviors/outputs "Output Selection Behavior | ZMK Firmware"
[10]: https://zmk.dev/docs/keymaps/behaviors/reset?utm_source=chatgpt.com "Reset Behaviors"
[11]: https://zmk.dev/docs/keymaps/behaviors/mouse-emulation "Mouse Emulation Behaviors | ZMK Firmware"
[12]: https://zmk.dev/docs/keymaps/behaviors/tap-dance?utm_source=chatgpt.com "Tap-Dance Behavior"
[13]: https://zmk.dev/docs/keymaps/list-of-keycodes?utm_source=chatgpt.com "List of Keycodes"
