# zmk-auto-shift

QMK-style global auto shift for ZMK. Every plain `&kp` letter, digit, and punctuation key gets
auto shift. Tap it and you get the normal character. Hold it past the tapping term and you get the
shifted one. You don't need per-key macros.

## Behavior

- Hold past `tapping-term-ms` and the module sends one shifted tap (press + release). The physical
  release is swallowed. The shifted character doesn't auto-repeat.
- The pending key resolves unshifted when it's released, when another keycode is pressed, or when
  another key position is pressed. Releasing other keys doesn't resolve it.
- Presses that are already older than the term are never shifted. Mod-tap holds count here, so
  `&mt LEFT_BRACKET LEFT_BRACE` held still sends `[`.
- Keys with implicit or explicit mods, and keys pressed while any explicit mod is held, are never
  shifted.
- On split keyboards the module only runs on the central half.

## Installation

Add the module to your `config/west.yml`:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: nicholasptran
      url-base: https://github.com/nicholasptran
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-auto-shift
      remote: nicholasptran
      revision: main
  self:
    path: config
```

## Usage

Declare one instance in your keymap's `behaviors {}` block. Its presence turns the feature on:
`CONFIG_ZMK_AUTO_SHIFT` defaults to `y` when a `zmk,behavior-auto-shift` node exists. Bind the
instance to a key to toggle auto shift on and off.

```dts
/ {
    behaviors {
        as_tog: auto_shift_toggle {
            compatible = "zmk,behavior-auto-shift";
            #binding-cells = <0>;
            tapping-term-ms = <170>;
            enabled-on-boot;
        };
    };

    keymap {
        compatible = "zmk,keymap";
        some_layer {
            bindings = <&as_tog /* ... */>;
        };
    };
};
```

| Property          | Type    | Default | Description                                         |
| ----------------- | ------- | ------- | --------------------------------------------------- |
| `tapping-term-ms` | int     | `170`   | How long a key must be held before it sends shifted. |
| `enabled-on-boot` | boolean | off     | Start with auto shift enabled.                      |

## Implementation note

`CMakeLists.txt` adds the source with `target_sources(app ...)`, not `zephyr_library()`. ZMK
doesn't sort its event subscription section, so listeners run in link order. Module sources link
before ZMK's `hid_listener.c`, and this listener has to run before it.
