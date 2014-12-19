---
layout: post
title: VPN для избранных приложений с помощью неймспейсов Linux
description: Настройка OpenVPN в сетевом неймспейсе Linux для тунеллирования избранных приложений.
categories: linux
tags: linux, networking, namespaces, vpn, openvpn, frootvpn
permalink: /blog/linux/vpn-in-a-namespace-for-selected-applications/
sitemap:
  priority: 0.5
  changefreq: 'monthly'
---

Попробовав [бесплатный (пока) VPN](https://frootvpn.com) и приятно удивившись необычно высокой для таких проектов скоростью работы, я пристрастился и стал пользоваться сервисом регулярно.

Единственной проблемой стало то, что при подключении OpenVPN заменяет основной шлюз системы на свой, и, соответственно, пускает весь трафик через VPN. Подобная ситуация не устраивала по ряду причин и нужно было что-то делать.

Беглый поиск дал 2 решения: использовать конфигурацию с 2 основными шлюзами (рулить трафиком между ними с помощью ```rt_tables```) и использовать ```LD_PRELOAD``` для биндинга на виртуальный IP или запускать OpenVPN и использующие его приложения с отдельном неймспейсе.

Второй вариант показался меньшим костылем, все команды выполняются под рутом.


Переменные:
{% highlight bash %}

# Имя неймспейса
NS_NAME=vpn
# Имя интерфейса
BASE_ETH=eth0
# Текущий IP
BASE_IP=192.168.1.2
# Текущий шлюз
DEF_GW=192.168.1.1
# Маска
NETMASK=255.255.255.0
# IP виртуального интерфейса неймспейса
NS_IP=192.168.1.3

{% endhighlight %}

Создаем неймспейс:
{% highlight bash %}
ip netns add $NS_NAME
{% endhighlight %}

Добавляем виртуальные интерфейсы:
{% highlight bash %}
ip link add veth0 type veth peer name veth1
ifconfig veth0 0.0.0.0 up
ip link set veth1 netns $NS_NAME
{% endhighlight %}

Отключаем основной интерфейс, он будет объединен в мост:
{% highlight bash %}
ifdown $BASE_ETH
ifconfig $BASE_ETH 0.0.0.0 up
{% endhighlight %}

Создаем мост:
{% highlight bash %}
brctl addbr br0
brctl addif br0 $BASE_ETH veth0
ifconfig br0 $BASE_IP netmask $NETMASK up
route add default gw $DEF_GW
{% endhighlight %}

Включаем IPv6 (в новых системах включен по умолчанию):
{% highlight bash %}
sysctl -w net.ipv6.conf.all.disable_ipv6=0
{% endhighlight %}

Теперь нужно присвоить IP вируальному интерфейсу неймспейса и добавить основной шлюз.

Для этого команды необходимо выполнять в неймспейсе, и это удобно делать из шелла уже в неймспейсе:

{% highlight bash %}
ip netns exec $NS_NAME bash
{% endhighlight %}

Присваиваем IP:
{% highlight bash %}
ifconfig veth1 $NS_IP netmask $NETMASK up
route add default gw $DEF_GW
{% endhighlight %}

У меня установлен локальный кэширующий DNS (в resolv.conf указан localhost), привязанный к $BASE_ETH.
Чтобы в неймспейсе резолвить адреса мы перенаправим обращения к локальному серверу на OpenDNS:
{% highlight bash %}
OLD_DNS=127.0.0.1
NEW_DNS=208.67.222.222

iptables -t nat -A OUTPUT -p tcp -d $OLD_DNS -j DNAT --dport 53 --to-destination $NEW_DNS
iptables -t nat -A INPUT  -p tcp -s $OLD_DNS --sport 53 -j SNAT --to-source      $NEW_DNS
iptables -t nat -A OUTPUT -p udp -d $OLD_DNS -j DNAT --dport 53 --to-destination $NEW_DNS
iptables -t nat -A INPUT  -p udp -s $OLD_DNS --sport 53 -j SNAT --to-source      $NEW_DNS

{% endhighlight %}

Запускаем OpenVPN:
{% highlight bash %}
openvpn --auth-user-pass /etc/openvpn/frootvpn.auth --config /etc/openvpn/frootvpn.ovpn
{% endhighlight %}

После установки соединения шелл окажется занят процессом OpenVPN. Открываем еще один шелл в неймспейсе под обычным пользователем, из которого можно запускать нужные программы:
{% highlight bash %}
ip netns exec $NS_NAME bash -c 'su stanislav'
chromium-browser
{% endhighlight %}

[Источник](http://www.evolware.org/?p=293)
