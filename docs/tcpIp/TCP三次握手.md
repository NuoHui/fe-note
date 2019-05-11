## 三次握手

<img width="576" alt="屏幕快照 2019-05-11 上午10 46 32" src="https://user-images.githubusercontent.com/42414989/57564167-0fffd500-73da-11e9-9ef5-ba2e4a45a364.png">

分析上图:

- 第一次握手

客户端发送一个syn包到服务端, 等待服务端回应, 进入到syn-send状态。同时会发送一个Seq=j, Seq是一个序列号, 第一次建立连接时候客户端会随机生成一个ISN(32位长到序列号Initial Sequence Number)作为起始原点, 并把该值告诉服务端, 只要为了后续对字节进行序号确认保证传输的可靠性。


- 第二次握手

服务器接受到syn包返回(acknowledgment)ack=j+1表示确认接受, 以及一个syn=k包, 进入到syn-received状态

- 第三次握手

客户端接受到ack=j+1, 进入ESTABLISHED状态。然后根据服务端发过来的syn=k, 返回ack+k+1表示确认接受, 等待服务端接受ack回复。至此服务器与客户端可以进行正常数据收发操作。


引用知乎上一张书中图片：

 ![11](https://user-images.githubusercontent.com/42414989/57564295-535b4300-73dc-11e9-8adb-997c8ada272d.jpg)
