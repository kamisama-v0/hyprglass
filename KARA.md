# Fork hyprglass — catatan perawatan (kara / Omarchy)

Fork lokal dari https://github.com/hyprnux/hyprglass untuk dock Omarchy (`rosakodu.dock`, layer namespace
`omarchy-dock`). Remote `upstream` = repo asli. Branch kerja: `kara/hyprland-0.56`.

## Cara dipasang
- hyprpm memakai repo INI langsung: `hyprpm add /home/kara/Documents/omarchy/dock/hyprglass`
- `hyprpm.toml` memetakan commit Hyprland -> commit hyprglass yang dibangun. Pin kita:
  Hyprland 0.56.2 (`efb5099…`) -> hyprglass v0.10.0 (`84c1a5a`), bukan v0.8.1 bawaan upstream (v0.10.0 punya
  perbaikan layer `fix/layer-faint-content`).
- Config: `~/.config/hypr/looknfeel.lua`, blok `if hl.plugin.hyprglass then … else … end`
  (glass hanya untuk layer `omarchy-dock`; `enabled = false` agar jendela tidak ber-glass dan shadow global tetap mati).

## Pengaman versi (tidak crash)
1. hyprpm hanya memuat build yang cocok dengan header Hyprland yang sedang jalan.
2. hyprglass memeriksa versi sendiri; kalau beda, ia "paused" (tidak menggambar apa-apa), bukan crash.
3. Kalau plugin tidak termuat, blok `else` di looknfeel.lua memberi dock blur Hyprland biasa.
Cek kapan saja: `hyprctl hyprglass status` (harus `active` + `version check: match`).

## Saat Hyprland di-update
1. `hyprctl version` -> catat commit baru. `git fetch upstream`.
2. Kalau upstream sudah punya pin untuk commit itu: rebase/merge `upstream/hyprland-X.Y` ke branch kerja
   (pertahankan pin kita kalau masih relevan).
3. Kalau belum: buat branch `kara/hyprland-X.Y` dari `upstream/hyprland-X.Y` (atau `upstream/main`), perbaiki build bila
   perlu, lalu tambah pin `["<commit hyprland>", "<commit hyprglass yang mau dibangun>"]` di `hyprpm.toml` dan commit.
4. `hyprpm update` lalu `hyprpm reload`, cek `hyprctl hyprglass status`, `hyprctl configerrors`.
5. Gagal? `hyprpm disable hyprglass` -> dock otomatis kembali ke blur biasa.
