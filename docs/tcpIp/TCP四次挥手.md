## TCP四次挥手

<img width="558" alt="屏幕快照 2019-05-11 上午11 12 58" src="https://user-images.githubusercontent.com/42414989/57564414-c1543a00-73dd-11e9-88c8-48baa4ee2894.png">

- 第一次挥手

事实上任何一方都可以先表示断开连接, 这里以客户端为例。首先客户端发送FIN包➕一个Seq=M(客户端的ibn号), 此时表示客户端进入FIN-WAIT-1状态。表示客户端已经没有任何数据需要再发送了。

- 第二次挥手

服务端接受到客户端发过来到FIN M包, 然后向客户端返回ACK=M+1, 此时服务端进入CLOSE-WAIT阶段, 客户端进入FIN-WAIT-2状态。

- 第三次挥手

服务端向客户端发送FIN包➕Seq=N, 请求关闭连接，同时server进入LAST-ACK状态。

- 第四次挥手

client收到server发送的FIN(N)包，进入TIME-WAIT状态。向server发送**ACK(N+1)**包，server收到client的ACK(N+1)包以后，进入CLOSE状态；client等待一段时间还没有得到回复后判断server已正式关闭，进入CLOSE状态。

最后通过指数一个完整的图来描述:
![22](https://user-images.githubusercontent.com/42414989/57564488-062ca080-73df-11e9-9aa1-e0f79c805969.jpg)



