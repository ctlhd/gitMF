### 1. Время жизни соеденения, по дефолту 24ч
  ```bash
  /ip firewall connection tracking set tcp-established-timeout=1h 
  ```
Маршрутизатор хранит данные об установленных TCP-соединениях в таблице connection tracking не дольше 1 часа (вместо стандартных 24 часов).


### 2.Cрок аренды dhcp

```bash
/ip dhcp-server set [find name="defconf"] lease-time=1d
```
DHCP-сервер выдаёт клиентам IP-адреса сроком на 1 день. После этого аренду нужно продлить или получить новый адрес.


### 3.Отключение не используемых helper'ы (ALG или service port)

```bash
/ip firewall service-port
set ftp disabled=yes
set tftp disabled=yes
set irc dissbled=yes
set h323 disabled=yes
set sip disabled=yes
set pptp disabled=yes
set dccp disabled=yes
set sctp disabled=yes
```
Все встроенные помощники для протоколов (FTP, TFTP, IRC, H323, SIP, PPTP, DCCP, SCTP) отключены.
Это повышает безопасность и снижает ненужную нагрузку на маршрутизатор.

### 4.Запретить ICMP ответы с WAN портов в цепочке INPUT и отключить MAC Ping, в этом случае наш микротик не будет отвечать на пинг, то есть снижает вероятность сканирования, попыток подключкения и переборов намного меньше будет, но есть одно но....ксли нам понадобится как то пинговать этот роутер, не помешаетл нам самим это правило?

```bash
/ip firewall filter add chain=input action=drop protocol-icmp icmp-options=8:0 in-interface-list=WAN src-address-list="!AllowIPRemoteManagment comment="Drop IN echo request"
/tool mac-server ping set enabled-no
```
* Запросы ping (ICMP echo-request) с WAN блокируются, кроме адресов из списка AllowIPRemoteManagment.

* MAC-ping полностью отключён.

* Это защищает роутер от внешних пингов и сканирования.

### 5.Настройки беспроводной сети:

```bash
/caps-man channel {
       add band=2ghz-g/n control-channel-widht=20mhz extension-channel=disabled frequ,ncy=2412,2437,2462 name=2.4Channels reselect-interval=1d tx-power=20
       add band=5ghz-n/ac control-channel-width=20mhz extension-channel=Ce frequency=5180,5220,5260,5300,5680,5745,5785 name=5Channels reselect-interval=1d tx-power=20 skip-dfs-channels=yes
                  }
```           

Созданы профили частот для Wi-Fi:

2.4 GHz: каналы 2412, 2437, 2462; ширина 20 MHz; мощность 20 dBm.

5 GHz: каналы 5180–5300 и 5680–5785; ширина 20 MHz + расширение Ce; мощность 20 dBm; DFS-каналы пропущены.

Канал может автоматически пересматриваться раз в 1 день.

Это задаёт список допустимых каналов и мощность для точек доступа.


### 6.Настройки безопасности:

```bash
/caps-man security add authentication-types=wpa2-psk encryption=aes-ccm group-encryption=aes-ccm disable-pmkid=yes name=OfficeNetPass passphrase="$PassOffice"
```
Создан профиль безопасности:

Шифрование: WPA2-PSK, AES-CCM.

PMKID отключён.

Пароль: "$PassOffice".

Этот профиль применяется к Wi-Fi, чтобы клиенты подключались только по WPA2 с указанным паролем.


### 7.Настройки access-list'a когда у клиента плохой сигнал, и точка вырубает его что бы он мог соеденится к другой точке:

```bash
/caps-man access-list {
           add action-accept allow-signal-out-of-range=5s disabled=no interface=any mac-address=00:00:00:00:00:00 signal-range=-75..0 ssid-regexp=""
           add action=reject allow-signal-out-of-range=always disabed=no interface-any mac-address=00:00:00:00:00:00 signal-range=-120..120 ssid-regexp=""
           		}
```

Первое правило: клиенту разрешается подключение, если сигнал не хуже –75 dBm (и до 0).

Второе правило: все остальные подключения отклоняются.

Таким образом, слабые клиенты с сигналом ниже –75 dBm не допускаются в сеть.


### 8.Настройка правила потока:

```bash
/caps-man datapath add client-to-client-forwarding=yes local-forwarding=yes name=OfficeNet
```
Создан профиль OfficeNet:

local-forwarding=yes — трафик клиентов идёт напрямую через точку, не через контроллер.

client-to-client-forwarding=yes — клиенты внутри одной точки могут общаться друг с другом.


### 9.Конфигурации для применения на wifi точках:

```bash
/caps-man configuration {
       add channel=2.4Channels country=uzbekistan3 datapath=OfficeNet distance=indoors guard-interval=long max-sta-count=32 mode=ap multicast-helper=default name=OfficeNet2 rates=StandartDataRates rx-chains=0,1 securit=OfficeNetPass ssid="$DDISOffice-2.4Ghz" tx-chains=0.1
       add channel=5Channels country=uzbekistan3 datapath=OfficeNet distance=indoors guard-interval=long max-sta-count=32 mode=ap multicast-helper=default name=OfficeNet5 rates=StandartDataRates rx-chains=0.1 security=OfficeNetPass ssid="$SSIDOffice-5Ghz" tx-chains00.1
```
Созданы два профиля Wi-Fi:

OfficeNet2 (2.4 GHz)

Каналы: 2.4Channels

Страна: Uzbekistan3

SSID: $DDISOffice-2.4Ghz

До 32 клиентов, режим AP, трафик через OfficeNet

OfficeNet5 (5 GHz)

Каналы: 5Channels

Страна: Uzbekistan3

SSID: $SSIDOffice-5Ghz

До 32 клиентов, режим AP, трафик через OfficeNet

Оба профиля используют WPA2 (OfficeNetPass) и стандартные скорости передачи.


### 10.Настройка автоконфигурацию в сети:
```bash
/caps-man provisioning {
      add action=create-disabled hw-supported-modes=gn master-configuration= OfficeNet2 name-format=prefix-identity name-prefix=2Ghz
      add action=create-disabled hw-supported-modes=ac master-configuration= OfficeNet5 name-format=prefix-identity name-prefix=5Ghz
                       }
```
Для устройств с режимом 2.4 GHz (gn) создаётся точка с конфигурацией OfficeNet2, имя начинается с 2Ghz.

Для устройств с режимом 5 GHz (ac) создаётся точка с конфигурацией OfficeNet5, имя начинается с 5Ghz.

Все новые точки добавляются в статусе disabled, чтобы админ мог вручную включить их.


### 11.Включение CAPsMAN:
```bash
/caps-man manager set enabled-yes
```