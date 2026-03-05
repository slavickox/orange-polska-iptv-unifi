# Orange TV (Polska) na UniFiOS

Ten skrypty umożliwiają skonfigurowanie usługi Orange TV w Polsce dla klientów, którzy zastąpili swojego Funboxa bramą UniFi opartą na UniFiOS (Cloud Gateways (UCG-XX), Gateways (UXG-XX), Dream Routers (UDM-XX) itd.).

Ponieważ obecnie nie ma możliwości skonfigurowania w GUI UniFi interfejsu WAN z wieloma VLAN-ami, skrypt tworzy nowy interfejs oraz uruchamia proxy IGMP w celu przekazywania strumieni IPTV

Skrypty bazują na pracy Fabian Mastenbroek: https://github.com/fabianishere/udm-iptv

Testowane na:

UniFi Gateway Fiber v5.0.12

UniFi Network v10.1.85

dekoder Orange 4K

# Wymagania

1. Działające połączenie internetowe Orange

Uwaga: w kontrolerze, w ustawieniach WAN, opcja IGMP Proxy musi być odznaczona (aby uniknąć konfliktów przy uruchamianiu proxy IGMP przez skrypt).

2. (Zalecane, ale nieobowiązkowe) Dedykowany VLAN dla TV w sieci lokalnej

Pozwala to zastosować specjalną konfigurację wymaganą do działania TV wyłącznie dla dekodera (DNS itd.).

3. Konfiguracja w GUI

W sieci, do której podłączony jest dekoder TV, ustaw następujące wartości:

DHCP: włączone

DNS Server: ustaw wyłącznie serwery DNS Orange: `194.204.152.34` i `194.204.159.1`

Exemple :

![Screenshot GUI](https://raw.githubusercontent.com/jbbodart/orange-iptv/refs/heads/main/img/Configuration%20r%C3%A9seau.png)
 
# Instalacja

Wymagany jest dostęp SSH do bramy.
Na bramie pobierz repozytorium do katalogu /data/orange-iptv
(zawartość katalogu /data jest zachowywana po restartach i aktualizacjach firmware):

```bash
cd /data
curl -sL https://github.com/jbbodart/orange-iptv/archive/refs/tags/v0.2.tar.gz | tar -xvz
mv orange-iptv-0.2 orange-iptv
```

Otwórz skrypt orange-iptvd i skonfiguruj wartości zgodnie ze swoją instalacją, w szczególności:

```
# Interface on which IPTV traffic enters the router (configure according to your Unifi device)
IPTV_WAN_INTERFACE="eth4"
# LAN interfaces on which IPTV should be made available (configure according to your home network)
IPTV_LAN_INTERFACES="br102"
```

IPTV_WAN_INTERFACE - interfejs WAN bramy (np. eth4 w Gateway Max)

IPTV_LAN_INTERFACES - lista interfejsów sieci lokalnej, do których ma być rozgłaszany strumień multicast (np. br102 = VLAN 102)

Po skonfigurowaniu skryptu zainstaluj go:

```bash
cd /data/orange-iptv
./orange-iptv install
```

# Weryfikacja

Po uruchomieniu skryptu powinien zostać utworzony nowy interfejs iptv z odpowiednimi parametrami QoS:

```
root@UXGMax:/data/orange-iptv# ip -d link show iptv
XX: iptv@eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 9c:05:d6:d7:b6:eb brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 0 maxmtu 65535 
    vlan protocol 802.1Q id 840 <REORDER_HDR> 
      egress-qos-map { 0:4 1:4 2:4 3:4 4:4 5:4 6:4 7:4 } addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 
```

Dodatkowo powinien być uruchomiony proxy IGMP (IMProxy):

```
root@UXGMax:/data/orange-iptv# systemctl status orange-iptvd
● orange-iptvd.service - Orange IPTV support for UniFi
     Loaded: loaded (/lib/systemd/system/orange-iptvd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-09-22 09:10:12 CEST; 55min ago
   Main PID: 7346 (improxy)
      Tasks: 1 (limit: 2328)
     Memory: 1.3M
        CPU: 523ms
     CGroup: /system.slice/orange-iptvd.service
             └─7346 improxy -c /var/run/improxy.orange-iptv.conf -p /var/run/improxy.orange-iptv.pid

Sep 22 09:10:12 UXGMax systemd[1]: Started Orange IPTV support for UniFi.
Sep 22 09:10:12 UXGMax orange-iptvd[7346]: Setting up IMProxy
Sep 22 09:10:12 UXGMax orange-iptvd[7346]: Starting IMProxy
```

Jeśli usługa jest aktywna i działa, TV powinna być teraz dostępna na dekoderze.

PS. Warto odłączyć dekoder z prądu i go spowrotem podłączyć dla totalnego restartu usługi lub zrobić to z ustawień dekodera(opcja 1 dla starych dekoderów opcja 2 dla nowszych z androidem)

# Aktualizacja firmware

Po aktualizacji firmware skrypt nie zostanie zainstalowany automatycznie.

Należy ponownie wykonać instalację:

```bash
cd /data/orange-iptv
./orange-iptv install
```

# Odinstalowanie

Aby całkowicie usunąć skrypt:

```bash
cd /data/orange-iptv
./orange-iptv uninstall
```

Następnie usuń katalog:

`/data/orange-iptv`
