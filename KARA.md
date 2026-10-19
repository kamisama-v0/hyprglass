# Fork hyprglass — catatan perawatan (kara / Omarchy)

Fork lokal dari https://github.com/hyprnux/hyprglass untuk dock Omarchy (`rosakodu.dock`, layer namespace
`omarchy-dock`). Remote `upstream` = repo asli, `origin` = fork GitHub git@github.com:kamisama-v0/hyprglass.git
(push dengan key ~/.ssh/id_ed25519_ztzyrant, akun zTzyrant). Branch kerja: `kara/hyprland-0.56`.

## Cara dipasang
- hyprpm memakai repo INI langsung: `hyprpm add /home/kara/Documents/omarchy/dock/hyprglass`
- `hyprpm.toml` memetakan commit Hyprland -> commit hyprglass yang dibangun. Pin kita:
  Hyprland 0.56.2 (`efb5099…`) -> hyprglass v0.10.0 (`84c1a5a`), bukan v0.8.1 bawaan upstream (v0.10.0 punya
  perbaikan layer `fix/layer-faint-content`).
- Config: `~/.config/hypr/looknfeel.lua`, blok `if hl.plugin.hyprglass then … else … end`
  (glass hanya untuk layer `omarchy-dock`; `enabled = false` agar jendela tidak ber-glass dan shadow global tetap mati).

## Fitur tambahan fork: lensa tepi Kyant (2026-10-19)
Port shader AGSL dari Kyant0/AndroidLiquidGlass (`io.github.kyant0:backdrop`, Apache-2.0) yang dipakai SimpMusic.
Parameter baru (global, `dark:`/`light:`, preset, Lua `hg.preset`), default MATI sehingga preset bawaan tidak berubah:
| Key | Default | Arti |
| --- | --- | --- |
| `edge_lens` | 0 | 0 = refraksi tepi hyprglass asli, 1 = lensa Kyant, di antaranya = campuran |
| `edge_lens_height` | 0.25 | lebar pita lensa dari tepi (pecahan sisi pendek; layer bermask: × 64 px × skala monitor) |
| `edge_lens_amount` | 0.5 | geser sampel maksimal KE LUAR di tepi (pecahan yang sama) |
| `spectral_dispersion` | 0 | > 0.5 = dispersi 7 warna Kyant (kuat di sudut) menggantikan aberasi RGB; kekuatan = `chromatic_aberration` |
- Box (jendela/subsurface): SDF rounded-rect + gradien (radius × 1.5) persis Kyant.
- Layer alpha-mask (dock, notifikasi, menu — surface sering selayar penuh): tepi dicari dari mask (4 arah, maks 12 langkah,
  hanya di dalam pita) karena box layer ≠ bentuk yang terlihat. Tanpa ini kartu notifikasi jadi hitam (sampel ratusan px keluar).
- Dipakai lewat preset Lua `kara-glass` (inherits `glass`) di `~/.config/hypr/looknfeel.lua`.
- Shader diuji: `glslangValidator -S frag` atas string `liquidglass.frag` (buang baris kosong di depan `#version`).

## Pengaman versi (tidak crash)
1. hyprpm hanya memuat build yang cocok dengan header Hyprland yang sedang jalan.
2. hyprglass memeriksa versi sendiri; kalau beda, ia "paused" (tidak menggambar apa-apa), bukan crash.
3. Kalau plugin tidak termuat, blok `else` di looknfeel.lua memberi dock blur Hyprland biasa.
Cek kapan saja: `hyprctl hyprglass status` (harus `active` + `version check: match`).

## Saat paket Hyprland di-update
- Sejak `hyprland 0.56.2-4` perintah `hyprpm` ada di PAKET TERPISAH `hyprpm` (tidak ikut terpasang). Tanpa itu autostart
  `hyprpm reload` gagal dan SEMUA plugin tidak termuat (gejala: "unknown config key plugin.hyprexpo.*", dock blur cadangan).
  Pasang + tandai explicit: `sudo pacman -S hyprpm hyprwayland-scanner && sudo pacman -D --asexplicit hyprpm hyprwayland-scanner`
  (`hyprwayland-scanner` dibutuhkan build plugin; sempat terhapus lewat prompt "Remove orphaned packages"). `kara-check` mengecek keduanya.

## Saat Hyprland di-update
1. `hyprctl version` -> catat commit baru. `git fetch upstream`.
2. Kalau upstream sudah punya pin untuk commit itu: rebase/merge `upstream/hyprland-X.Y` ke branch kerja
   (pertahankan pin kita kalau masih relevan).
3. Kalau belum: buat branch `kara/hyprland-X.Y` dari `upstream/hyprland-X.Y` (atau `upstream/main`), perbaiki build bila
   perlu, lalu tambah pin `["<commit hyprland>", "<commit hyprglass yang mau dibangun>"]` di `hyprpm.toml` dan commit.
4. `hyprpm update` lalu `hyprpm reload`, cek `hyprctl hyprglass status`, `hyprctl configerrors`.
5. Gagal? `hyprpm disable hyprglass` -> dock otomatis kembali ke blur biasa.
