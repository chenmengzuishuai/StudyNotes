# HeartBeat
## 发现问题
使用 go 语言中的socket编程，调试发现客户端意外终止进程，未经过四次挥手，服务端的连接依然存在（资源未释放）
## 解决办法
使用 go 语言net包中Conn对象的SetDeadline()方法，设置timeout，如果超时未接收到消息，在读消息队列的时候，就会获取到一个error，获取到error后，调用Conn.Close()即可关闭连接，释放资源