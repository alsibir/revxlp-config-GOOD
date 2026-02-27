# revxlp ZMK user configuration (xiao_ble//zmk)

## 1 Назначение репозитория
Репозиторий предназначен для хранения и воспроизводимой сборки конфигурации прошивки ZMK для самодельной клавиатуры `revxlp` на контроллере Seeed Studio XIAO BLE (nRF52840). Конфигурация ориентирована на сборку строго под целевую плату `xiao_ble//zmk` и набор shield:
- `revxlp` — основная прошивка клавиатуры;
- `settings_reset` — прошивка сброса настроек ZMK.

Сборка выполняется через GitHub Actions (reusable workflow ZMK) и, при необходимости, локально с использованием `west`.

## 2 Поддерживаемые цели сборки
- Плата (board): `xiao_ble//zmk`
- Shield: `revxlp`, `settings_reset`

В репозитории предусмотрена проверка на отсутствие устаревших обозначений плат (например, `seeeduino _xiao_ble`, `seeeduino _xiao`) во всех файлах проекта.

## 3 Краткое описание аппаратной конфигурации
- Контроллер: Seeed Studio XIAO BLE (nRF52840).
- Матрица клавиатуры: 6 столбцов × 7 строк.
- Управление столбцами: сдвиговый регистр 74HC595 (SPI), сканирование матрицы реализовано на стороне shield `revxlp`.

## 4 Структура репозитория
Типовая структура (может расширяться):
- `build.yaml` — матрица сборки GitHub Actions (board/shield).
- `config/`
  - `revxlp.conf` — конфигурация Kconfig для целевого shield.
  - `revxlp.keymap` — раскладка ZMK для shield.
- `boards/`
  - `shields/revxlp/` — описание shield, overlay и привязки к плате.
- `.github/workflows/`
  - `build.yml` — pipeline сборки и верификации.

## 5 Сборка через GitHub Actions
1. Выполнить отправку изменений в ветку `main` или открыть pull request.
2. Перейти в GitHub: вкладка Actions → выбрать workflow сборки.
3. По завершении job скачать артефакты (файлы прошивок `.uf2`) из результата выполнения workflow.

Результаты сборки формируются на основе `build.yaml`. При изменении board/shield следует вносить изменения только в `build.yaml` и связанные файлы конфигурации.

## 6 Локальная сборка (west)
### 6.1 Предварительные требования
- Linux (рекомендуется Ubuntu/Debian-совместимые).
- `west` и зависимости ZMK/Zephyr.
- Установленный Zephyr SDK (версия определяется используемым стеком ZMK/Zephyr).

Примечание: наиболее воспроизводимый вариант локальной сборки — использование официального окружения ZMK (контейнер/докер или dev container), чтобы исключить расхождения версий инструментов.

### 6.2 Команды сборки
Пример сборки основной прошивки:
```bash
west build -s zmk/app -d build/revxlp -b xiao_ble//zmk -- \
  -DZMK_CONFIG="$(pwd)/config" \
  -DSHIELD=revxlp
````

Пример сборки прошивки сброса:

```bash
west build -s zmk/app -d build/settings_reset -b xiao_ble//zmk -- \
  -DZMK_CONFIG="$(pwd)/config" \
  -DSHIELD=settings_reset
```

Готовые файлы `.uf2` формируются в каталоге сборки (как правило, `build/<target>/zephyr/`).

## 7 Прошивка контроллера (UF2)

1. Перевести XIAO BLE в режим загрузчика UF2 (обычно двойное нажатие кнопки Reset).
2. В системе появится USB-накопитель (диск загрузчика).
3. Скопировать нужный файл `.uf2` на этот накопитель.
4. Дождаться автоматического перезапуска контроллера.

Рекомендуемый порядок при устранении проблем с настройками:

1. прошить `settings_reset`;
2. затем прошить основную прошивку `revxlp`.

## 8 Внесение изменений в раскладку и конфигурацию

* Раскладка: редактирование `config/revxlp.keymap`.
* Параметры Kconfig: редактирование `config/revxlp.conf`.
* Описание аппаратной части shield (матрица, линии, SPI и пр.): `boards/shields/revxlp/*`.

После изменений рекомендуется:

1. выполнить локальную проверку (сборка или статическая валидация);
2. отправить изменения в репозиторий и дождаться успешного прохождения GitHub Actions.

## 9 Контроль именования плат и типовые проверки

Для предотвращения использования устаревших имен плат в репозитории применяется проверка в CI. При необходимости локальной проверки:

```bash
grep -RIn --exclude-dir=.git --exclude-dir=build \
  -E 'seeeduino _xiao_ble|seeeduino _xiao([^_]|$)' .
```

Также рекомендуется контролировать отсутствие устаревших/неопределённых Kconfig-символов в `config/*.conf` перед отправкой изменений.

## 10 Файлы в корневом каталоге репозитрия для изучения раскладки revxlp

1. [`howto_layers.md`](howto_layers.md)
   Назначение: сводная фиксация логики слоёв в текущей конфигурации раскладки.
   Что можно получить:

   * перечень слоёв, их идентификаторы и назначение (Base, Nav, Num, Adjust, Media, Mouse, Reserved);
   * способы включения каждого слоя (включая tri-layer для Adjust) и связи между слоями;
   * выявленные ограничения текущих привязок и условия работоспособности (включая требования для слоя Mouse и указание на `&to BASE_L` как механизм принудительного возврата на базовый слой).

2. [`howto_combos.md`](howto_combos.md)
   Назначение: методический материал по применению комбинаций клавиш (combos) в ZMK с акцентом на настройку и диагностику.
   Что можно получить:

   * описание принципа обработки combos и места их объявления в `.keymap`;
   * требования к структуре узла `combos`, обязательные свойства и типовые параметры (`timeout-ms`, `layers`, `slow-release`, `require-prior-idle-ms`);
   * правила определения позиций (`key-positions`) и практические рекомендации по подбору значений;
   * перечень типовых причин «пробивания букв» вместо срабатывания комбинации и способы устранения;
   * пример расширенной конфигурации и разбор причин неработоспособности combos применительно к текущему репозиторию, включая рекомендуемые правки без изменения общей логики слоёв.

3. [`howto_atributes.md`](howto_atributes.md)
   Назначение: расширенный справочник по чтению привязок `bindings` и интерпретации поведений ZMK, используемых в раскладке.
   Что можно получить:

   * общие правила чтения devicetree-привязок в `.keymap` и проверки числа/смысла аргументов;
   * описание поведений, начинающихся с `&` (включая `&kp`, `&mt`, `&lt`, `&mo`, `&to`, `&trans`, `&bt`, `&out`, `&sys_reset`, `&bootloader`, `&mkp`, `&mmv`, `&msc`, `&shifty`);
   * систематизацию параметров после `&` (идентификаторы слоёв, команды Bluetooth и вывода, параметры эмуляции мыши, модификаторы и модификаторные функции, keycodes);
   * минимальный порядок действий для разработки и сопровождения раскладки (контроль корректности поведений, параметров и предусловий, включая требования для pointing).

4. [`howto_atributes+.md`](howto_atributes+.md)
   Назначение: укороченный «памятный лист» по тем же атрибутам и аргументам, применяемым в слоях раскладки.
   Что можно получить:

   * краткие определения ключевых поведений и их формат применения;
   * перечень типовых аргументов для слоёв, Bluetooth, вывода, модификаторов, keycodes и мыши;
   * контрольные указания по проверке числа аргументов и типовых ошибок при записи привязок.


## 11 Источники

1. `https://github.com/petejohanson/revxlp-config`
   Назначение: эталонный (референсный) репозиторий конфигурации ZMK для проекта revxlp; используется как источник примеров структуры пользовательской конфигурации, подходов к организации слоёв, биндингов и сопутствующих файлов.

2. `https://github.com/devpew/revxlp-config`
   Назначение: исходные материалы (донорская конфигурация), на основе которых выполнена адаптация текущего репозитория; используется для сопоставления изменений, переноса элементов раскладки и проверки совместимости.

3. `https://nickcoutsos.github.io/keymap-editor/`
   Назначение: веб-инструмент Keymap Editor для визуального редактирования раскладок ZMK и сохранения изменений в репозиторий через GitHub-интеграцию. ([nickcoutsos.github.io][1])

4. `https://github.com/nickcoutsos/keymap-editor/wiki/Limitations-of-the-Keymap-Editor`
   Назначение: ограничения парсинга Keymap Editor, требования к структуре репозитория ZMK user config и правила подготовки layout-описаний (в т.ч. `config/info.json`) для корректного отображения раскладки. ([GitHub][2])

5. `https://github.com/caksoylar/keymap-drawer`
   Назначение: инструмент keymap-drawer для генерации статических изображений слоёв и схем раскладки на основе описания keymap; применяется для подготовки иллюстраций в документации.

6. `https://zmk.dev/docs/`
   Назначение: официальная документация ZMK (общее описание возможностей, структура конфигурации, навигация по разделам). ([zmk.dev][3])

7. `https://zmk.dev/docs/keymaps`
   Назначение: базовые принципы описания раскладок ZMK (devicetree keymap), структура слоёв и привязок `bindings`. ([zmk.dev][4])

8. `https://zmk.dev/docs/keymaps/behaviors/layers`
   Назначение: поведения управления слоями (`&mo`, `&lt`, `&to`, и др.), применяемые в текущей конфигурации. ([zmk.dev][5])

9. `https://zmk.dev/docs/keymaps/conditional-layers`
   Назначение: conditional layers (tri-layer), используемые для включения слоя `Adjust` при одновременной активности `Nav` и `Num`. ([zmk.dev][6])

10. `https://zmk.dev/docs/keymaps/behaviors/hold-tap`
    Назначение: параметры и принципы работы hold-tap/layer-tap (`&lt`, `&mt`), включая особенности определения «тап/удержание». ([zmk.dev][7])

11. `https://zmk.dev/docs/keymaps/behaviors/tap-dance`
    Назначение: документация по tap-dance, применяемому для поведения `shifty`. ([zmk.dev][8])

12. `https://zmk.dev/docs/keymaps/behaviors/mouse-emulation`
    Назначение: mouse emulation (`&mkp`, `&mmv`, `&msc`) и требования к подключению `dt-bindings/zmk/pointing.h`, используемые в слое `Mouse Layer`. ([zmk.dev][9])

13. `https://zmk.dev/docs/config/pointing`
    Назначение: конфигурация поддержки мышиных HID-событий, включая ключевой параметр `CONFIG_ZMK_POINTING`. ([zmk.dev][10])

14. `https://zmk.dev/docs/keymaps/list-of-keycodes`
    Назначение: справочник keycodes (включая сокращённые формы), используемых в `&kp ...` и смежных поведениях. ([zmk.dev][11])

15. `https://zmk.dev/docs/user-setup`
    Назначение: порядок организации репозитория пользовательской конфигурации и настройка `build.yaml` для сборки прошивок через GitHub Actions. ([zmk.dev][12])

16. `https://github.com/zmkfirmware/zmk/blob/main/.github/workflows/build-user-config.yml`
    Назначение: эталонный reusable workflow ZMK для сборки user config; используется для проверки актуальных входных параметров и механики сборки в CI. ([GitHub][13])

17. `https://wiki.seeedstudio.com/XIAO_BLE/`
    Назначение: документация производителя по Seeed Studio XIAO BLE (nRF52840), включая особенности платы, базовые процедуры работы и справочные сведения. ([wiki.seeedstudio.com][14])

18. `https://nickcoutsos.github.io/keymap-layout-tools/`
    Назначение: Keymap Layout Helper для визуальной проверки и отладки `info.json`/layout-описаний (позиции, порядок, переупорядочивание), применяемых для корректного отображения раскладки в редакторах. ([nickcoutsos.github.io][15])

[1]: https://nickcoutsos.github.io/keymap-editor/?utm_source=chatgpt.com "Keymap Editor"
[2]: https://github.com/nickcoutsos/keymap-editor/wiki/Limitations-of-the-Keymap-Editor?utm_source=chatgpt.com "Limitations of the Keymap Editor"
[3]: https://zmk.dev/docs?utm_source=chatgpt.com "Introduction to ZMK"
[4]: https://zmk.dev/docs/keymaps?utm_source=chatgpt.com "Keymaps & Behaviors"
[5]: https://zmk.dev/docs/keymaps/behaviors/layers?utm_source=chatgpt.com "Layer Behaviors"
[6]: https://zmk.dev/docs/keymaps/conditional-layers?utm_source=chatgpt.com "Conditional Layers"
[7]: https://zmk.dev/docs/keymaps/behaviors/hold-tap?utm_source=chatgpt.com "Hold-Tap Behavior"
[8]: https://zmk.dev/docs/keymaps/behaviors/tap-dance?utm_source=chatgpt.com "Tap-Dance Behavior"
[9]: https://zmk.dev/docs/keymaps/behaviors/mouse-emulation?utm_source=chatgpt.com "Mouse Emulation Behaviors"
[10]: https://zmk.dev/docs/config/pointing?utm_source=chatgpt.com "Pointing Device Configuration"
[11]: https://zmk.dev/docs/keymaps/list-of-keycodes?utm_source=chatgpt.com "List of Keycodes"
[12]: https://zmk.dev/docs/user-setup?utm_source=chatgpt.com "Installing ZMK"
[13]: https://github.com/zmkfirmware/zmk/blob/main/.github/workflows/build-user-config.yml?utm_source=chatgpt.com "build-user-config.yml - zmk"
[14]: https://wiki.seeedstudio.com/XIAO_BLE/?utm_source=chatgpt.com "Getting Started with Seeed Studio XIAO nRF52840 Series"
[15]: https://nickcoutsos.github.io/keymap-layout-tools/?utm_source=chatgpt.com "Keymap Layout Helper"
