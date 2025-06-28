# Dokumentasi Setup YOLO dengan WSL Ubuntu dan Docker

## Prasyarat
- Windows 10/11 dengan WSL2 terinstall
- Docker Desktop terinstall dan terintegrasi dengan WSL
- Ubuntu WSL terinstall
- Kamera yang terhubung ke sistem

## Status Sistem
### 1. Pemeriksaan WSL
```bash
wsl -l -v
```
Output yang diharapkan:
```
  NAME              STATE           VERSION
* docker-desktop    Running         2      
  Ubuntu            Running         2
```

### 2. Pemeriksaan Docker
```bash
docker ps
docker images
```
Output yang diharapkan untuk `docker images`:
```
REPOSITORY                TAG       IMAGE ID       CREATED        SIZE  
ollama/ollama             latest    45008241d610   45 hours ago   3.45GB
devlikeapro/waha          latest    aaf8c2c639d8   2 days ago     3.96GB
ultralytics/ultralytics   latest    8de1cf9f1aee   3 days ago     31.8GB
bitnami/laravel           latest    c761e5190769   2 weeks ago    1.14GB
```

### 3. Pemeriksaan Akses Kamera
```bash
ls -l /dev/video*
```
Output yang diharapkan:
```
crw-rw---- 1 root video 81, 0 Jun 28 12:29 /dev/video0      
crw-rw---- 1 root video 81, 1 Jun 28 12:29 /dev/video1      
crw-rw---- 1 root video 81, 2 Jun 28 12:30 /dev/video2      
crw-rw---- 1 root video 81, 3 Jun 28 12:30 /dev/video3
```

### 4. Pemeriksaan Network
```bash
ip addr
```
Output penting yang perlu diperhatikan:
```
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 172.24.238.76/20
docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP>
    inet 172.17.0.1/16
```

## Menjalankan Container YOLO

### 1. Persiapan Direktori
Pastikan Anda berada di direktori kerja yang diinginkan:
```bash
cd /mnt/d/Cursor/server  # Sesuaikan dengan path Anda
```

### 2. Menjalankan Container
```bash
docker run -it --rm \
    --network host \
    --device=/dev/video0:/dev/video0 \
    --device=/dev/video1:/dev/video1 \
    --device=/dev/video2:/dev/video2 \
    --device=/dev/video3:/dev/video3 \
    -v /mnt/d/Cursor/server:/workspace \
    ultralytics/ultralytics
```

Parameter yang digunakan:
- `-it`: Mode interaktif dengan terminal
- `--rm`: Otomatis menghapus container setelah selesai
- `--network host`: Menggunakan network stack host
- `--device`: Mounting device kamera dari host ke container
- `-v`: Mounting volume direktori kerja

## Penggunaan YOLO

### 1. Deteksi Objek dari Kamera
```bash
# Menggunakan kamera pertama (video0)
yolo predict model=yolov8n.pt source=0

# Menggunakan kamera kedua (video1)
yolo predict model=yolov8n.pt source=1
```

### 2. Deteksi Objek dari File
```bash
# Deteksi pada file video
yolo predict model=yolov8n.pt source=/workspace/video.mp4

# Deteksi pada file gambar
yolo predict model=yolov8n.pt source=/workspace/image.jpg
```

### 3. Menyimpan Hasil Deteksi
```bash
# Menyimpan hasil ke direktori tertentu
yolo predict model=yolov8n.pt source=0 save=True project=/workspace/results
```

## Troubleshooting

### 1. Masalah Akses Kamera
Jika terjadi masalah akses kamera, periksa:
- Permission device kamera (`ls -l /dev/video*`)
- Status WSL (`wsl --status`)
- Koneksi kamera ke sistem

### 2. Masalah Network
Jika terjadi masalah network:
- Periksa status Docker (`docker ps`)
- Periksa konfigurasi network (`ip addr`)
- Restart Docker Desktop jika diperlukan

### 3. Masalah Volume
Jika terjadi masalah akses file:
- Periksa mounting point (`docker inspect [container_id]`)
- Periksa permission direktori
- Pastikan path yang digunakan benar

## Referensi
- [Dokumentasi Ultralytics YOLO](https://docs.ultralytics.com/)
- [Dokumentasi Docker](https://docs.docker.com/)
- [Dokumentasi WSL](https://learn.microsoft.com/en-us/windows/wsl/) 
