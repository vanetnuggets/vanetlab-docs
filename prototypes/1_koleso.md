# Prototyp 1 - Koleso

## Špecifikácia funkcionality

Prototyp bude vedieť zostaviť a simulovať topológiou typu [second.cc](https://github.com/Gabrielcarvfer/NS3/blob/master/examples/tutorial/second.cc) a [second.py](https://github.com/Gabrielcarvfer/NS3/blob/master/examples/tutorial/second.py). Z topologického hladiska to sú uzly typu CSMA a Point-to-point. Komunikácia medzi uzlami bude zabezpečená UDP echo klientom a serverom. Prototyp bude vedieť zostaviť topológiu s ľubovoľným počtom uzlov, klientov a serverov, ktorá sa bude dať odsimulovať.

## Vybrané technológie
Front-end je implementovaný v jazyku svelte, oproti iným webovým frameworkom je rýchlejší,  nakoľko je kompilovaný a po builde neobsahuje zbytčnosti. Back-end je implementovaný v jazyku python s frameworkom Flask.

# Implementácia a dátové štruktúry

## Front-end
V NS3 simuláciach sa konfigurácie inštalujú buď na uzly, alebo na kontajnery, ktoré obsahujú viacero uzlov s rovnakou konfiguráciou. Z tohoto dôvodu bude implementácia uzlov zjednotená do kontajnerov. Pre prehľadnosť a reaktivitu sú všetky vlastnosti konfigurácie uložené v globálnom svelte store.

Rozhranie front-endu je rozdelené na 5 častí - ľavý, pravý drawer, v ktorom sú HTML formy na konfiguráciu topológie, horný toolbar, v ktorom sa dá prepínať medzi hlavnými scénami a dolný toolbar, v ktorom je tlačidlo na komunikáciu s backendom. Hlavná scéna má 2 režimy - jeden na interaktívne vytváranie topológie a druhý na zobrazenie výsledkov simulácie. Prepínať medzi režimi sa dá čez tlačidlá v hornom toolbare.

![](/media/1/fe_1.png)

### Kontajner a jeho atribúty
 - Unikátny názov kontajneru
 - Typ kontajneru (CSMA/P2P)
 - Atribúty kontajneru - zatial spoločné pre obydva typy kontajneru
     - Delay - konfiguračná hodnota - oneskorenie odoslania správy
     - DataRate - konfiguračná hodnota - priepustnosť spojenia
 - Adresa siete - IP adresa siete, z ktorej rozsahu sa budú nastavovať adresy jednotlivým uzlom v kontajneri
 - Maska siete - Maska siete vyžšie špecifikovanej adresy
 - Názov siete - bude odstránené, nepotrebné.

Do kontajnerov sa budú môcť vkladať uzly. Každý uzol môže byť v ľubovoľnom počte kontajnerov, to že je uzol v kontajneri si môžeme predstaviť ako sieťové interface uzlu.


### Aplikácie

1. UDP echo server

    Ide o službu, ktorá počúva na porte sieťového rozhrania a odpovedá na UDP echo požiadavky. Na jeho vytvorenie potrebuje atrbúty:
    - Názov serveru
    - Čas spustenia
    - Čas vypnutia
    - Port, na ktorom bude počúvať
    - Kontajner, z ktorého uzlov sa bude vyberať uzol na nainštalovanie
    - ID uzla na ktorý bude server nainštalovaný (jeden zo zvoleného kontajneru)


2. UDP echo klient

    Ide o službu, ktorá bude posielať požiadavky na zvolený server a bude počúvať odpovede. Má atribúty:
    - server, na ktorý bude posielať poŽiadavky
    - čas spustenia
    - čas vypnutia
    - kontajner, v ktorom bude klient nainštalovaný
    - ID uzla v rámci kontajnera, na ktorý bude klient nainštalovaný
    - konfiguračné atribúty:
        - interval odosielania
        - maxialny počet odoslaných paketov
        - veľkosť individuálnych paketov


Konfigurácia je implementovaná cez HTML formuláry a pridávanie a odoberanie uzlov cez grafické drag-and-drop interaktívne rozhranie. Spojenia medzi uzlami v rámci kontajneru budú dynamicky graficky zobrazené.

Aplikácia vytvorí veľký JSON obsahujúci celú konfiguráciu scenára, ktorá bude sparsovaná a odsimulovaná na back-ende. Výstup sa bude vedieť vizualizovať jednoduchou formou zobrazenia správ vygenerovaných pri simulácii a možnosťou stiahnuť si vygenerované .pcap súbory. Príklad vygenerovaného JSONu:
```json
{
  "topology": {
    "node_count": 6,
    "node_containers": [
      "csma_nodes",
      "p2p_nodes"
    ],
    "container_settings": {
      "p2p_nodes": {
        "id": 1,
        "name": "p2p_nodes",
        "type": "point_to_point",
        "data_rate": {
          "value": 100,
          "format": "Mbps"
        },
        "delay": {
          "value": 1,
          "format": "ns"
        },
        "network_name": "p2p_interfaces",
        "network_address": "10.2.2.0",
        "network_mask": "255.255.255.0",
        "log_pcap": true,
        "nodes": [
          7, 9
        ]
      },
      "csma_nodes": {
        "id": 0,
        "name": "csma_nodes_interfaces",
        "type": "csma",
        "data_rate": {
          "value": 100,
          "format": "Mbps"
        },
        "delay": {
          "value": 1,
          "format": "ns"
        },
        "network_name": "csma_interfaces",
        "network_address": "10.1.1.0",
        "network_mask": "255.255.255.0",
        "log_pcap": true,
        "nodes": [
          2, 3, 4, 7
        ]
      }
    }
  },
  "simulation": {
    "server": {
      "echo_server": {
        "id": 0,
        "name": "echo_server",
        "port": 9,
        "start": {
          "value": 1,
          "format": "s"
        },
        "stop": {
          "value": 10,
          "format": "s"
        },
        "network": "csma_nodes",
        "node": 7
      }
    },
    "client": {
      "echo_client": {
        "id": 0,
        "name": "echo_client",
        "port": 9,
        "start": {
          "value": 1,
          "format": "s"
        },
        "stop": {
          "value": 10,
          "format": "s"
        },
        "network": "csma_nodes",
        "node": 2,
        "server": "echo_server",
        "interval": {
          "value": 1,
          "format": "s"
        },
        "max_packets": 10,
        "packet_size": 128
      }
    }
  }
}
```

## Implementácia back-endu

Backend slúži na parsovanie JSONu topológie a jej transformáciu na simulačný scenár. Poskytuje dva API endpointy.

`POST /tracejson` - V tele požiadavky dostane JSON scenáru v ygenerovaný front-end  časťou aplikácie, ktorý sa pretransformuje na simulačný scenár, spustí v NS-3 simulátore a vráti výstup simulácie v JSON objekte. Výstupný objekt obsahuje dva atribúty:
   - `logs: list<{string: name, int: size}>` - zoznam vygenerovaných pcap súborov - dvojica názov súboru a jeho veľkosť
   - `output: list<string>` - zoznam výstupu scenáru. Správy typu uzol *x* poslal *y* bajtov na IP adresu *z*.

`GET /trace?name=XYZ` - Ako argument name dostane meno `.pcap` súboru, získané cez /tracejson požiadavku. Vráti `.pcap` súbor s rovnakým meno.

Po prijatí JSON-u sa zavolá parser. Ten je zložený z viacerých modulov. Pre prehľadnosť každý modul parsuje logický celok scenáru. Aktuálne aplikácia podporuje  parsovanie nasledujúcich celkov cez moduly:
- `nodes` - rozdelenie uzlov do kontajnerov
- `p2p` - konfigurácia point-to-point kontajnerov
- `csma` - konfigurácia csma kontajnerov
- `echoudp` - echo udp klient/server-y
- `ipv4` - ipv4 komunikácia a adresy
- `log` - logovanie - aké ns3 udalosti sa majú logovať, zatial nastavené staticky. 
- `pcap` - vytváranie pcap súborov pre komunikáciu v rámci kontajnerov
 
Vytiahnuté údaje z JSON objektu sú presmerované do virtuálneho modelu jednotlivých prvkov scenára. Momentálne je model implementovaný len pre zložitejšie prvky scenáru - udp klient a server. Virtuálny model má okrem konštruktoru implementované funkcie
 - `dumppy` - vráti python zdrojový kód simulačného scenáru modelu
 - `dumpcc` - zatiaľ neimplementované - to isté ale pre jazyk c++.

Na formátovanie elementov ako čas a formát atribútov uzlov slúžia takzvaní helperi, máme implementovaný jeden, `format_helper`, ktorý parsuje údaje na formát, ktorý NS-3 potrebuje.

Parser zavolá všetky moduly a vytvorí zdrojový kód scenáru, ktorý sa uloží na disku a  vykoná v ns-3 simulátori. 

API endpoint má 2 pomocné manažéry
 - File manažér, ktorý má na starosti prácu so súborovým systémom - ukladať vytvorené scenáre na disku, spustenie scenárov, kopírovanie výstupných súborov scenáru do priečinku dostupného cez API. 
 - NS-3 manažér, ktorý má na starosti operácie s NS-3 simulátorom - vykonanie scenáru ak pozná názvy súboru, získanie vygenerovaných logov a podobne.

V tomto prototype backend posytuje aj jednoduchú statickú web-stránku pod endpointom `GET /` , ktorá slúži na testovanie. V neskorších verziach programu bude odstránená, nakoľko nebude potrebná.

![](/media/1/be_if.png)