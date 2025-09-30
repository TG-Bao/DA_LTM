---
title: "Lập trình Socket trong Java - Kết nối mạng cấp thấp"
date: 2024-10-16T10:00:00+07:00
draft: false
author: "Nguyễn Văn A"
description: "Học cách sử dụng Socket để tạo kết nối mạng trong Java"
category: "Java"
tags: ["Java", "Socket", "TCP", "Network Programming", "Server"]
readingTime: 12
---

## Socket Programming - Nền tảng của mạng 🌐

Socket programming là **cách thức cơ bản nhất** để tạo kết nối mạng giữa các ứng dụng. Trong Java, chúng ta có thể dễ dàng tạo ra các ứng dụng client-server sử dụng TCP hoặc UDP.

## Hiểu về Socket

### Socket là gì? 🔌

Socket là một **endpoint** trong giao tiếp mạng. Mỗi socket có:
- **IP Address** - Địa chỉ máy tính
- **Port** - Cổng dịch vụ
- **Protocol** - Giao thức (TCP/UDP)

```java
// Socket = IP + Port + Protocol
Socket socket = new Socket("192.168.1.100", 8080);
```

## TCP Socket Programming

### 1. Tạo Server Socket

```java
import java.net.*;
import java.io.*;

public class TCPServer {
    private static final int PORT = 8080;
    
    public static void main(String[] args) {
        try {
            // Tạo ServerSocket
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("🚀 Server đang chạy trên port " + PORT);
            
            while (true) {
                // Chờ client kết nối
                Socket clientSocket = serverSocket.accept();
                System.out.println("✅ Client kết nối: " + 
                    clientSocket.getInetAddress().getHostAddress());
                
                // Xử lý client trong thread riêng
                new Thread(() -> handleClient(clientSocket)).start();
            }
        } catch (IOException e) {
            System.err.println("❌ Lỗi server: " + e.getMessage());
        }
    }
    
    private static void handleClient(Socket clientSocket) {
        try (BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(
                clientSocket.getOutputStream(), true)) {
            
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                System.out.println("📨 Nhận: " + inputLine);
                
                // Echo lại cho client
                out.println("Echo: " + inputLine);
                
                // Thoát nếu client gửi "bye"
                if ("bye".equalsIgnoreCase(inputLine.trim())) {
                    break;
                }
            }
        } catch (IOException e) {
            System.err.println("❌ Lỗi xử lý client: " + e.getMessage());
        } finally {
            try {
                clientSocket.close();
                System.out.println("🔌 Đóng kết nối client");
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 2. Tạo Client Socket

```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class TCPClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 8080;
    
    public static void main(String[] args) {
        try (Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(
                 socket.getOutputStream(), true);
             Scanner scanner = new Scanner(System.in)) {
            
            System.out.println("🔗 Đã kết nối đến server");
            System.out.println("💡 Gõ 'bye' để thoát");
            
            String userInput;
            while (true) {
                System.out.print("📝 Nhập tin nhắn: ");
                userInput = scanner.nextLine();
                
                // Gửi tin nhắn đến server
                out.println(userInput);
                
                // Nhận phản hồi từ server
                String response = in.readLine();
                System.out.println("📨 Server: " + response);
                
                if ("bye".equalsIgnoreCase(userInput.trim())) {
                    break;
                }
            }
        } catch (IOException e) {
            System.err.println("❌ Lỗi client: " + e.getMessage());
        }
    }
}
```

## UDP Socket Programming

UDP nhanh hơn TCP nhưng không đảm bảo thứ tự và độ tin cậy:

### UDP Server

```java
import java.net.*;
import java.io.*;

public class UDPServer {
    private static final int PORT = 8080;
    private static final int BUFFER_SIZE = 1024;
    
    public static void main(String[] args) {
        try (DatagramSocket socket = new DatagramSocket(PORT)) {
            System.out.println("🚀 UDP Server đang chạy trên port " + PORT);
            
            byte[] buffer = new byte[BUFFER_SIZE];
            
            while (true) {
                // Tạo packet để nhận dữ liệu
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                
                // Nhận dữ liệu
                socket.receive(packet);
                
                // Lấy thông tin client
                InetAddress clientAddress = packet.getAddress();
                int clientPort = packet.getPort();
                String message = new String(packet.getData(), 0, packet.getLength());
                
                System.out.println("📨 Nhận từ " + clientAddress + ":" + clientPort + 
                    " - " + message);
                
                // Gửi phản hồi
                String response = "UDP Echo: " + message;
                byte[] responseData = response.getBytes();
                DatagramPacket responsePacket = new DatagramPacket(
                    responseData, responseData.length, clientAddress, clientPort);
                socket.send(responsePacket);
            }
        } catch (IOException e) {
            System.err.println("❌ Lỗi UDP Server: " + e.getMessage());
        }
    }
}
```

### UDP Client

```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class UDPClient {
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 8080;
    
    public static void main(String[] args) {
        try (DatagramSocket socket = new DatagramSocket();
             Scanner scanner = new Scanner(System.in)) {
            
            InetAddress serverAddress = InetAddress.getByName(SERVER_HOST);
            
            System.out.println("🔗 UDP Client đã sẵn sàng");
            System.out.println("💡 Gõ 'bye' để thoát");
            
            String userInput;
            while (true) {
                System.out.print("📝 Nhập tin nhắn: ");
                userInput = scanner.nextLine();
                
                // Gửi dữ liệu
                byte[] data = userInput.getBytes();
                DatagramPacket packet = new DatagramPacket(
                    data, data.length, serverAddress, SERVER_PORT);
                socket.send(packet);
                
                // Nhận phản hồi
                byte[] buffer = new byte[1024];
                DatagramPacket responsePacket = new DatagramPacket(buffer, buffer.length);
                socket.receive(responsePacket);
                
                String response = new String(responsePacket.getData(), 0, responsePacket.getLength());
                System.out.println("📨 Server: " + response);
                
                if ("bye".equalsIgnoreCase(userInput.trim())) {
                    break;
                }
            }
        } catch (IOException e) {
            System.err.println("❌ Lỗi UDP Client: " + e.getMessage());
        }
    }
}
```

## Multi-threaded Socket Server

Để xử lý nhiều client đồng thời:

```java
import java.net.*;
import java.io.*;
import java.util.concurrent.*;

public class MultiThreadedServer {
    private static final int PORT = 8080;
    private static final int MAX_THREADS = 10;
    
    private final ExecutorService threadPool;
    
    public MultiThreadedServer() {
        this.threadPool = Executors.newFixedThreadPool(MAX_THREADS);
    }
    
    public void start() {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("🚀 Multi-threaded Server đang chạy trên port " + PORT);
            
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("✅ Client kết nối: " + 
                    clientSocket.getInetAddress().getHostAddress());
                
                // Xử lý client trong thread pool
                threadPool.submit(new ClientHandler(clientSocket));
            }
        } catch (IOException e) {
            System.err.println("❌ Lỗi server: " + e.getMessage());
        }
    }
    
    private static class ClientHandler implements Runnable {
        private final Socket clientSocket;
        
        public ClientHandler(Socket socket) {
            this.clientSocket = socket;
        }
        
        @Override
        public void run() {
            try (BufferedReader in = new BufferedReader(
                    new InputStreamReader(clientSocket.getInputStream()));
                 PrintWriter out = new PrintWriter(
                    clientSocket.getOutputStream(), true)) {
                
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    System.out.println("📨 Thread " + Thread.currentThread().getId() + 
                        " nhận: " + inputLine);
                    
                    // Xử lý logic nghiệp vụ
                    String response = processRequest(inputLine);
                    out.println(response);
                    
                    if ("bye".equalsIgnoreCase(inputLine.trim())) {
                        break;
                    }
                }
            } catch (IOException e) {
                System.err.println("❌ Lỗi xử lý client: " + e.getMessage());
            } finally {
                try {
                    clientSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        
        private String processRequest(String request) {
            // Logic xử lý request
            return "Processed: " + request;
        }
    }
    
    public static void main(String[] args) {
        new MultiThreadedServer().start();
    }
}
```

## Best Practices cho Socket Programming

### 1. **Resource Management** 🛡️
```java
// Luôn đóng resources
try (Socket socket = new Socket(host, port);
     BufferedReader in = new BufferedReader(
         new InputStreamReader(socket.getInputStream()))) {
    // Sử dụng socket
} catch (IOException e) {
    // Xử lý lỗi
}
```

### 2. **Exception Handling** ⚠️
```java
try {
    // Socket operations
} catch (ConnectException e) {
    System.err.println("Không thể kết nối đến server");
} catch (SocketTimeoutException e) {
    System.err.println("Timeout khi kết nối");
} catch (IOException e) {
    System.err.println("Lỗi I/O: " + e.getMessage());
}
```

### 3. **Timeout Configuration** ⏰
```java
Socket socket = new Socket();
socket.connect(new InetSocketAddress(host, port), 5000); // 5 giây timeout
```

## Kết luận

Socket programming là **nền tảng** của mọi ứng dụng mạng. Với Java, chúng ta có thể:

- ✅ Tạo TCP/UDP servers và clients
- ✅ Xử lý multi-threading
- ✅ Quản lý resources an toàn
- ✅ Xây dựng các ứng dụng mạng phức tạp

Trong bài tiếp theo, chúng ta sẽ tìm hiểu về **RESTful API với Spring Boot** - một cách tiếp cận hiện đại hơn cho việc xây dựng ứng dụng mạng! 🚀

---

**Tài liệu tham khảo:**
- [Java Socket Documentation](https://docs.oracle.com/javase/tutorial/networking/sockets/)
- [TCP vs UDP](https://www.geeksforgeeks.org/differences-between-tcp-and-udp/)
- [Java Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
