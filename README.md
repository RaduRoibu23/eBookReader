#  OpenBook E-Reader
---

## Prezentare Generala

**OpenBook** este un e-reader open-source, proiectat in jurul microcontrollerului **ESP32-C6**, gandit pentru eficienta energetica, modularitate si conectivitate moderna. Dispozitivul utilizeaza un ecran e-paper de **7.5”**, asa cum a fost ceurt in enunt.

Proiectul a fost realizat in conformitate cu cerintele specifice [proiectului TSC 2025](https://ocw.cs.pub.ro/courses/tsc/proiect2025), punand accent pe documentare clara, organizare modulara si integrarea eficienta a componentelor hardware si software.

Am investit aproximativ **35–40 de ore de lucru** in dezvoltarea acestuia pentru a-l aduce in forma actuala, timp in care am proiectat schema electronica si PCB-ul . Din pacate, din cauza unor probleme tehnice peste care nu am reusit sa trec in timp util, **nu am reusit sa finalizez partea de modelare 3D** a carcasei si ansamblului complet.

In ciuda acestui neajuns, consider ca proiectul reflecta un efort solid si o abordare completa in ceea ce priveste realizarea.


![Poza Bottom](https://github.com/user-attachments/assets/567f6415-72f1-43f7-a293-d9e6d18dc0e8)
![Poza Top](https://github.com/user-attachments/assets/7dabf822-8540-4531-a2b5-caf79dd70128)
![Routing ](https://github.com/user-attachments/assets/619b44ae-11c9-4613-958d-04b963e42e6a)


---

##  Diagrama de Bloc
```

                +---------------------------+
                |     USB-C Connector       |
                | (USB Data ↔ ESP32 USB)    |
                | (ESD & Termination Prot.) |
                +-------------+-------------+
                              |
                              v
                +--------------------------+
                |       Charging IC        |<-- USB-C Power
                +-------------+------------+
                              |
                              v
                +--------------------------+
                |         Battery          |
                +-------------+------------+
                              |
                              v
                +--------------------------+
                |           LDO            |
                |       (3.3V Reg)         |
                +-------------+------------+
                              |
                              v
                      +--------------+
                      |    ESP32-C6   |
                      +------+--------+
                             |
      +----------------------+-------------------------+
      |                      |                         |
      v                      v                         v
+-----------+       +----------------+         +-----------------+
|  Display  |<----->|     BME688     |<------->| Tactile Buttons |
| (SPI 4W)  |       |   (I2C + PU)   |         |  (GPIO + RC)     |
+-----------+       +----------------+         +-----------------+
```
Legend:
- PU = Pull-up resistors
- RC = Debouncing circuit


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
- 
###  Sistem Alimentare

- **Baterie:** Li-Po 3.7V / 2500mAh
- **Incarcare:** USB-C (5V, max. 1A) – MCP73831
- **Monitorizare:** MAX17048 (prin I2C)
- **Consum estimat:**
  - Activ (Wi-Fi + Display): ~150mA
  - Standby: ~10mA
  - Deep Sleep: <50µA
- **Autonomie:** ~250 ore (~1 saptamana)

### Navigare

- **Butoane:** Panasonic EVQPUJ02K
- **Rezistenta pull-up:** 10KΩ

### USB-C

- **Rol:** Alimentare si transfer de date
- **Protectie:** Diode ESD (USBLC6-2SC6Y)

---

##  Configuratie Pini ESP32-C6

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

