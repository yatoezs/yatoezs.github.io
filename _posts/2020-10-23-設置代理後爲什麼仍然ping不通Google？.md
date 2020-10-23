---
layout:	post
title:      設置代理後爲什麼仍然ping不通Google？
subtitle:	衆所周知，由於GFW的封鎖在印度等國不能直連Google，但爲什麼掛上代理可以通過瀏覽器訪問後仍然ping不通呢？
date:	2020-10-23
author:	y1
header-img: img/110474.jpg
catelog:	false
tags:
	- linux
	- network
---

# 設置代理後爲什麼仍然ping不通Google？

### 衆所周知，由於GFW的封鎖在印度等國不能直連Google，但爲什麼掛上代理可以通過瀏覽器訪問後仍然ping不通呢？

​	比較流行的代理工具比如Shadowsocks、ShadowsocksR、V2ray,連上後都ping不通不存在的網站，但是可以通過http/https進行訪問，這是怎麼回事呢？要明白其原因，先查一下這些代理工具的工作原理：

> ​	Shadowsocks的執行原理與其他代理工具基本相同，使用特定的中轉伺服器完成數據傳輸。例如，用戶無法直接存取Google，但代理伺服器可以存取，且用戶可以直接連接代理伺服器，那麼用戶就可以通過特定軟件連接代理伺服器，然後由代理伺服器取得網站內容並回傳給用戶，從而實現代理上網的效果。伺服器和用戶端軟件會要求提供密碼和加密方式，雙方一致後才能成功連接。連接到伺服器後，用戶端會在本機構建一個本地**Socks5**代理（或VPN、透明代理等）。瀏覽網絡時，用戶端通過這個Socks5（或其他形式）代理收集網絡流量，然後再經混淆加密傳送到伺服器端，以防網絡流量被辨識和攔截，反之亦然。
>
> [^Shadowsocks]: https://zh.wikipedia.org/zh-hk/Shadowsocks
>
> 

​	V2ray與之一樣，也是使用了Socks5代理。那麼什麼是Socks5代理呢？維基上這樣描述：

> ​	SOCKS是一種網絡傳輸協定，主要用於客戶端與外網伺服器之間通訊的中間傳遞。SOCKS是"SOCKetS"的縮寫。當防火牆後的客戶端要存取外部的伺服器時，就跟SOCKS代理伺服器連接。這個代理伺服器控制客戶端存取外網的資格，允許的話，就將客戶端的請求發往外部的伺服器。這個協定最初由David Koblas開發，而後由NEC的Ying-Da Lee將其擴充到SOCKS4。最新協定是SOCKS5，與前一版本相比，增加支援UDP、驗證，以及IPv6。根據OSI模型，SOCKS**是對談層的協定**，位於**表示層與傳輸層之間**。 	
>
> [^SOCKS]: https://zh.wikipedia.org/zh-hk/SOCKS

​	現在我們知道了：SOCKS是工作在表示層與傳輸層之間——即會話層，那麼ping命令是如何工作的？

> ​	ping（呯）是一種電腦網絡工具，用來測試封包能否透過IP協定到達特定主機。ping的運作原理是向目標主機傳出一個**ICMP**的請求回顯封包，並等待接收回顯回應封包。程式會按時間和成功響應的次數估算遺失封包率（丟包率）和封包往返時間（網絡時延，Round-trip delay time）。
>
> [^ping]: https://zh.wikipedia.org/zh-hk/Ping

> ​	互聯網控制訊息協定（英語：Internet Control Message Protocol，縮寫：ICMP）是互聯網協定套組的核心協定之一。它用於網際網絡協定（IP）中傳送控制訊息，提供可能發生在通訊環境中的各種問題反饋。通過這些資訊，使管理者可以對所發生的問題作出診斷，然後採取適當的措施解決。 
>
> [^ICMP]: https://zh.wikipedia.org/zh-hk/ICMP

​	顯然ping命令使用的是ICMP協議，而ICMP工作在網絡層。

​	下面回顧一下OSI模型：

| OSI模型（Open System Interconnection Model） |
| :------------------------------------------: |
|            第七層 應用層（HTTP）             |
|                第六層 表示層                 |
|           第五層 會話層（SOCKS5）            |
|                第四層 傳輸層                 |
|            第三層 網絡層（ICMP）             |
|              第二層 數據鏈路層               |
|                第一層 物理層                 |

​	到這裏問題的答案就很明顯了：代理工具工作在會話層，而ping命令工作在網絡層，由於在計算機網絡中服務是由下層提供給上層，所以代理工具對ping命令是沒有效用的。