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


## BOM:

| **Piece Name**                          | **Piece Type (Optional)**            | **Link** |
|----------------------------------------|--------------------------------------|----------|
| ESP32-CAP C0402                        | C                                    | https://componentsearchengine.com/part-view/CC0402MRX5R5BB106/YAGEO |
| ADAFRUIT_CHIP-LED0603                  | LED                                  | https://www.snapeda.com/parts/KP-1608SURCK/Kingbright/view-part/?ref=search&t=LED%200603 |
| 112ATAARR03                            | microSD                              | https://www.snapeda.com/parts/112A-TAAR-R03/Attend/view-part/ |
| 744043680                              | L                                    | https://eu.mouser.com/ProductDetail/Wurth-Elektronik/744043680?qs=PGXP4M47uW6VkZq%252BkzjrHA%3D%3D |
| BD5229G-TR                             | Voltage Detector                     | https://www.snapeda.com/parts/BD5229G-TR/Rohm/view-part/?ref=search&t=BD5229G-TR |
| BUTTON_CUSYOMV1                        | Button                               | https://www.snapeda.com/search/?q=EVQP7L01P&search-type=parts |
| CPH3225A                               | C                                    | https://www.snapeda.com/parts/CPH3225A/Seiko/view-part/ |
| DS3231SN#                              | I²C-Integrated RTC/TCXO/Crystal      | https://www.snapeda.com/parts/DS3231SN%23/Analog%20Devices/view-part/?ref=search&t=DS3231SN%23 |
| ESP32-C6-WROOM-1-N8                    | ESP32                              | https://www.snapeda.com/parts/ESP32-C6-WROOM-1-N8/Espressif%20Systems/view-part/?ref=search&t=ESP32-C6-WROOM-1-N8 |
| ESP VARSISTOR                          | Varsistor (B72520T0350K062)                      | https://ro.mouser.com/ProductDetail/EPCOS-TDK/B72520T0350K062?qs=dEfas%2FXlABIszF52uu7vrg%3D%3D |
| ESP32_WROVER_AVX---SD0805S020S1R0      |  DIODE SCHOTTKY                       | https://componentsearchengine.com/part-view/SD0805S020S1R0/Kyocera%20AVX |
| ESP32_WROVER_BME680_BME680             | Env Senzor                            | https://www.snapeda.com/parts/BME680/Bosch%20Sensortec/view-part/?ref=search&t=bme680 |
| ESP32_WROVER_EAGLE-LTSPICE_R           | R                                    | https://componentsearchengine.com/part-view/R0402%201%25%20100%20K%20(RC0402FR-07100KL)/YAGEO |
| ESP32_WROVER_SPARKFUN-DISCRETESEMI_MOSFET_PCH | T DMG2305UX-7                 | https://componentsearchengine.com/part-view/DMG2305UX-7/Diodes%20Incorporated |
| ESP32_WROVER_SPARKFUN-IC-POWER_MCP73831 | TINY INTEGRATED LI-ION/LI-POLY CHARGE MGNT CONTROLLER | https://componentsearchengine.com/part-view/MCP73831T-2ACI%2FOT/Microchip |
| FH34SRJ-24S-0.5SH_99_                   | FH34SRJ-24S-0.5SH(99)               | https://componentsearchengine.com/part-view/FH34SRJ-24S-0.5SH(99)/Hirose |
| MAX17048G+T10                          |  Cell Fuel Gauge with ModelGauge     | https://www.snapeda.com/parts/MAX17048G+T10/Analog%20Devices/view-part/ |
| MBR0530                                |  Diode Schottky                      | https://www.snapeda.com/parts/MBR0530/Onsemi/view-part/ |
| PGB1010603MR                           |  Ipp Tvs Diode Surface Mount 0603    | https://www.snapeda.com/parts/PGB1010603MR/Littelfuse/view-part/ |
| QWIIC_CONNECTOR                        | PRT-14417 QWIIC_CONNECTOR                 | https://www.snapeda.com/parts/PRT-14417/SparkFun/view-part/ |
| RCL_CPOL-EU                            | C pol                                | https://grabcad.com/library/tantalum-smd-capacitor-type-b-3528-1 |
| SAMACSYS_PARTS_USB4110-GF-A            | USB4110-GF-A                         | https://www.snapeda.com/parts/USB4110-GF-A./Global%20Connector%20Technology/view-part/ |
| SJ                                     | Jumper-SolderPasteJumper3way         | https://grabcad.com/library/solder-jumpers-1 |
| TP                                     | Test-Pad                             | self-made |
| SI1308EDL-T1-GE3                       |  MOSFET Transistor                 | https://www.snapeda.com/parts/SI1308EDL-T1-GE3/Vishay/view-part/ |
| USBLC6-2SC6Y                           |  Ipp Tvs Diode Surface Mount                                    | https://www.snapeda.com/parts/USBLC6-2SC6Y/STMicroelectronics/view-part/?ref=dk&t=USBLC6-2SC6Y&con_ref=None |
| W25Q512JVEIQ                           | FLASH - NOR Memory                                 | https://www.snapeda.com/parts/W25Q512JVEIQ/Winbond%20Electronics/view-part/?ref=search&t=W25Q512JVEIQ |
| XC6220A331MR-G                         | Voltage Regulator              | https://ro.mouser.com/ProductDetail/Torex-Semiconductor/XC6220A331MR-G?qs=AsjdqWjXhJ8ZSWznL1J0gg%3D%3D&utm_source=octopart&utm_medium=aggregator&utm_campaign=865-XC6220A331MR-

