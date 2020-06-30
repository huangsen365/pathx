# PathX 使用

## 为什么看到大量固定的IP地址在访问源站服务器？

这些地址是加速集群出口EIP，起到两个作用：  
1、传输业务流量（可通过toa模块获取访问者真实IP）  
2、对源站服务器做健康检查  
对源站服务器做健康检查是通过TCP三次握手的原理来探测源站可用性，为短连接。

注意：源站相关的云服务商或IDC安全策略，可能对短时间内大量的tcp短连接比较敏感，例如友商的安骑士、云盾会在您不知情的情况下误杀这类IP。安全起见，建议在加速线路详情页获得出口EIP
列表后 提前加入到云盾白名单内（包括友商的CDN厂商信任列表，WAF产品白名单）。

## 如何查看PathX的出口IP？

控制台加速管理或线路管理的资源详情页的基本信息中都可查看。

## 非UCloud服务器是否可以使用全球动态加速？

可以，只要是公网路由可达的服务器即可。

## 网站是否需要备案？

1）若加速区域在中国大陆地区，当您的业务域名CName到加速域名后，需要在中华人民共和国工业和信息化部完成相关备案；客户端直接使用加速域名的情况是不需要备案的。
2）加速区域在中国大陆地区之外，不需要备案。

## HTTP(s)网站或API场景是否可以使用？

1）可以使用，全球动态加速支持tcp透传回源。在控制台配置 TCP 80或443端口，证书仍然部署在您的业务服务器上，不需要其他设置。
2）如果您的业务场景需要就近Offloading SSL, 用HTTP协议回源，可以使用PathX 7层端口转发中HTTPS-HTTP方式。
3）如果您使用的HTTP(s)请求，源站不方便安装TOA模块 又想获得真实客户端IP，可以使用PathX  HTTPS-HTTPS或HTTP-HTTP转发方式。

## 资源使用一段时间后，PathX或GlobalSSH的加速域名+端口突然无法正常访问，而源站+端口可以正常访问

用curl测试一般会报"curl: (56) Failure when receiving data from the peer"
用telnet测试 提示连接成功后，立即收到"Reset By Remote Peer"
重点！重点！重点！用nc测试，则提示连接成功建立。

1. 检查源站是否有安全策略设置，如阿里云的安骑士（云盾会在用户不做任何配置情况下自动封堵，需要开启白名单IP保护 CDN信任厂商），fail2ban等，若有，可以从console资源详情页获取pathx或globalssh 出口ip加入白名单。
2. 以上不能解决您的问题，请先提工单咨询您的源站服务器供应商，是否有自动封堵不明来源IP的安全策略。

2. 检查系统参数设置

    net.ipv4.tcp_timestamps = 1
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 1

开启tcp_tw_resuse足够进行TCP连接的回收，tcp_tw_recycle由于设计的时间较为早期，并没有考虑NAT技术在如今公网已经普及，会导致经过NAT（诸如网吧、4G、WIFI）用户部分的连接失败。当前这个参数已经基本废弃。提升tcp回收速度，也可以通过减小tcp等待timeout的时间来实现。
