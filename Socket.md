# 用代码检验 go socket 阻塞或非阻塞
## 代码如下
```
package main

import (
	"bufio"
	"fmt"
	"net"
	"strings"
	"time"
)

func process(conn net.Conn, timeout int) {
	defer conn.Close()

	for {

		//声明一个从连接conn中读取消息的读取器
		reader := bufio.NewReader(conn)

		var buffer [512]byte

		//用切片的方式从读取器中把消息字节读取到数组变量buffer中
		n, err := reader.Read(buffer[:])

		if err != nil {
			fmt.Println("HeartBeat timeout,server close this connection")
			return
		}

		if n > 0 {
			reciveStr := string(buffer[:n])
			if strings.ToUpper(reciveStr) == "Q" {
				return
			}
			fmt.Println(reciveStr)
			conn.SetDeadline(time.Now().Add(time.Duration(timeout) * time.Second))
		} else {
			fmt.Println("No messages,unblocked")
		}

		//把收到的字节消息转换为字符串

		/*
			//组装回复消息字节，拼接字节切片
			replyStrSlice := append([]byte("You just sent me:"), buffer[:n]...)

			conn.Write(replyStrSlice)
		*/

	}
}

func main() {
	//先创建一个监听端口的变量listen
	listen, err := net.Listen("tcp4", ":8000")

	if err != nil {
		//fmt.Println("Listenning port 8000 failed ,err:", err)
		return
	}

	//这个监听端口的变量listen似乎是一个消息队列，有连接进来的时候也是遵循先进先出的规则

	for {

		conn, err := listen.Accept()
		time.Sleep(1000)
		fmt.Println("continue")
		if err != nil {
			//fmt.Println("Accepted connection from var listen failed,err:", err)
			return
		}
		fmt.Printf("Connected from %v\n",conn.RemoteAddr())
		go process(conn, 600)

	}
}
```
## 在windows中
实验结果，不管是读取连接队列还是连接后读取消息，都是阻塞的，也就是
`fmt.Printf("Connected from %v\n",conn.RemoteAddr())`在没有连接的时候没有持续打印，且`fmt.Println("No messages,unblocked")`在没有消息发送的时候也没有持续打印。因为如果内核是非阻塞的，那么这两行语句应该是持续打印的。