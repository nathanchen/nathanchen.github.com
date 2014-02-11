---
layout: post
title: "Nagle算法和Delayed ACK"
category: Reading Notes
tags: ["读文章", "网络"]
---
{% include JB/setup %}

## Nagle算法和Delayed ACK

#### Nagle算法

某个应用程序不断的发出小单位的数据，且某些只占1字节大小。因为TCP封包有40字节的报头，这导致了41字节大小的封包中只有1字节的可用数据。

	if 有新資料要傳送
	   if 訊窗大小 >= MSS and 可傳送的資料 >= MSS
	     立刻傳送完整MSS大小的segment
	   else
	    if 管線中有尚未確認的資料，即之前发送的数据未收到ACK
	      在下一個確認(ACK)封包收到前，將資料排進緩衝區队列
	    else
	      立即傳送資料  

#### TCP delayed acknowledgment

several ACK responses may be combined together into a single response, reducing protocol overhead.

#### Probelm

If Nagle's algorithm is being used by the sending party, data will be queued by the sender until an ACK is received. If the sender does not send enough data to fill the maximum segment size, then the transfer will pause up to the ACK delay timeout.





#### Reference