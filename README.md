#  OpenBook E-Reader

**Autor:** Roibu Radu

---

##  Prezentare Generala

**OpenBook** este un e-reader open-source construit in jurul microcontrollerului **ESP32-C6**, proiectat pentru eficienta energetica si conectivitate moderna. Integreaza un ecran e-paper de **7.5”**, senzori de mediu, gestiune avansata a bateriei si a comunicarii.

Proiectul urmareste standardele de documentare si modularitate cerute de [proiectul TSC 2025](https://ocw.cs.pub.ro/courses/tsc/proiect2025), fiind un exemplu de integrare hardware-software in embedded systems. Am incercat sa respect toate cerintele atat cat mi s-a permis.

![Poza Bottom](https://github.com/user-attachments/assets/567f6415-72f1-43f7-a293-d9e6d18dc0e8)
![Poza Top](https://github.com/user-attachments/assets/7dabf822-8540-4531-a2b5-caf79dd70128)
![Routing ](https://github.com/user-attachments/assets/619b44ae-11c9-4613-958d-04b963e42e6a)


---

##  Diagrama de Bloc

TODO: de completat ulterior cu schema bloc a componentelor

---

##  Lista de Componente (BOM)

| Componenta             | Cod Produs         | Descriere                        | Furnizor | Datasheet |
|------------------------|--------------------|----------------------------------|----------|-----------|
| **ESP32-C6-WROOM-1-N8** | U2                 | MCU Wi-Fi 6 + Bluetooth 5        | [Mouser](https://eu.mouser.com/ProductDetail/Espressif-Systems/ESP32-C6-WROOM-1-N8) | [Link](https://www.espressif.com/sites/default/files/documentation/esp32-c6-wroom-1_datasheet_en.pdf) |
| **W25Q512JVEIQ**        | U1                 | Flash SPI, 512Mb                 | [Mouser](https://eu.mouser.com/ProductDetail/Winbond/W25Q512JVEIQ) | [Link](https://www.winbond.com/resource-files/W25Q512JV%20RevD%2004082020.pdf) |
| **BME680**              | S                  | Senzor de mediu (T/H/P + GAZ)    | [Mouser](https://eu.mouser.com/ProductDetail/Bosch-Sensortec/BME680) | [Link](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bme680-ds001.pdf) |

---

##  Arhitectura Hardware

###  ESP32-C6-WROOM-1-N8

- **Rol:** Controler principal
- **Frecventa:** 160 MHz
- **Memorie:** 512KB SRAM + 8MB Flash
- **Conectivitate:** Wi-Fi 6, Bluetooth 5, USB 2.0
- **Interfete:**
  - SPI: Display, Flash, MicroSD
  - I2C: Senzori si RTC
  - UART: Debug
  - GPIO: Butoane si LED-uri

###  Display E-Paper

- **Dimensiune:** 7.5 inch
- **Rezolutie:** 800×480 px
- **Consum:** <50mA activ, ~0 in standby
- **Interfata:** SPI

### 🌡 Senzor BME680

- **Masuratori:** Temperatura, umiditate, presiune, calitate aer
- **Interfata:** I2C @ 400kHz
- **Consum:** <1mA activ, <1µA standby

###  Sistem Alimentare

- **Baterie:** Li-Po 3.7V / 2500mAh
- **Incarcare:** USB-C (5V, max. 1A) – MCP73831
- **Monitorizare:** MAX17048 (prin I2C)
- **Consum estimat:**
  - Activ (Wi-Fi + Display): ~150mA
  - Standby: ~10mA
  - Deep Sleep: <50µA
- **Autonomie:** ~250 ore (~1 saptamana)

### 🔘 Navigare

- **Butoane:** Panasonic EVQPUJ02K
- **Rezistenta pull-up:** 10KΩ

### 🔌 USB-C

- **Rol:** Incarcare si transfer de date
- **Protectie:** Diode ESD (USBLC6-2SC6Y)

---

## ⚙️ Configuratie Pini ESP32-C6

| Pin     | Functie         | Componenta         | Descriere                        |
|---------|------------------|---------------------|----------------------------------|
| GPIO0   | BOOT_BUTTON      | Buton               | Init bootloader                  |
| GPIO1   | RESET_BUTTON     | Buton               | Reset sistem                     |
| GPIO2   | CHANGE_BUTTON    | Buton               | Navigare meniuri                 |
| GPIO3   | EPD_CS           | Display             | SPI Chip Select                  |
| GPIO4   | EPD_DC           | Display             | Date / Comenzi                   |
| GPIO5   | EPD_RST          | Display             | Reset display                    |
| GPIO6   | EPD_BUSY         | Display             | Status refresh                   |
| GPIO7   | SPI_MOSI         | Flash, SD, Display  | Date SPI                         |
| GPIO8   | SPI_MISO         | Flash, SD, Display  | Date SPI (read)                  |
| GPIO9   | SPI_SCK          | Flash, SD, Display  | Ceas SPI                         |
| GPIO10  | FLASH_CS         | Flash               | SPI Chip Select                  |
| GPIO11  | SD_CS            | Card SD             | SPI Chip Select                  |
| GPIO12  | I2C_SDA          | Senzori, RTC        | Date I2C                         |
| GPIO13  | I2C_SCL          | Senzori, RTC        | Ceas I2C                         |
| GPIO14  | CHG_LED          | LED                 | Indicator incarcare              |
| GPIO15  | UART_TX          | Debug               | Serial TX                        |
| GPIO16  | UART_RX          | Debug               | Serial RX                        |
| GPIO17  | USB_D+           | USB-C               | Linie USB pozitiva               |
| GPIO18  | USB_D-           | USB-C               | Linie USB negativa               |

---

## Structura Proiectului

```bash
openbook/
├── firmware/         # Cod sursa pentru ESP32 (C / C++)
│   ├── main/
│   ├── components/
│   └── sdkconfig
├── hardware/         # Scheme electronice, PCB, KiCAD
├── docs/             # Documentatie tehnica
└── README.md         # Acest fisier
📌 Observatii Finale
```
## Detalii
Pinii si componentele au fost alese pentru a optimiza consumul, costul si usurinta integrarii. Proiectul este potrivit pentru extensii ulterioare: adaugarea de touch, conectivitate BLE avansata, suport pentru formate multiple de fisiere e-book etc.

