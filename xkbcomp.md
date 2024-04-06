# Linux: Keyboard Configuration

I configure my keyboard using `xkbcomp`.

## Master file

My master file starts with the `us` keyboard and includes two custom files,
`accents` and `caps`.

File `~/.xkb/map`:

    xkb_keymap {
      xkb_keycodes { include "evdev+aliases(qwerty)" };
      xkb_types { include "complete" };
      xkb_compat { include "complete" };
      xkb_symbols {
        include "pc+us+us:2+inet(evdev)"
        include "accents(accents)"
        include "caps(caps)"
      };
      xkb_geometry  { include "pc(pc105)" };
    };

## Spanish & German support

The first custom file sets accents for Spanish and diaeresis for German.
It also sets a few other custom characters, such as `emdash`, `nobreakspace`,
arrow characters (`←↑↓→`), and `×`.

File `~/.xkb/symbols/accents`:

    partial alphanumeric_keys
    xkb_symbols "accents" {
      // Map the right Alt key (`RALT`) to act as an ISO_Level3_Shift,
      // ISO_Next_Group key, providing access to third-level symbols or
      // characters.
      key <RALT> {
        type[group1]= "PC_LALT_LEVEL2",
        symbols[Group1]= [ ISO_Level3_Shift,  ISO_Next_Group ]
      };

      replace key <SPCE> {[ space, space, space, nobreakspace ]};
      replace key <AE11> {[ minus, underscore, endash, emdash ]};
      replace key <AB09> {[ period, greater, U2026 ]};

      // Spanish characters.
      replace key <AE01> {[ 1, exclam, exclamdown ]};
      replace key <AD03> {[ e, E, eacute, Eacute ]};
      replace key <AD07> {[ u, U, uacute, Uacute ]};
      replace key <AD08> {[ i, I, iacute, Iacute ]};
      replace key <AD09> {[ o, O, oacute, Oacute ]};
      replace key <AC01> {[ a, A, aacute, Aacute ]};
      replace key <AB06> {[ n, N, ntilde, Ntilde ]};
      replace key <AB10> {[ slash, question, questiondown ]};

      // German characters. To avoid clashes with the definitions for Spanish
      // characters, I define the special keys *below* the corresponding normal
      // characters (e.g., `ä` in `z`, the key physically below `a`).
      replace key <AC03> {[ d, D, ediaeresis, Ediaeresis ]};
      replace key <AC07> {[ j, J, udiaeresis, Udiaeresis ]};
      replace key <AC08> {[ k, K, idiaeresis, Idiaeresis ]};
      replace key <AC09> {[ l, L, odiaeresis, Odiaeresis ]};
      replace key <AB01> {[ z, Z, adiaeresis, Adiaeresis ]};
      replace key <AB02> {[ x, X, ssharp, U00D7 ]};

      // Arrows.
      replace key <LEFT> {[ Left, Left, U2190 ]};
      replace key <UP>   {[ Up, Up, U2191 ]};
      replace key <DOWN> {[ Down, Down, U2193 ]};
      replace key <RGHT> {[ Right, Right, U2192 ]};
    };

## Caps Lock

I disable caps lock.
I never found it useful.

Instead, I map it to Control.
I use Control very frequently
and I find it uncomfortable to press Control
in the default position in most keyboards.

File `~/.xkb/symbols/caps`:

    hidden partial modifier_keys
    xkb_symbols "caps" {
      replace key <CAPS> {[ Control_L ]};
      modifier_map Control { <CAPS> };
    };

## Applying this configuration

### Executing xkbcomp directly

I can apply my configurations with this command:

    xkbcomp -I$HOME/.xkb ~/.xkb/map $DISPLAY

### Integration with Udev

My keyboard is connected to a USB switch.
Occasionally, the configuration is dropped, as if I had unplugged the keyboard and plugged it back in.
To apply my settings every time this happens, I wrote the following script:

File `~/bin/xkb` (mode 755):

    #!/bin/bash
    export XAUTHORITY="/run/user/$(id -u alejo)/gdm/Xauthority"
    export DISPLAY=":1"
    (sleep 2 ; date ; xkbcomp -I$(eval echo ~alejo/.xkb) ~alejo/.xkb/map $DISPLAY ) >/tmp/xkb.log 2>&1 &

I register it with this file:

File `/etc/udev/rules.d/99-custom-kb.rules`:

    ACTION=="add", SUBSYSTEM=="input", ENV{ID_INPUT_KEYBOARD}=="1", RUN+="/home/alejo/bin/xkb"

The reason for the call to `sleep 2` is that my distribution comes with some `/etc/udev/rules.d/99-...` files that override my settings.
I don't want to edit (or move) any of these files;
I fear this would cause ongoing pain
(e.g., having to restore things or manage conflicts whenever my distribution is updated).

I opted for the ugly solution:
for the first ~2 seconds after the keyboard is re-detected
(which happens a few times per day), my customizations are missing.

## Xmodmap

I had originally used an `xmodmap` file with all my bindings.
This work well in practice up until some point around 2014;
something happened that made it incredibly slow.
So I migrated away from `xmodmap` to `xkbcomp`.

