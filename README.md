# ⚡ Arduino Uno Synchronous Buck MPPT Solar Charge Controller

[![Lisensi: MIT](https://img.shields.io/badge/Lisensi-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: Arduino Uno](https://img.shields.io/badge/Platform-Arduino%20Uno%20|%20ATmega328P-blue.svg)](#)
[![Framework: Arduino IDE](https://img.shields.io/badge/Framework-Arduino%20IDE%202.0%2B-teal.svg)](https://www.arduino.cc/)
[![Status: Firmware Produksi](https://img.shields.io/badge/Status-Firmware%20Produksi-brightgreen.svg)](#)
[![Developer: Muhammad Fikri](https://img.shields.io/badge/Developer-Muhammad%20Fikri-blue.svg)](#)

Maximum Power Point Tracking solar charger implementing Perturb and Observe (P&O) and Incremental Conductance algorithms with high-frequency synchronous buck PWM.

---

## 📊 Diagram Blok Arsitektur & Skema Alur Rangkaian

Visualisasi interaktif alur daya, akuisisi sinyal sensor, pemrosesan algoritma inti, dan aktuasi proteksi perangkat:

```mermaid
graph TD
    subgraph Power_Source ["⚡ Sumber Energi & Jaringan"]
        PV["Panel Surya / Baterai / AC Mains"] --> SENS_V["Sensor Tegangan (ZMPT101B / Divider)"]
        PV --> SENS_I["Sensor Arus (ACS712 / Shunt)"]
        SENS_V -->|"Analog Signal"| MCU["🧠 Arduino Uno (ATmega328P 16MHz)"]
        SENS_I -->|"Analog Signal"| MCU
    end

    subgraph Energy_Processing ["🧠 Algoritma & Power DSP"]
        MCU -->|"Discrete Sampling"| TRMS["True-RMS & Active Power Math"]
        TRMS -->|"MPPT / State Engine"| CTRL["P&O MPPT / Load Shedding Logic"]
        CTRL -->|"Fast PWM (Timer1/MCPWM)"| GATE["Gate Driver Stage"]
    end

    subgraph Load_Protection ["🔋 Manajemen Beban & Storage"]
        GATE -->|"Switching MOSFET/SSR"| LOAD["Beban Listrik / Bank Baterai"]
        MCU -->|"I2C Visual"| DISP["Layar Monitor Daya Real-Time"]
        MCU -->|"Cloud Telemetry"| CLOUD["RS485 Modbus RTU Telemetry"]
    end

    style MCU fill:#1565c0,stroke:#0d47a1,stroke-width:2px,color:#fff
    style TRMS fill:#2e7d32,stroke:#1b5e20,stroke-width:2px,color:#fff
    style GATE fill:#bf360c,stroke:#870000,stroke-width:2px,color:#fff
```

---

## 📦 Daftar Komponen & Bahan Lengkap (Bill of Materials - BOM)

Berikut rincian spesifikasi komponen fisik dan modul yang dibutuhkan untuk membangun proyek ini:

| No | Nama Komponen / Modul | Estimasi Jumlah | Fungsi & Spesifikasi Teknis |
|:---|:---|:---|:---|
| 1 | **Arduino Uno R3 (ATmega328P)** | 1 Unit | Mikrokontroler 8-bit deterministik 16MHz |
| 2 | **Adaptor Daya DC 9V-12V 1A / USB 5V** | 1 Unit | Sumber daya listrik stabil dengan proteksi arus |
| 3 | **Sensor Arus Hall Effect ACS712 / INA219 / Shunt Resistor** | 1-2 Unit | Pengukuran arus DC/AC presisi tinggi |
| 4 | **Modul Sensor Tegangan ZMPT101B / Precision Voltage Divider** | 1-2 Unit | Pengukuran tegangan True-RMS jaringan listrik |
| 5 | **Power MOSFET N-Channel (IRFZ44N / FDP047N10) / SSR Relay** | 2-4 Unit | Sakelar daya switching kontrol beban dan MPPT |
| 6 | **Induktor Daya & Kapasitor Filter Low-ESR** | 1 Set | Komponen penyaring riak daya switching konverter |
| 7 | **Layar OLED SSD1306 / LCD 20x4 I2C** | 1 Unit | Tampilan metrik Watt, Volt, Ampere, dan Efisiensi |

---

## 🧠 Arsitektur Sistem & Fitur Utama

- **Deterministic Non-Blocking State Machine:** Memisahkan pemrosesan sinyal presisi tinggi dari task telemetri untuk mencegah *latency jitter*.
- **Digital Signal Processing (DSP) & Filtering:** Dilengkapi algoritma digital filtering terdedikasi untuk eliminasi derau sinyal analog.
- **Non-Volatile Storage (Internal EEPROM):** Parameter kalibrasi, *setpoint*, dan konfigurasi tersimpan secara persisten terhadap siklus pemadaman daya.
- **Hardware Failsafe & Emergency Interlock:** Perlindungan otomatis jika terjadi anomali tegangan, kelebihan beban arus, atau pemicuan tombol *Emergency Stop*.
- **Industrial Telemetry & Diagnostics:** Pelaporan status operasional secara real-time via Serial/JSON stream.

---

## 🔌 Skema Pinout & Koneksi Hardware

| Komponen / Sinyal | Pin (Arduino Uno) | Deskripsi Fungsi |
|:---|:---|:---|
| **Sensor Analog Input** | `Pin A0` | Jalur pembacaan sensor utama berpresisi tinggi |
| **Emergency Stop (E-Stop)** | `Pin 2 (INT0)` | Pemicu pengaman darurat hardware interrupt |
| **Actuator / Relay Utama** | `Pin 9 (PWM) / Pin 7` | Pengendali beban daya tinggi / relay aktuator |
| **Acoustic Alarm Buzzer** | `Pin 8` | Indikator peringatan audible saat terjadi anomali |
| **Status / Heartbeat LED** | `Pin 13` | Indikator status aktivitas sistem real-time |

---

## 🛠️ Panduan Perakitan Hardware (Langkah Demi Langkah)

1. **Persiapan Catu Daya:** Hubungkan catu daya utama ke jalur daya mikrokontroler. Pasang kapasitor *decoupling* 100nF di dekat pin VCC untuk meredam ripple switching.
2. **Pemasangan Sensor & Modul:** Sambungkan jalur sinyal sensor ke pin mikrokontroler yang telah ditentukan. Gunakan resistor pull-up 4.7kΩ pada jalur SDA/SCL jika menggunakan modul I2C.
3. **Pemasangan Aktuator:** Hubungkan modul relay / gate driver MOSFET ke pin kontrol output. Pasang dioda *flyback* (1N4007) pada beban induktif untuk mengeliminasi lonjakan tegangan balik (*back-EMF*).
4. **Pemasangan Tombol Emergency Stop:** Sambungkan tombol darurat ke pin interupsi eksternal dengan konfigurasi *Active-LOW* menggunakan resistor *pull-up*.
5. **Verifikasi Koneksi:** Lakukan pengecekan jalur ground bersama (*Common Ground*) pada seluruh modul sebelum menyalakan daya.

---

## 🚀 Panduan Kompilasi & Upload (Arduino IDE)

1. Buka **Arduino IDE 2.0+**.
2. Masuk ke menu **Tools > Board**:
   * Pilih **`Arduino Uno`**.
3. Pastikan dependensi pustaka terpasang via Library Manager:
   * `ArduinoJson`
   * `Wire` & `SPI`
   * `EEPROM`
4. Buka berkas [`arduino-uno-mppt-solar-charger.ino`](./arduino-uno-mppt-solar-charger.ino).
5. Klik tombol **Verify** (✓) kemudian **Upload** (➔).
6. Buka **Serial Monitor** pada baudrate **`115200`** untuk melihat streaming telemetri dan status operasional.

---

## 📄 Lisensi
Didistribusikan di bawah lisensi open-source **MIT License**. Dibuat dengan ❤️ oleh **Muhammad Fikri Dev**.
