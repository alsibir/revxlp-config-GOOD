Причина неисправности, как правило, связана не с синтаксисом devicetree, а с семантикой `zmk,behavior-tap-dance`:

1. Поведение tap-dance формирует выходное действие только после выбора варианта (по числу нажатий) и обычно ожидает истечения `tapping-term-ms` после последнего нажатия, прежде чем «выпустить» выбранный binding. ([ZMK Firmware][1])
2. В текущей конфигурации первый binding задан как `&kp LSHFT`. Нажатие Shift само по себе не печатает символ и визуально «ничего не делает», поэтому создаётся впечатление отсутствия срабатывания.
3. Если цель состоит в получении «одиночное нажатие — Shift для следующей клавиши», то `&kp LSHFT` не является «липким» Shift. Для такого сценария предназначен sticky key `&sk`, который удерживает модификатор до следующего нажатия. ([ZMK Firmware][2])
4. При использовании tap-dance для «Shift на одиночный tap» возможна ситуация, когда следующая клавиша нажимается раньше, чем tap-dance успевает выбрать и применить первый binding (из-за ожидания `tapping-term-ms`). Это воспринимается как «не срабатывает».

Практическое исправление для сценария «одиночный tap — Shift для следующей клавиши, двойной tap — caps_word» состоит в замене первого binding на sticky shift:

```dts
tap_dances {
    shifty: shift_caps_word {
        compatible = "zmk,behavior-tap-dance";
        display-name = "Shift+";
        #binding-cells = <0>;
        tapping-term-ms = <TAP_DANCE_TERM_MS>;
        bindings = <&sk LSHFT>, <&caps_word>;
    };
};
```

Пояснения к исправлению:

* `&sk LSHFT` реализует «липкий Shift» (модификатор действует до следующей клавиши), что соответствует ожидаемому эффекту от одиночного нажатия. ([ZMK Firmware][2])
* `&caps_word` является отдельным поведением и корректно вызывается без параметров. ([ZMK Firmware][3])

Дополнительная настройка (при необходимости):

* Если двойной tap распознаётся нестабильно, допускается увеличить `TAP_DANCE_TERM_MS` (например, до 180–220 мс). Следует учитывать, что увеличение `tapping-term-ms` одновременно увеличивает задержку, после которой выбирается одиночный вариант tap-dance. ([ZMK Firmware][1])

Если требуется именно «удержание — обычный Shift без задержки» (модификатор должен начинать действовать сразу при удержании, без ожидания решения tap-dance), то tap-dance для Shift является конструктивно неподходящим механизмом; в таком случае рекомендуется переносить Shift на отдельный `&kp LSHFT` или проектировать поведение через hold-tap, а `caps_word` выносить на отдельную клавишу/комбо. ([ZMK Firmware][4])

[1]: https://zmk.dev/docs/keymaps/behaviors/tap-dance?utm_source=chatgpt.com "Tap-Dance Behavior"
[2]: https://zmk.dev/docs/keymaps/behaviors/sticky-key?utm_source=chatgpt.com "Sticky Key Behavior"
[3]: https://zmk.dev/docs/keymaps/behaviors/caps-word "Caps Word Behavior | ZMK Firmware"
[4]: https://zmk.dev/docs/keymaps/behaviors/hold-tap?utm_source=chatgpt.com "Hold-Tap Behavior"
