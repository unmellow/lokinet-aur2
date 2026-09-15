# lokinet (local overlay)

Personal rebuild of [AUR/lokinet](https://aur.archlinux.org/packages/lokinet)
for GCC 16 / CMake 4. **Do not submit this `pkgbase` to the AUR** — the
package already exists and is maintained. Send improvements as comments on
the existing AUR package.

Package sources are licensed 0BSD (`LICENSE`, `REUSE.toml`) per
[AUR submission guidelines](https://wiki.archlinux.org/title/AUR_submission_guidelines).

## Build

```bash
sudo pacman -S --needed base-devel cmake ninja git python cppzmq nlohmann-json
makepkg -Csi
```
