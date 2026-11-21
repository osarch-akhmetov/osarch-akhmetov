# Lab 5 — Encoding forensics

Name: Akhmetov · Date: 2026-11-21 · Tools: Ubuntu Server (VirtualBox), Excel, LibreOffice Calc

---

## What I received

- **Excel (double-click):** Opened in columns separated by `;`, but all Cyrillic and Kazakh characters appeared as mojibake (`Р¤Р°РјРёР»РёСЏ` / Windows-1252 misinterpretation) because the file had no BOM.
- **LibreOffice Calc (import dialog):** Defaulted to Western European (`Windows-1252` / system default) character set, showing unreadable Latin-1 symbols in the preview table.
- **Linux terminal (`cat evidence/staff-broken.csv`):** Displayed replacement diamonds (``) for every Cyrillic and Kazakh byte because the terminal expects UTF-8.

---

## Diagnosis (made before the fix)

- **file said:** `evidence/staff-broken.csv: Non-ISO extended-ASCII text, with CRLF line terminators` (`evidence/file-before.txt`)
- **First 16 bytes:** `49 44 3b d4 e0 ec e8 eb e8 ff 3b c8 ec ff 3b ce` (`ID;Фамилия;Имя;О...`) (`evidence/hexdump-before.txt`)
- **Encoding:** `KZ-1048` (`RK1048` in `iconv`) — because every Cyrillic letter is stored as a single byte (`e4 e0 ec e8 eb e8 ff` = `Фамилия`), and row `101` (`31 30 31 3b` at offset `0x27`) contains single-byte Kazakh-specific characters **`a3`** (`Ә`) and **`b3`** (`і`) in `Әбдірахманова`, which do not exist in standard Windows-1251.
- **Line ending:** `CRLF` — because line 1 ends with the two-byte sequence **`0d 0a`** at offset `0x23`.
- **BOM:** `No` — because the file starts immediately with ASCII **`49 44 3b`** (`ID;`) rather than `EF BB BF` or `FF FE`; in the fixed file I **added a UTF-8 BOM (`EF BB BF`) and kept `CRLF` line endings** so that Windows Excel recognizes UTF-8 automatically on double-click without manual import steps.
- **Separator:** `;` (semicolon, byte `3b`).

---

## Fix

Script saved at [`scripts/fix.sh`](./scripts/fix.sh):

```sh
#!/bin/sh
# Lab 5: convert staff-broken.csv from KZ-1048 (RK1048) to UTF-8 with BOM and CRLF
in=evidence/staff-broken.csv
out=evidence/staff-fixed.csv
iconv -f RK1048 -t UTF-8 "$in" > "$out"
printf '\357\273\277' | cat - "$out" > tmp && mv tmp "$out"
