# Terminal Manager

[English](README.md) · **Bahasa Indonesia**

Launcher profil terminal untuk Windows yang menjalankan terminalnya **di dalam window-nya sendiri**,
seperti panel terminal di IDE. Simpan shell-mu sebagai profil, kelompokkan, lalu buka sebagai grid
pane yang semuanya hidup — bukan window yang berserakan.

Tanpa runtime tambahan, tanpa dependensi. Satu `.exe`.

```
+-- Local ---------+---------------+
| PS C:\> npm run  | $ git status  |
| ready on :3000   | On branch main|
| _                | _             |
+------------------+---------------+
| wsl:~$ htop      | ssh prod      |
| CPU  32%         | root@prod:~#  |
| _                | _             |
+------------------+---------------+
```

Terminalnya asli. Terminal Manager berbicara langsung dengan Windows pseudo console (ConPTY), jadi
warna ANSI, kontrol kursor, resize, dan program TUI layar penuh seperti `vim`, `htop`, atau prompt
password `ssh` berperilaku persis seperti di konsol biasa.

---

## Instalasi

1. Unduh `TerminalManagerSetup.exe` dari
   [rilis terbaru](https://github.com/alenovan/terminal-manager/releases/latest).
2. Jalankan.

Hanya itu yang didistribusikan — satu installer, aplikasinya sudah tertanam di dalamnya.

Secara default dipasang **per pengguna**, ke `%LOCALAPPDATA%\Programs\TerminalManager`, sehingga
Windows tidak memunculkan prompt UAC. Centang **Install for all users** untuk memasang ke
`Program Files`; installer akan meminta hak administrator lalu menjalankan salinan dirinya sendiri
secara elevated.

Installer membuat shortcut Desktop dan Start Menu (opsional) serta mendaftarkan aplikasi di
**Add or remove programs**, jadi bisa dicopot seperti program lain.

`profiles.json` tidak ikut dalam installer. File itu dibuat saat aplikasi pertama dijalankan, berisi
beberapa profil contoh yang bisa kamu ganti nama atau hapus.

### Kebutuhan

| | |
|---|---|
| Windows | 10 versi 1809 atau lebih baru, untuk pseudo console |
| .NET | Framework 4.x — sudah ada di setiap Windows 10 dan 11 |

Di Windows lama aplikasinya tetap jalan, tapi profil jatuh ke window terpisah.

### Instalasi senyap

```
TerminalManagerSetup.exe /S                        pasang dengan pengaturan default
TerminalManagerSetup.exe /S /allusers              pasang ke Program Files (butuh admin)
TerminalManagerSetup.exe /S /dir:"C:\path"         tentukan foldernya
TerminalManagerSetup.exe /S /nodesktop /nostartmenu
```

### Mencopot

Lewat **Add or remove programs**, atau jalankan `uninstall.exe` di folder instalasi. Profilmu tetap
disimpan kecuali kamu mencentang opsi untuk menghapusnya.

```
uninstall.exe /uninstall /S            copot senyap, profil disimpan
uninstall.exe /uninstall /S /purge     copot senyap, profil ikut dihapus
```

---

## Penggunaan

### Grup dan profil

Sidebar berisi **grup** — folder biasa seperti `Local`, `Dev`, `Servers` — dan tiap grup berisi
**profil**. Satu profil adalah satu konfigurasi terminal: program yang dijalankan, argumennya,
folder kerja, environment variable, dan cara membukanya.

Semuanya ditulis ke `profiles.json` begitu ada perubahan. Formatnya JSON yang rapi; boleh diedit
tangan kalau lebih suka.

### Membuka terminal

Tekan **Launch** (atau `Ctrl+Enter`, atau klik dua kali profil di sidebar). Terminalnya terbuka
sebagai pane **di dalam window**.

Tekan Launch lagi dan terminal berikutnya bergabung ke tab yang sama sebagai pane baru, memecah
mengikuti sisi terpanjang. Itu perilaku default: Launch berulang menumbuhkan grid seimbang, bukan
tumpukan strip tipis.

**Launch whole group** membuka semua profil in-app satu grup sekaligus, tersusun sebagai grid — dua
profil berdampingan, empat jadi 2x2.

Kalau ingin dipisah, pakai **New tab** (`Ctrl+Shift+Enter`).

### Pane

| | |
|---|---|
| Pecah ke kanan / ke bawah | `Alt+Shift+D` / `Alt+Shift+E` |
| Pindah antar pane | `Alt`+tombol panah |
| Ubah ukuran | seret garis pemisah antar pane |
| Pindahkan pane ke tab sendiri | tombol di header pane, atau `Alt+Shift+T` |
| Lipat pane kembali ke tab sebelumnya | `Alt+Shift+G` |
| Tutup pane | tanda silang di header-nya, atau `Ctrl+Shift+W` |

Memindahkan pane antar tab **tidak** me-restart prosesnya. Kontrolnya hanya berganti induk; shell-nya
tetap hidup beserta seluruh scrollback-nya.

Menutup pane atau tab menghentikan proses **beserta semua anaknya**, karena tiap sesi berjalan di
dalam job object. `npm run dev` yang dijalankan di sebuah pane tidak bisa lolos jadi proses yatim.

### Mode peluncuran

| Mode | Terminal terbuka di mana |
|---|---|
| `Inline` | di dalam window ini |
| `Auto` | di dalam window ini |
| `Windows Terminal` | window Windows Terminal terpisah |
| `Console` | window konsol terpisah |

Di dalam aplikasi adalah default. Membuka ke luar adalah pilihan eksplisit.

Dua kasus otomatis jatuh ke window terpisah, dan status bar menjelaskan alasannya:

- Profil **Run as Administrator**, kecuali Terminal Manager sendiri sedang elevated. Proses
  medium-integrity tidak bisa menyerahkan pseudo console ke anak high-integrity, karena elevasi UAC
  lewat `ShellExecute` yang tidak menerima attribute list. Pakai **Restart as admin** di toolbar
  untuk membuka ulang seluruh aplikasi secara elevated — setelah itu semua terminal di dalamnya juga
  elevated.
- Windows lebih lama dari 10 versi 1809, yang belum punya pseudo console.

### Menyimpan sesi

Shell yang hidup tidak bisa dibawa melewati restart — Windows tidak menyediakan cara memindahkan
proses beserta pseudo console-nya ke sesi baru. Yang bisa disimpan adalah susunannya, atau teks di
layarnya.

**Buka lagi seperti terakhir ditutup.** Tab dan pane yang terbuka ditulis ke `session.json` saat
aplikasi ditutup: profil apa, di posisi mana, dengan rasio pemisah berapa. Saat dibuka lagi
susunannya dibangun ulang dan shell-nya dijalankan **baru** — prompt bersih, bukan output lama.
Profil yang dihapus di antaranya tidak menggagalkan restore; daunnya diciutkan dan saudaranya
mengambil alih ruangnya.

**Workspace bernama.** **Save layout** (`Ctrl+Shift+L`) menyimpan grid tab yang sedang aktif dengan
sebuah nama. Workspace muncul di sidebar di atas daftar grup, ditandai ikon 2x2 dan jumlah pane-nya.
Klik dua kali untuk membukanya. Menghapus workspace tidak menghapus profil yang ditunjuknya.

**Simpan outputnya.** **Save output** (`Ctrl+Shift+S`) menulis seluruh scrollback pane yang sedang
fokus — sampai 5000 baris — ke file `.log` atau `.txt` sebagai teks polos, tanpa warna ANSI.

### Menyeleksi dan menyalin

Seret untuk menyeleksi, klik dua kali untuk satu kata, tiga kali untuk satu baris. `Ctrl+Shift+C`
menyalin, `Ctrl+Shift+V` menempel, klik kanan juga menempel. Bracketed paste didukung, jadi shell
yang mengaktifkannya menerima tempelan multi-baris dengan aman.

Tombol lainnya diteruskan ke shell. `Ctrl+C`, `Ctrl+S`, `Ctrl+N`, dan seterusnya sampai ke proses,
bukan ke aplikasi.

---

## Field profil

| Field | Fungsinya |
|---|---|
| Profile name | Nama di sidebar, sekaligus judul tab default |
| Group | Memindahkan profil ke grup lain |
| Shell preset | Mengisi otomatis Executable dan Arguments. Pilih `Custom` untuk mengisi sendiri |
| Launch mode | Lihat tabel di atas |
| Executable | Program yang dijalankan, misal `pwsh.exe` |
| Arguments | Argumen mentah, misal `-NoLogo -NoExit` |
| Working directory | Folder awal. Environment variable diterjemahkan: `%USERPROFILE%` |
| Environment variables | `KEY=VALUE`, satu per baris. Baris diawali `#` diabaikan |
| Tab title | Judul tab untuk Windows Terminal |
| Windows Terminal profile | Profil WT yang dipakai (`-p`), untuk mewarisi warna dan font-nya |
| Run as Administrator | Jalankan elevated (memunculkan prompt UAC) |

Tersedia preset untuk PowerShell 5.1, PowerShell 7, Command Prompt, WSL, Git Bash, SSH, Node, dan
Python.

---

## Pintasan papan ketik

### Aplikasi

| Tombol | Aksi |
|---|---|
| `Ctrl+Enter` | Jalankan profil terpilih |
| `Ctrl+Shift+Enter` | Jalankan di tab baru |
| `Ctrl+S` | Simpan form profil |
| `Ctrl+F` | Fokus ke kotak pencarian |
| `Ctrl+N` | Profil baru |
| `F2` | Ganti nama |
| `Ctrl+Tab` / `Ctrl+Shift+Tab` | Pindah tab |
| `Ctrl+Shift+L` | Simpan tab aktif sebagai workspace |
| `Ctrl+Shift+S` | Simpan output pane yang fokus ke file |

### Pane

| Tombol | Aksi |
|---|---|
| `Alt+Shift+D` / `Alt+Shift+E` | Pecah ke kanan / ke bawah |
| `Alt`+panah | Pindah antar pane |
| `Alt+Shift+T` | Pindahkan pane ke tab sendiri |
| `Alt+Shift+G` | Lipat pane ke tab sebelumnya |
| `Ctrl+Shift+W` | Tutup pane |

### Di dalam terminal

| Tombol | Aksi |
|---|---|
| `Ctrl+Shift+C` / `Ctrl+Shift+V` | Salin / tempel (klik kanan juga menempel) |
| `Ctrl+Shift+A` | Pilih semua |
| `Shift+PgUp` / `Shift+PgDn` | Gulir scrollback |
| `Ctrl+Shift+Home` / `Ctrl+Shift+End` | Ke awal / akhir scrollback |
| `Ctrl+=` / `Ctrl+-` / `Ctrl+0` | Perbesar / perkecil / reset ukuran font |
| `Ctrl+R` | Jalankan ulang sesi yang prosesnya sudah selesai |

---

## Catatan teknis

- **Terminalnya.** `CreatePseudoConsole` plus `CreateProcess` dengan
  `PROC_THREAD_ATTRIBUTE_PSEUDOCONSOLE`. Output dibaca di thread latar dan dikuras oleh timer UI
  16 ms dengan jatah byte per tick, sehingga banjir output tidak membekukan antarmuka.
- **Renderer-nya.** Sel digambar dengan `ExtTextOut` beserta array advance eksplisit, jadi kolomnya
  presisi dan tidak melenceng di baris panjang. Deretan sel bergaya sama digambar dalam satu panggilan.
- **Emulator VT-nya** subset yang praktis, bukan xterm lengkap: SGR termasuk 256 warna dan truecolor,
  gerak kursor, scroll region, alternate buffer, bracketed paste, dan mouse reporting. Resize tidak
  me-reflow teks yang sudah tercetak — kolomnya tetap di posisi lama, seperti conhost sebelum era
  reflow.
- **Pohon proses.** Tiap sesi dimasukkan ke job object dengan kill-on-close, jadi menutup pane
  merobohkan seluruh pohonnya.
- **Environment variable** diserahkan ke ConPTY sebagai blok environment sungguhan. Pembungkus `.cmd`
  sementara hanya dibutuhkan untuk peluncuran eksternal, di mana `ShellExecute` — yang wajib untuk
  UAC — tidak bisa membawanya.
- **Data portable.** `profiles.json` berada di sebelah executable selama foldernya bisa ditulis, dan
  jatuh ke `%APPDATA%\TerminalManager` kalau tidak, misalnya saat dipasang di `Program Files`. Path
  yang sedang aktif selalu terlihat di status bar.

---

## Lisensi

[MIT](LICENSE)
