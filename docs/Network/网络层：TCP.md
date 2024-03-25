Java 实现 Echo 协议代码。
```java
package com.haizhi.hivetest;  
  
import java.io.*;  
import java.net.ServerSocket;  
import java.net.Socket;  
import java.nio.charset.StandardCharsets;  
import java.util.Scanner;  
  
public class EchoServer {  
  
    private static final int PORT = 2333;  
  
    public static void main(String[] args) {  
  
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {  
  
            while (true) {  
                Socket accept = serverSocket.accept();  
                new Thread(() -> {  
                    try (InputStream inputStream = accept.getInputStream();  
                         OutputStream outputStream = accept.getOutputStream();  
                         Scanner in = new Scanner(inputStream, "UTF-8");  
                         PrintWriter out = new PrintWriter(new OutputStreamWriter(outputStream, StandardCharsets.UTF_8), true)) {  
                          
                        out.println("Hello! Enter BYE to exit.");  
  
                        boolean done = false;  
                        while (!done && in.hasNextLine()) {  
                            String line = in.nextLine();  
                            out.println("Echo: " + line);  
                            if (line.trim().equalsIgnoreCase("bye")) {  
                                done = true;  
                            }  
                        }  
                    } catch (IOException e) {  
                        e.printStackTrace();  
                    }  
                }).start();  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}
```

使用 telnet 程序测试。先进性三次握手，telnet 会每敲一个字符就发送一个字符 TCP 包，
```
telnet localhost 2333
Hello! Enter BYE to exit.
a
Echo: a
hello echo server!
Echo: hello echo server!
bye
Echo: bye
```

