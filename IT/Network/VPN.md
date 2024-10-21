Какие задачи нам нужно решить с впн: 
- доступ до роутеров других офисов
- доступ к внутреннем и заблокированным ресурсам из офисной сети 
- доступ к внутреннем и заблокированным ресурсам с windows, osx, linux и телефонов. Тестеры берут телефоны домой. Сейчас мне кажется проше это будет сделать черезн обычный протокол OpenVPN LDAP до офисов, а с них уже мощные впн маршрутизируют трафик. 
  


VLESS + reality

Nekoray очень медленный, советую установить на роутер Openwrt и поставить туда v2rayA для подключения vless на весь роутер

Вопросы 

active probing???
блокируются ли socks5 прокси? потому что через https только https мы и пустим трафик. 

VLESS имеет гораздо более широкие возможности: Например, он может работать через CDN (Content Delivery Networks) благодаря транспорту WS, gRPC и HTTPUpgrade. ????


почему если я хочу, чтобы мои запросы уходили через айпи сервера, мне нужно прописать вот такое: 
PostDown = iptables -t nat -D POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE

PostUp = iptables -A INPUT -p udp -m state --state NEW -m udp --dport 51820 -j ACCEPT && iptables -A INPUT -i wg0 -j ACCEPT


[root@jingru centos]# ip rule ls

ip route add default via 10.10.22.1 dev wg0 table germany
ip ru add from 192.168.11.0/24 lookup germany 

https://datahacker.blog/industry/technology-menu/networking/routes-and-rules/iproute-and-routing-tables


в новых условиях надо сразу быть готовыми к отказу от интеграшек.  
интеграшкой я называю удобно-простую систему с веб мордой, в которой настраивается куча разных функций.  
или хотя бы быть готовым к выходу за пределы интеграшки  
в случае pfsense можно выйти на уровень freebsd.  
но PF фаервол не очень хорош тем, что придется вписывать свои якоря в код скриптов самого pfsense. отдельную табличку nftables или свои iptables не трогая код основной системы не выйдет залепить туда.  
если знаешь freebsd достаточно хорошо , то вариант. но я считаю linux имеет больше возможностей как на уровне управления сетью, так и на уровне совместимого софта


Классические OpenVPN, Wireguard и IPSec отметаем сразу - их уже давно умеют блокировать и блокировали не раз. [Модифицированный Wireguard от проекта Amnezia под названием AmneziaWG](http://web.archive.org/web/20240831134917/https://habr.com/ru/companies/amnezia/articles/769992/) — отличная задумка, но есть одно _но_. Некоторое время назад, во время известных событий в Дагестане, РКН пытался заблокировать Telegram в некоторых регионах. Telegram-прокси для DPI выглядят совсем недетектируемо - как набор рандомных байт без каких-то сигнатур. И знаете что сделал РКН? Они просто заблокировали _все_ неопознанные протоколы, работащие поверх TCP. HTTP работает, HTTPS работает, SMTP работает, IMAP работает, а все что "неизвестное и ни на что не похожее" - нет.

Поэтому надо искать что-то, что не только скрывает трафик, но и умеет маскироваться под что-нибудь безобидное. На Хабре уже не раз упоминали про [Cloak](http://web.archive.org/web/20240831134917/https://github.com/cbeuw/Cloak), в который можно спрятать OpenVPN, замаскировав его под какой-нибудь популярный веб-сайт, такой вариант тоже поддерживается в клиенте Amnezia, но они сами пишут, что скорость работы у такой связки не очень.


Классическими VPN-протоколами, которые внешне неотличимы от обычного HTTP являются SoftEther, MS SSTP и AnyConnect/OpenConnect. В своей первой статье "[Интернет-цензура и обход блокировок: не время расслабляться](http://web.archive.org/web/20240831134917/https://habr.com/ru/articles/710980/)" я обращал внимание на то, что все они на тот момент были уязвимы к детектированию методом active probing, что позволяло их элементарно заблокировать. Однако с недавних пор в новые версии сервера OpenConnect завезли защиту от такого, и теперь им вполне можно пользоваться.

В отличие от **OpenVPN**, **IPSec** и **WG** он внешне выглядит как самое обычное HTTPS-подключение.

В отличие от **SoftEther VPN**, для него существуют клиенты под все популярные платформы: Windows, Linux, macOS, Android, iOS, да и не одни (об этом мы поговорим чуть позже). Ну и есть защита от active probing.

В сравнении с **MS SSTP** у него более производительная серверная часть под Linux, а еще, когда блокировок трафика не осущевляется, он может в дополнение к HTTPS TLS-подключению поднимать DTLS поверх UDP для еще более лучшей производительности (Softether тоже умеет добавлять UDP, но у них свой выглядящий "неизвестным" протокол, а здесь же популярный и всем известный DTLS, который, например, используется в WebRTC). Ну и есть защита от active probing.


в версии 1.2.0 добавили режим "camouflage" для защиты от active probing;  
в версии 1.2.1 добавили поддержку клиента OneConnect (об этом чуть позже);  
в версии 1.2.3 (она еще официально не зарезилизась!) поправили баг, который мешал подключению некоторых клиентов от Cisco при включенной маскировке (по факту мобильные версии работают нормально и с 1.2.2, а вот десктопная с включенной маскировкой подключаться отказывается).



Приветствую на канале ! 1. Это не агитация, я лишь показываю то, чем сам пользуюсь и значит доверяю. 2. Отличие от WG как минимум в том, что Вам не нужен сервер для организации доступа к Вашим ресурсам. +peer2peer соеденение что называется "из коробки" 3. Вы можете организовать свой сервер с netbird \ netmaker и не зависеть от сторонних решений. Спасибо за лайк\комментарий\подписку\донат и любую другую поддержку проекта! Удачного самохостинга!


A split tunnel SSL VPN is a way better option - entirely self hosted and self configurable, and only the traffic that needs to go over the VPN does so (this negates the "everything goes through the VPN device" point that Chuck makes, only specific traffic that you define will go through it)- and their are products out there for this that also have ACLs etc. - I hate the idea of this going through a 3rd party service/server to access a private network.

Nice technology. I wonder if this protocol will be abused. When a PC is behind NAT , a home router uses Port-NAT. A statefull firewall "expects" data on the inbound port. Since the TwinGate client installed on a PC behind a NAT. It is basicly a backdoor relayer (SOCK5 proxy) in your LAN environment. Why? External users can connect to other devices in your LAN. Oh well, it is a cool tech, but I hope IDS/IPS firewall can detect this kind of traffic in business environment. A employee can easily make backdoors in your network if you are not carefull. Thanks for the clip and explanation.

https://www.twingate.com/docs/how-twingate-works
I think the bit about the client and connector talking directly to one another is technically incorrect. While the relay knows which IPs and ports the client and connector use (after NAT), you cannot have them connect to each other. That is because the NAT routers will only accept packets originating from the relay for those ports. So, in order to connect client and connector, the traffic has to be routed through the relay as a proxy. And while that traffic is probably encrypted, all of this is controlled by non-open software provided by Twingate. Thus, you essentially have to trust that Twingate is a. "not evil" and b. "stays secure". Also, the ressources that are being exposed are controlled via a cloud instance ("controller") and also, who may connect to them. You essentially delegate control over what can be accessed to Twingate, putting a remote control to your network in their hands (aka "firewall piercing"). Surely, nothing to worry about, huh?

https://habr.com/ru/articles/844760/
https://github.com/Jigsaw-Code/outline-ss-server/issues/108
https://docs.amnezia.org/documentation/amnezia-wg/

https://github.com/openwrt-xiaomi/awg-openwrt/wiki/AmneziaWG-installing#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-amneziawg-%D0%B8-%D0%B4%D1%80%D1%83%D0%B3%D0%B8%D1%85-%D0%BD%D1%83%D0%B6%D0%BD%D1%8B%D1%85-%D1%83%D1%82%D0%B8%D0%BB%D0%B8%D1%82-%D0%BD%D0%B0-vds-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B5
https://github.com/openwrt-xiaomi/awg-openwrt/wiki/AmneziaWG-installing#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-amneziawg-%D0%BD%D0%B0-openwrt-%D1%83%D1%81%D1%82%D1%80%D0%BE%D0%B9%D1%81%D1%82%D0%B2%D0%B5
http://web.archive.org/web/20240901091127/https://habr.com/ru/articles/835228/
http://web.archive.org/web/20240831134917/https://habr.com/ru/articles/776256/
https://habr.com/ru/articles/687512/
https://habr.com/ru/articles/770400/
https://habr.com/ru/articles/727868/
https://habr.com/ru/articles/731608/
https://habr.com/ru/articles/776256/
https://habr.com/ru/articles/774838/
https://habr.com/ru/articles/777656/
https://habr.com/ru/articles/728836/
https://habr.com/ru/articles/774838/


https://habr.com/ru/articles/839656/

https://habr.com/ru/articles/791724/
https://github.com/XTLS/Xray-core/discussions/3518
https://github.com/XTLS/Xray-core



XRAY это впн север. У него есть протоклы. VLESS и VLITE 

Обратите внимание: то, что VLESS не предусматривает шифрования на уровне протокола, не значит, что данные передаются в нешифрованном виде. VLESS всегда работает поверх TLS, трафик шифруется именно механизмами TLS, а не самого VLESS.

Lite есть только в V2Ray (в XRay его нет), поддерживает только передачу UDP-пакетов, и максимально оптимизирован именно для этого, что может быть полезно, например, для онлайн игр, но параллельно придется настроить еще VMess/VLESS для TCP – поэтому я считаю его только “половиной”

С протоколами закончили, перейдем к транспортам. VLESS, VMess и другие могут работать, скажем так, разным образом. Самый простой вариант – обычный TCP-транспорт. VMess+TCP в данном случае очень похож на Shadowsocks, а VLESS+TCP не имеет смысла (из-за отсутствия шифрования). Более интересный вариант – TLS-транспорт, когда устанавливается обычное TLS-подключение (как и в случае с любыми HTTPS-сайтами), а уже внутри этого зашифрованного соединения работает протокол. V2Ray и XRay умеют также работать поверх mKCP (о нем будет в следущих главах), QUIC (aka HTTP/3, правда в России его массово блокируют и смысла в нем мало), gRPC, и самое интересное – через Websockets.

Вариант с Websockets очень ценен тем, что:

1. Позволяет легко поставить V2Ray/XRay не перед, а за Nginx/Caddy/любымдругимвебсервером;
    
2. Позволяет пролезать через строгие корпоративные фаерволы;
    
3. Добавляет дополнительный уровень защиты (не зная URI невозможно достучаться до прокси-сервера);
    
4. И самое интересное – позволяет работать через CDN (upd.: gRPC тоже позволяет). 
    

На последнем пункте остановимся чуть подробнее. Некоторые CDN, в том числе и имеющие бесплатные тарифы, такие как [Cloudflare](https://developers.cloudflare.com/support/network/using-cloudflare-with-websockets/) и [GCore](https://gcore.com/), разрешают проксирование веб-сокетов даже на бесплатных тарифах. Таким образом, это может быть хорошим подспорьем – если по какой-то причине IP-адрес вашего сервера попал в бан, вы все равно можете подключиться к нему через CDN, а полный бан всей CDN гораздо менее вероятен, чем какого-то одного VPS. А еще Cloudflare (возможно и GCore тоже, не уточнял) [умеет](https://community.cloudflare.com/t/proxy-ipv4-visitors-to-ipv6-only-backend/115186/4) проксировать IPv4 запросы на IPv6 адрес, то есть свой прокси-сервер вы можете поднять даже на копеечном (можно найти варианты за 60 центов в месяц!) IPv6-only или NAT VPS без IPv4 адреса, и наплодить таких серверов чуть ли не десяток в разных локациях 🙂

https://paxvpn.com/elementor-830/