# Dokumentasi: Akses Kamera USB di WSL2 dengan usbipd-win

## Tujuan

Memberikan akses kamera USB dari host Windows ke WSL2 (dalam kasus ini Docker Desktop), agar dapat digunakan untuk aplikasi seperti OpenCV, YOLO, atau sistem AI Vision lainnya.

## Syarat Sistem

* Windows 10/11 Build 22000+ dengan WSL2 aktif
* WSL2 terpasang (contoh: docker-desktop)
* Installer `usbipd-win` versi terbaru (.msi) dari GitHub

---

## Langkah 0 – Instalasi `usbipd-win`

### 0.1 Unduh Installer Resmi

1. Kunjungi halaman resmi GitHub: [https://github.com/dorssel/usbipd-win/releases](https://github.com/dorssel/usbipd-win/releases)
2. Pilih versi terbaru.
3. Unduh file bernama: `usbipd-win-x.x.x.msi` (contoh: `usbipd-win-5.1.0.msi`)
4. Jalankan file tersebut dan ikuti wizard instalasi hingga selesai.

### 0.2 Verifikasi Instalasi

Buka PowerShell dan jalankan:

```powershell
usbipd --version
```

Jika berhasil, akan muncul versi yang terinstal.

---

## Langkah 1 – Cek dan Identifikasi Kamera USB

1. Buka **PowerShell sebagai Administrator**
2. Jalankan:

   ```powershell
   usbipd list
   ```
3. Identifikasi kamera dari daftar, contoh:

   ```
   BUSID  VID:PID    DEVICE                      STATE
   1-6    0bda:5671  Integrated Webcam           Not shared
   1-14   0bda:565e  Rear Integrated Webcam      Not shared
   ```

---

## Langkah 2 – Bind (Share) Kamera dari Host

Jalankan perintah berikut di PowerShell Administrator:

```powershell
usbipd bind --busid 1-6     # Integrated Webcam
usbipd bind --busid 1-14    # Rear Integrated Webcam
```

Setelah ini, status kamera akan menjadi `Shared`.

---

## Langkah 3 – Attach Kamera ke WSL2

Untuk Docker Desktop:

```powershell
usbipd attach --busid 1-6 --wsl
usbipd attach --busid 1-14 --wsl
```

Jika distro WSL lain digunakan, seperti Ubuntu:

```powershell
usbipd attach --busid 1-6 --wsl Ubuntu
usbipd attach --busid 1-14 --wsl Ubuntu
```

Setelah berhasil, status menjadi `Attached`.

---

## Langkah 4 – Verifikasi di WSL2

Masuk ke WSL:

```bash
ls /dev/video*
```

Hasil yang diharapkan:

```bash
/dev/video0
/dev/video1
```

Lanjutkan dengan:

```bash
v4l2-ctl --list-devices
```

---

## Langkah 5 – Tes Akses Kamera dengan OpenCV (Opsional)

```bash
pip install opencv-python
python
```

```python
import cv2
cap = cv2.VideoCapture(0)
ret, frame = cap.read()
print("Kamera Aktif:", ret)
```

---

## Langkah 6 – Melepas Akses (Detach)

Jika ingin mengembalikan kamera ke kontrol Windows:

```powershell
usbipd detach --busid 1-6
usbipd detach --busid 1-14
```

---

## Catatan Penting

| Perangkat USB       | Boleh Attach ke WSL2? | Catatan                          |
| ------------------- | --------------------- | -------------------------------- |
| Webcam              | ✅ Ya                  | Kompatibel                       |
| Bluetooth           | ❌ Tidak disarankan    | Akan mematikan Bluetooth Windows |
| Fingerprint Reader  | ❌ Tidak berguna       | Tidak didukung di Linux          |
| USB Network Adapter | ⚠️ Eksperimen         | Gunakan hanya jika dibutuhkan    |

---

## Status Akhir

```text
1-6    Integrated Webcam        → Attached
1-14   Rear Integrated Webcam   → Attached
```

WSL2 sekarang dapat membaca kamera melalui `/dev/video0` dan `/dev/video1`. Siap digunakan untuk AI Vision atau aplikasi lainnya.
