# ESP-RUN: Arduino CLI Automation for ESP32

esp-run.sh adalah skrip otomatisasi berbasis Bash yang dirancang untuk menyederhanakan alur kerja pengembangan ESP32 menggunakan Arduino CLI. Skrip ini sangat cocok digunakan di lingkungan terminal seperti Linux atau Ubuntu chroot di perangkat Android (Termux).

## Fitur Utama

* **Advanced Board Setup**: Konfigurasi FQBN secara dinamis (ESP32-C3, ESP32 Dev Module, dll) mencakup CPU Frequency, Flash Mode, Partition Scheme, hingga USB CDC On Boot.
* **Library Manager**: Cari, instal, dan kelola library Arduino secara real-time langsung dari skrip.
* **Auto-Retry Port Detection**: Sistem tunggu otomatis selama 10 detik saat melakukan flashing untuk memberikan waktu bagi pengguna mencolok kabel USB OTG.
* **Hardware Diagnostics**: Integrasi dengan lsusb untuk memastikan hardware terdeteksi secara elektrik meskipun port serial belum muncul.
* **Manual Port Override**: Opsi untuk memasukkan path port secara manual jika deteksi otomatis gagal.
* **Project Management**: Kemudahan berpindah antar proyek di folder Arduino dan manajemen file konfigurasi .esp_config per proyek.

## Persyaratan Sistem

Sebelum menjalankan skrip ini, pastikan sistem Anda sudah memiliki:

* **Curl**: Untuk mengunduh engine Arduino CLI secara otomatis.
* **Nano**: Digunakan sebagai teks editor default dalam skrip.
* **Lsusb (usbutils)**: Untuk fitur diagnosa hardware.
* **Sudo**: Diperlukan untuk memberikan izin (chmod) pada port USB.

## Cara Instalasi & Penggunaan

1. **Unduh skrip**:
   ```bash
   git clone https://github.com/Dhans62/ESP-FLASHER.git Arduino
   ```

3. **Berikan izin eksekusi**:
   ```bash
   chmod +x esp-run.sh
   ```

5. **Jalankan skrip**:
   ```bash
   ./esp-run.sh
   ```

**Catatan**: Pada saat pertama kali dijalankan, skrip akan mengunduh dan mengonfigurasi arduino-cli secara otomatis jika belum tersedia.

## Credits

Proyek ini dibangun dengan dukungan dari:

* **Arduino IDE / Arduino CLI**
  https://github.com/arduino/arduino-cli
* **Ubuntu chroot**
https://github.com/ravindu644/Ubuntu-Chroot.git

**By Dhann**
Dibuat untuk mempermudah ekosistem pengembangan embedded system di lingkungan mobile terminal.
