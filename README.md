# Uvod 

Tento repozitar obsahuje MCU firmware knihovnu pro obsluhu komunikacnich chipsetu [BlueNRG-MS](www.st.com/bluenrg-ms) (Bluetooth Low Energy v4.1) a [CC3100](http://www.ti.com/product/CC3100) (802.11 bgn). Knihovna je urcena predevsim pro platformu MCU STM32, nicmene byla pouzita i na platforme CC3200.

# Struktura repozitare

Repo je strukturovano do nasledujicich slozek:

- Atollic_workspace - jednotlive ukazkove projekty pro TrueStudio IDE.
- Board_package - knihovna Board support package pro vsechny vyvinute platformy.
- Docs - dokumentace.
- Drivers - nizko-urovnove drivery pro obsluhu MCU a jeho periferii.
- Middleware - bloky softwaroveho middleware, obsahuje zakladni stavebni bloky SW. 
- STM32CubeMx - projektove soubory inicializacniho SW STM32CubeMx obsahujici konfiguraci MCU a pinout.
- Utils - dalsi nastroje pro podporu techto ukazkovych projektu ze SW tretich stran. [Popis zde.](security.md)

# Dokumentace

Dokumentace je strukturovana takto:

- Docs/middleware_documentation.docx - Uzivatelsku popis middleware knihoven ble a wlan.
- docs/ble_profiles - Uzivatelsky popis podporovanych Ble sluzeb vcetne struktury jejich charakteristik, UUID atd.
- Docs/ble\_mw\_refmanual.chm - referencni manual Ble middleware knihovny.
- Docs/wlan\_mw_refmanual.chm - referencni manual Wlan middleware knihovny.
- Tyto Github stranky.

# Podporovane platformy

Knihovna byla tvorena predevsim pro rodinu procesoru STM32 od ST, ale jiz v minulosti byla overena jeji pouzitelnost v procesoru CC3200 a prostredi TI-RTOS. V ramci vyvoje byly vytvoreny 2 prototypy pro demonstraci pouze BlueNRG s procesorem STM32L052 a dva prototypy pro demonstraci obou chipsetu v aplikaci wifi provisioningu na platforme STM32F411.

- StmBle - STM32L0 s BlueNRG-MS chipsetem ve vlastnim provedeni. Obsahuje LIS3DH akcelerometr a HDC1000 sensor teploty.
- BlueModule - vlastni kompaktni modul s BlueNRG-MS chipsetem. Pripraven pro integraci do dalsich zarizeni. Hned v zapeti po vytvoreni tohoto moulu predstavilo ST vlastni modul, ktery je jiz predcertifikovany (SPBTLE)
- BlueCarrier - STM32L0 s vlastnim modulem BlueModule.
- WifiBle RevA - STM32F411 s chipsety BlueNRG-MS a CC3100 ve vlastnim provedeni. Obsahuje LIS3DH akcelerometr a HDC1050 sensor teploty.
- WifiBle RevB - STM32F411 s moduly BlueModule nebo SPBTLE a CC3100MOD.

# Ukazkove aplikace
 
Slozka Atollic_workspace obsahuje pripravene Atollic projekty pro nekolik nasledujicich ukazkovych aplikaci. Kazda aplikace obsahuje slozku Application s aplikacne zavislymi moduly (main, irq, usb, atd.), vsechny ostani komponenty spolecne pro vice projektu (Middleware, stack, BSP) jsou do projektu vlozeny jako Linked Folder. 

- [StmBle_server](stm_ble.md) - Aplikace prototypu StmBle v roli BLE GATT serveru, ktery implementuje sluzby DoorLock a Temperature. Tento projekt je pouzit pro demonstraci DoorLock odemykani dveri pomoci mobilniho telefonu s BLE, pripadne pro vzdaleny odecet teploty do IoT hubu (zatim pouze na papire).
- [BlueCarrier](blue_carrier.md) - Aplikace prototypu BlueCarrier v roli BLE GATT serveru, ktery implementuje sluzby DoorLock a Benchmark. Tento projekt taktez muze slouzit pro demonstraci odemykani dveri pomoci mobilniho telefonu s BLE.
- [Wifi_ble](wifi_ble.md) - Aplikace prototypu WifiBle RevA i RevB v aplikaci wifi provisioningu bez pouziti RTOSu.
- [Wifi\_ble_rtos](wifi_ble_rtos.md) - Aplikace prototypu WifiBle RevA i RevB v aplikaci wifi provisioningu s pouzitim RTOSu.

# Drivery

Knihovna je postavena na nizko-urovnovych driverech tretich stran CMSIS a STM32xx\_HAL\_driver, oboji v aktualni verzi z STM32CubeMX (1.9 pro STM32F4 a 1.3 pro STM32L0).

# Middleware

Knihovna obsahuje jednak middleware tretich stran, coz jsou predevsim komunikacni stacky vyrobcu chipsetu a pote nas vlastni middleware jenz zjednodusuje pouziti techto stacku v aplikacich.

- FreeRTOSv8\_1_2 - FreeRtos portovany pro STM32, generovany pomoci STM32CubeMX. Vsechny knihovny jsou upraveny pro podporu CMSIS port RTOSu.
- Le\_ble\_library - vlastni knihovna pro obsluhu BlueNRG-MS. Vice informaci v ./Docs/.
- Le\_wlan\_library - vlastni knihovna pro obsluhu CC3100. Vice informaci v ./Docs/. 
- SimpleBlueNRG_HCI - Nizko-urovnovy stack pro komunikaci s chipsetem BlueNRG-MS poskytnuty ST v aktualni verzi 1.8.0.
- Simplelink - Nizko-urovnovy stack pro komunikaci s chipsetem CC3100 poskytnuty TI.
- STM32\_USB\_Device_Library - USB device knihovna pro tridu Communications device class (CDC). 

# STM32CubeMX

Tato slozka obsahuje projekty pro inicializacni software STM32CubeMX od ST, ktery omoznuje rychle zalozeni projektu a pinoutu pro procesory STM32.


# Utils

Tato slozka obsahuje nekolik podpurnych nastroju/scriptu pro praci se softwarem tretich stran, predevsim pro konfiguraci seriove flash pameti chipu CC3100, spravu jeji file systemu a bezpecnostich certifikatu.


