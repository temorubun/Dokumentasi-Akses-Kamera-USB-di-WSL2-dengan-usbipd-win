# Dokumentasi: Mengakses Kamera di WSL2 untuk YOLO Server

## 1. Instalasi Awal

### a. Install WSL2 dan Distro Ubuntu

```powershell
wsl --install -d Ubuntu
```

Ikuti proses instalasi hingga Anda diminta membuat username dan password untuk akun Ubuntu.

### b. Periksa Versi dan Distribusi

```powershell
wsl -l -v
```

Output akan menampilkan:

```
  NAME              STATE           VERSION
* docker-desktop    Running         2
  Ubuntu            Running         2
```

### c. Set Ubuntu sebagai default WSL (opsional tapi direkomendasikan)

```powershell
wsl --set-default Ubuntu
```

Setelah ini, Anda bisa masuk ke Ubuntu hanya dengan perintah:

```powershell
wsl
```

## 2. Instalasi usbipd-win (jika belum)

Unduh dan pasang dari:

> [https://github.com/dorssel/usbipd-win/releases](https://github.com/dorssel/usbipd-win/releases)

Pastikan `usbipd` bisa dijalankan di PowerShell:

```powershell
usbipd list
```

## 3. Bind dan Attach Kamera dari Windows ke WSL2

### a. Lihat daftar perangkat USB

```powershell
usbipd list
```

Contoh output:

```
BUSID  VID:PID    DEVICE                        STATE
1-6    0bda:5671  Integrated Webcam             Not shared
1-14   0bda:565e  Rear Integrated Webcam        Not shared
```

### b. Bind perangkat (kamera)

```powershell
usbipd bind --busid 1-6
usbipd bind --busid 1-14
```

### c. Attach perangkat ke WSL (Ubuntu)

```powershell
usbipd attach --busid 1-6 --wsl Ubuntu
usbipd attach --busid 1-14 --wsl Ubuntu
```

Jika berhasil, status kamera akan menjadi `Attached`.

## 4. Verifikasi Kamera di WSL Ubuntu

Masuk ke Ubuntu:

```powershell
wsl -d Ubuntu
```

Lalu di dalam Ubuntu:

```bash
ls /dev/video*
```

Output contoh:

```
/dev/video0  /dev/video1  /dev/video2  /dev/video3
```

Artinya kamera berhasil dikenali.

## 5. Menjalankan Container YOLO dengan Akses Kamera

Masih dari Ubuntu WSL:

```bash
export DOCKER_HOST=unix:///mnt/wsl/docker-desktop/shared-sockets/host-services/docker.sock

docker run -it --rm \
  --device=/dev/video0:/dev/video0 \
  --device=/dev/video1:/dev/video1 \
  --device=/dev/video2:/dev/video2 \
  --device=/dev/video3:/dev/video3 \
  -v /mnt/d/Cursor/yolo-server:/workspace/scripts \
  ultralytics/ultralytics /bin/bash
```

Container ini sekarang:

* Berjalan dari WSL Ubuntu
* Mengakses kamera fisik host
* Menggunakan image YOLO yang tersimpan di Docker Desktop

---

Jika Anda ingin agar perintah `DOCKER_HOST` aktif otomatis saat masuk Ubuntu, cukup tambahkan ke `.bashrc` atau minta saya bantu.

Dokumentasi ini siap dijadikan bagian dari laporan proyek, skripsi, atau automation script.
