# Zabezpeceni wi-fi komunikace pomoci certifikatu

V kontextu wi-fi komunikace s chipsetem CC3100 bereme v uvahu dve mozna zabezpeceni komunikace, ktera vyzaduji nejaky bezpecnostni certifikat.

- Pripojeni do zabezpecene site WPA Enterprise (napr. univerzitni site).
- Vytvoreni zabezpeceneho TLS (Transport layer security, nastupce SSL) spojeni proti se serverem.


# WPA Enterprise

WPA Enterprise je zebezpeceni site podobne jako bezne WPA2, ale nabizi pokrocile nastroje pro spravu, oboustrane overeni identity atd. Tyto site  potrebuji nektere z nasledujicich souboru pritomne v seriove flash pameti. WPA Ent site vetsinou vyzaduji overeni identity pomoci korenoveho certifikatu certifikacni autority (rootCA), nektere vsak mohou vyzadovat i overeni uzivatele pomoci RSA klice. Toto zalezi na provozovateli Wi-fi site. Simplelink knihovna jasne definuje jak se tyto soubory musi jmenovat, tudiz v jeden okamzik muze flash obsahovat prihlasovaci udaje pouze do jedne site. Tyto certifikaty museji byt ulozeny v base-64 formatu .pem.

- /cert/ca.pem - korenovy CA, poskytnuty provozovatelm wi-fi, na PC vetsinou musi byt v seznamu duveryhodnych korenovych certifikatu. Shodny pro vsechny uzivatele. Slouzi k overeni identity wi-fi site v CC3100.
- /cert/client.pem - verejny klic koncoveho zarizeni.
- /cert/private.key - privatni klic koncoveho zarizeni.

Vsechny tyto klice museji byt vytvoreny dle pozadavku wi-fi site. Slozka Utils neobsahuje zadne nastroje pro jejich tvorbu. Tyto klice jsou uzivatelsky zavisle, kazdy uzivatel bude potrebovat jine klice. Na druhou stranu WPA Enterprise site nejsou prilis obvykle (krome univerzit) vzhledem ke slozitejsi sprave.

# TLS

Transport layer security slouzi k vytvoreni zabezpeceneho kanalu do serveru. Parametry TLS tudiz zavisi na nastaveni serveru a tim padem aplikaci a mohou tudiz byt konstantni po celou zivotnost produktu. TLS je aktualne state-of-the-art sifrovani, ktere je schopno overit identitu obou stran, prenest sifrovaci klice po nezabezpecenem kanale, pote sifrovat a kontrolovat neporusenost dat. Nejvyssi dosazena sila sifrovani dosazena pri SSL handshaku proti .NET v4.5 frameworku je SL_SEC_MASK_TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA:

- Ephemeral Elipltic curve D-H podepsany RSA key exchange
- RSA overeni identity
- 256 bitove symetricke sifrovani
- 160 bitovy secure hash SHA1

CC3100 potrebuje pro TLS sifrovani tri soubory nahrane v seriove flash v binarinim .der formatu. Nazvy souboru jsou uzivatelske, cili je mozne do flash nahrat vice sad klicu. Nize jsou tyto soubory uvedeny s nami vybranymi nazvy.

- /cert/serverCa.der - korenovy certifikat serveru, slouzi pro overeni identity serveru. Pokud server pouziva self-signed certifikat, pote je tento i korenovym certifikatem rootCa (takto je to implementovane v ukazkove aplikaci). 
- /cert/clientPrivate.der - privatni RSA 20148 bit klic daneho zarizeni. Kazde zarizeni ma vlastni klic, pote muze slouzit k identifikaci a autorizaci zarizeni.
- /cert/clientCert.der - verejny klic prislusejici privatnimu klici podepsany duveryhodnou autoritou serveru, napriklad jeho vlastnim certifikatem (pokud je self-signed).


## Util/VS\_TLS\_Example

Tato slozka obsahuje Visual Studio projekt s jednoduchou konzolou aplikaci TLS serveru a TLS klienta. TLS serverova aplikace slouzi jako protistrana pro vytvoreni TLS socketu a zakladni benchmark prenosove rychlosti. Teto aplikaci musi byt predan serverovy certifikat ve formatu .pfx. Autorita, ktera podepsala certifikaty koncovym zarizenim, musi byt v seznamu duveryhodnych autorit OS.


## Util/Certificates\_openssl

Zde jsou k dispozici skripty pro vytvoreni certifikatu a klicu pomoci nastroje openssl. Tyto skripty ocekavaji openssl v defaultni slozce c:\OpenSSL-Win32\. Prvnim krokem pri tvorbe sady certifikatu je vytvoreni self-signed certifikatu pro server.

```
createSelfsignedCertificate testLePc2 12345678
```

V prubehu tvorby certifikatu je treba vyplnit nektere nacionale, predevsim `Common name`, ktery by se mel shodovat s nazvem certifikatu (v nasem pripade testLePc2, poznamka pro vysvetleni, LePc2 je nazev lokalniho serveru hostujici socket server aplikaci). Z vytvorenych souboru pouzijeme testLePc2.pfx jako certifikat (tento format obsahuje i privatni klic) pro .NET TLS\_Server a testLePc2CA.der jako cetifikacni autoritu pro CC3100 zarizeni (/cert/serverCa.der).

V druhem kroce vytvorime certifikat pro CC3100 zarizeni podepsany certifikacni autoritou (tedy testLePc2, ale v principu to muze byt libovolna duveryhodna autorita, od ktere vlastnime privatni klic).  

```
createDeviceCertificate testDev testLePc2
```

Vytvareni noveho certifikatu opet provazi vyplnovani informaci. Challenge password se vyplnit nemusi. Po skonceni mame testDev.der jako klientsky certifikat (/cert/clientCert.der) a testDevPrivate.der k nemu prislusejici privatni klic (/cert/clientPrivate.der).

Nyni tyto soubory bud pomoci Uniflash nebo pomoci wlan\_fs modulu nahrajeme do seriove flash pameti. V pripade zachovani danych nazvu souboru ve file systemu (vse /cert/) neni treba v wlan middleware knihovne cokoliv menit.

Pro prevedeni binarnich certifikatu do hlavickoveho souboru pro wlan\_fe modul muzeme pouzit skript

```
generateHeaderFromCertsTest
```

Vygenerovany soubor wlan\_cert.h je treba zkopirovat do /Middleware/Le\_wlan\_library slozky a zkompilovat projekt. V programu pote lze naprogramovat certifikat do flash

```
ret = WlanFs_InitializeCertificates();
```

Pro overeni identity CC3100 zarizeni na serverove strane musi byt certifikacni autorita  nainstalovana a pridana do uloziste duveryhodnych certifikatu. V nasem pripade, kdy jsme podepsali serverovym certifikatem, je treba nainstalovat prave tento testLePc2.pfx certifikat.  Ve windows to lze udelat interaktivne poklepanim na .pfx soubor.

Aplikace TLS\_Server bere jako parametr prave .pfx certifikat a pripadne heslo, kterym je tento certifikat sifrovany. Skripty defaultne pouzivaji heslo 12345678, nicmene muze byt zadane jine. V okamziku kdy aplikace TLS\_Server bezi, je mozne se k serveru pripojit a spustit benchmark, z programu napriklad

```
WlanTls_Benchmark(SL_IPV4_VAL(192,168,1,163), 4443, nbPackets, 1340);
```

jez provede benchmark na IP adresu 192.168.1.163, port 4443, kde bezi TLS\_Server, delka packetu je 1340 bytu.