
Вопросы active probing???

VLESS имеет гораздо более широкие возможности: Например, он может работать через CDN (Content Delivery Networks) благодаря транспорту WS, gRPC и HTTPUpgrade. ????

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
