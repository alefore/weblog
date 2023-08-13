# Linux: Keyboard Configuration

I configure my keyboard using `xkbcomp`.

I had originally used a `xmodmap` file.
This used to work well in practice a decade ago.
But something happened that made it very slow to apply.
So I migrated away from `xmodmap` to `xkbcomp`.

I apply my configurations with this command:

    xkbcomp -I$HOME/.xkb $HOME/.xkb/map $DISPLAY

## Master file

My master file starts with the `us` keyboard and includes two custom files,
`accepts` and `caps`.

File ~/.xkb/map:

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

## Spanish accents

The first custom file sets accents for Spanish.

File ~/.xkb/symbols/accents:

    partial alphanumeric_keys
    xkb_symbols "accents" {
      key <RALT> {
        type[group1]= "PC_LALT_LEVEL2",
        symbols[Group1]= [ ISO_Level3_Shift,  ISO_Next_Group ]
      };
      replace key <SPCE> {[ space, space, space, nobreakspace ]};
      replace key <AE01> {[ 1, exclam, exclamdown ]};
      replace key <AD03> {[ e, E, eacute, Eacute ]};
      replace key <AD07> {[ u, U, uacute, Uacute ]};
      replace key <AD08> {[ i, I, iacute, Iacute ]};
      replace key <AD09> {[ o, O, oacute, Oacute ]};
      replace key <AC01> {[ a, A, aacute, Aacute ]};
      replace key <AB06> {[ n, N, ntilde, Ntilde ]};
      replace key <AB10> {[ slash, question, questiondown ]};
    };

## Caps Lock

I disable caps lock. I never found it useful. Instead, I map it to tab.

File ~/.xkb/symbols/caps:

    hidden partial modifier_keys
    xkb_symbols "caps" {
      replace key <CAPS> {[ Tab ]};
    };

