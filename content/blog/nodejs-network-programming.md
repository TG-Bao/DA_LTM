---
title: "Node.js và lập trình mạng - Sức mạnh của JavaScript trên server"
date: 2024-10-18T10:00:00+07:00
draft: false
author: "Nguyễn Văn A"
description: "Khám phá Node.js cho lập trình mạng và xây dựng ứng dụng real-time"
category: "JavaScript"
tags: ["JavaScript", "Node.js", "Network Programming", "Real-time", "WebSocket"]
readingTime: 14
---

## Node.js - JavaScript không chỉ dành cho browser! 🚀

Node.js đã **cách mạng hóa** thế giới lập trình mạng bằng cách cho phép JavaScript chạy trên server. Với **event-driven, non-blocking I/O**, Node.js trở thành lựa chọn lý tưởng cho các ứng dụng mạng real-time.

## Tại sao Node.js lại mạnh cho Network Programming?

### 1. **Event Loop** - Trái tim của Node.js ⚡

```javascript
// Node.js Event Loop
console.log('1. Start');

setTimeout(() => console.log('2. Timeout'), 0);
setImmediate(() => console.log('3. Immediate'));

process.nextTick(() => console.log('4. NextTick'));

console.log('5. End');

// Output:
// 1. Start
// 5. End
// 4. NextTick
// 2. Timeout
// 3. Immediate
```

### 2. **Non-blocking I/O** 🔄

```javascript
// Blocking I/O (không nên dùng)
const fs = require('fs');
const data = fs.readFileSync('large-file.txt'); // Blocking!
console.log('File read complete');

// Non-blocking I/O (Node.js style)
fs.readFile('large-file.txt', (err, data) => {
    if (err) throw err;
    console.log('File read complete');
});
console.log('This runs immediately!');
```

## HTTP Server với Node.js

### 1. Basic HTTP Server

```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const path = parsedUrl.pathname;
    const method = req.method;
    
    console.log(`${method} ${path}`);
    
    // Set CORS headers
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
    
    // Route handling
    if (method === 'GET' && path === '/api/users') {
        handleGetUsers(req, res);
    } else if (method === 'POST' && path === '/api/users') {
        handleCreateUser(req, res);
    } else if (method === 'GET' && path.startsWith('/api/users/')) {
        const userId = path.split('/')[3];
        handleGetUser(req, res, userId);
    } else {
        res.writeHead(404, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Not Found' }));
    }
});

function handleGetUsers(req, res) {
    const users = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ];
    
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(users));
}

function handleCreateUser(req, res) {
    let body = '';
    
    req.on('data', chunk => {
        body += chunk.toString();
    });
    
    req.on('end', () => {
        try {
            const userData = JSON.parse(body);
            console.log('Creating user:', userData);
            
            // Simulate user creation
            const newUser = {
                id: Date.now(),
                ...userData,
                createdAt: new Date().toISOString()
            };
            
            res.writeHead(201, { 'Content-Type': 'application/json' });
            res.end(JSON.stringify(newUser));
        } catch (error) {
            res.writeHead(400, { 'Content-Type': 'application/json' });
            res.end(JSON.stringify({ error: 'Invalid JSON' }));
        }
    });
}

function handleGetUser(req, res, userId) {
    // Simulate user lookup
    const user = {
        id: parseInt(userId),
        name: 'John Doe',
        email: 'john@example.com'
    };
    
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(user));
}

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`🚀 Server đang chạy trên port ${PORT}`);
});
```

### 2. HTTP Client

```javascript
const http = require('http');
const https = require('https');

// HTTP GET request
function makeHttpGetRequest(url, callback) {
    const protocol = url.startsWith('https') ? https : http;
    
    protocol.get(url, (res) => {
        let data = '';
        
        res.on('data', (chunk) => {
            data += chunk;
        });
        
        res.on('end', () => {
            try {
                const jsonData = JSON.parse(data);
                callback(null, jsonData);
            } catch (error) {
                callback(error, null);
            }
        });
    }).on('error', (error) => {
        callback(error, null);
    });
}

// Sử dụng
makeHttpGetRequest('http://localhost:3000/api/users', (error, data) => {
    if (error) {
        console.error('Error:', error);
    } else {
        console.log('Users:', data);
    }
});

// HTTP POST request
function makeHttpPostRequest(url, data, callback) {
    const urlObj = new URL(url);
    const protocol = urlObj.protocol === 'https:' ? https : http;
    
    const postData = JSON.stringify(data);
    
    const options = {
        hostname: urlObj.hostname,
        port: urlObj.port || (urlObj.protocol === 'https:' ? 443 : 80),
        path: urlObj.pathname,
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Content-Length': Buffer.byteLength(postData)
        }
    };
    
    const req = protocol.request(options, (res) => {
        let responseData = '';
        
        res.on('data', (chunk) => {
            responseData += chunk;
        });
        
        res.on('end', () => {
            try {
                const jsonData = JSON.parse(responseData);
                callback(null, jsonData);
            } catch (error) {
                callback(error, null);
            }
        });
    });
    
    req.on('error', (error) => {
        callback(error, null);
    });
    
    req.write(postData);
    req.end();
}
```

## WebSocket - Real-time Communication

### 1. WebSocket Server

```javascript
const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

console.log('🚀 WebSocket Server đang chạy trên port 8080');

wss.on('connection', (ws, req) => {
    const clientIP = req.socket.remoteAddress;
    console.log(`✅ Client kết nối từ: ${clientIP}`);
    
    // Gửi welcome message
    ws.send(JSON.stringify({
        type: 'welcome',
        message: 'Chào mừng đến với WebSocket Server!',
        timestamp: new Date().toISOString()
    }));
    
    // Xử lý tin nhắn từ client
    ws.on('message', (message) => {
        try {
            const data = JSON.parse(message);
            console.log(`📨 Nhận từ ${clientIP}:`, data);
            
            // Echo lại cho client
            ws.send(JSON.stringify({
                type: 'echo',
                originalMessage: data,
                timestamp: new Date().toISOString()
            }));
            
            // Broadcast cho tất cả clients khác
            wss.clients.forEach((client) => {
                if (client !== ws && client.readyState === WebSocket.OPEN) {
                    client.send(JSON.stringify({
                        type: 'broadcast',
                        from: clientIP,
                        message: data,
                        timestamp: new Date().toISOString()
                    }));
                }
            });
        } catch (error) {
            console.error('❌ Lỗi parse message:', error);
            ws.send(JSON.stringify({
                type: 'error',
                message: 'Invalid JSON format'
            }));
        }
    });
    
    // Xử lý khi client ngắt kết nối
    ws.on('close', () => {
        console.log(`🔌 Client ${clientIP} đã ngắt kết nối`);
    });
    
    // Xử lý lỗi
    ws.on('error', (error) => {
        console.error(`❌ Lỗi WebSocket với ${clientIP}:`, error);
    });
});

// Broadcast message định kỳ
setInterval(() => {
    const message = {
        type: 'server_broadcast',
        message: 'Server đang hoạt động bình thường',
        timestamp: new Date().toISOString(),
        connectedClients: wss.clients.size
    };
    
    wss.clients.forEach((client) => {
        if (client.readyState === WebSocket.OPEN) {
            client.send(JSON.stringify(message));
        }
    });
}, 30000); // Mỗi 30 giây
```

### 2. WebSocket Client

```javascript
const WebSocket = require('ws');

const ws = new WebSocket('ws://localhost:8080');

ws.on('open', () => {
    console.log('🔗 Đã kết nối đến WebSocket Server');
    
    // Gửi tin nhắn test
    ws.send(JSON.stringify({
        type: 'message',
        content: 'Xin chào từ client!',
        user: 'TestUser'
    }));
});

ws.on('message', (data) => {
    try {
        const message = JSON.parse(data);
        console.log('📨 Nhận từ server:', message);
        
        // Xử lý các loại message khác nhau
        switch (message.type) {
            case 'welcome':
                console.log('🎉', message.message);
                break;
            case 'echo':
                console.log('🔄 Echo:', message.originalMessage);
                break;
            case 'broadcast':
                console.log(`📢 Broadcast từ ${message.from}:`, message.message);
                break;
            case 'server_broadcast':
                console.log('📊 Server status:', message.message);
                console.log(`👥 Connected clients: ${message.connectedClients}`);
                break;
            case 'error':
                console.error('❌ Server error:', message.message);
                break;
        }
    } catch (error) {
        console.error('❌ Lỗi parse message:', error);
    }
});

ws.on('close', () => {
    console.log('🔌 Đã ngắt kết nối');
});

ws.on('error', (error) => {
    console.error('❌ WebSocket error:', error);
});

// Gửi tin nhắn định kỳ
setInterval(() => {
    if (ws.readyState === WebSocket.OPEN) {
        ws.send(JSON.stringify({
            type: 'ping',
            timestamp: new Date().toISOString()
        }));
    }
}, 10000); // Mỗi 10 giây
```

## Express.js - Web Framework mạnh mẽ

### 1. Basic Express Server

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet()); // Security headers
app.use(cors()); // CORS
app.use(morgan('combined')); // Logging
app.use(express.json()); // Parse JSON
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded

// Routes
app.get('/', (req, res) => {
    res.json({
        message: 'Welcome to Node.js Network Programming API',
        version: '1.0.0',
        timestamp: new Date().toISOString()
    });
});

app.get('/api/health', (req, res) => {
    res.json({
        status: 'healthy',
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        timestamp: new Date().toISOString()
    });
});

// User routes
app.get('/api/users', (req, res) => {
    const users = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ];
    res.json(users);
});

app.post('/api/users', (req, res) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({
            error: 'Name and email are required'
        });
    }
    
    const newUser = {
        id: Date.now(),
        name,
        email,
        createdAt: new Date().toISOString()
    };
    
    res.status(201).json(newUser);
});

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        error: 'Something went wrong!',
        message: process.env.NODE_ENV === 'development' ? err.message : 'Internal Server Error'
    });
});

// 404 handler
app.use('*', (req, res) => {
    res.status(404).json({
        error: 'Route not found',
        path: req.originalUrl
    });
});

app.listen(PORT, () => {
    console.log(`🚀 Express server đang chạy trên port ${PORT}`);
});
```

### 2. Advanced Express với Middleware

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const slowDown = require('express-slow-down');

const app = express();

// Rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP, please try again later.'
});

// Slow down after rate limit
const speedLimiter = slowDown({
    windowMs: 15 * 60 * 1000, // 15 minutes
    delayAfter: 50, // allow 50 requests per 15 minutes, then...
    delayMs: 500 // begin adding 500ms of delay per request above 50
});

app.use(limiter);
app.use(speedLimiter);

// Custom middleware
const requestLogger = (req, res, next) => {
    console.log(`${new Date().toISOString()} - ${req.method} ${req.path}`);
    next();
};

const responseTime = (req, res, next) => {
    const start = Date.now();
    res.on('finish', () => {
        const duration = Date.now() - start;
        console.log(`Response time: ${duration}ms`);
    });
    next();
};

app.use(requestLogger);
app.use(responseTime);

// Routes với async/await
app.get('/api/async-data', async (req, res) => {
    try {
        // Simulate async operation
        const data = await new Promise((resolve) => {
            setTimeout(() => {
                resolve({ message: 'Async data loaded!' });
            }, 1000);
        });
        
        res.json(data);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

## Socket.io - Real-time Made Easy

### 1. Socket.io Server

```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
    cors: {
        origin: "*",
        methods: ["GET", "POST"]
    }
});

// Store connected users
const users = new Map();

io.on('connection', (socket) => {
    console.log(`✅ User connected: ${socket.id}`);
    
    // Handle user join
    socket.on('join', (userData) => {
        users.set(socket.id, userData);
        socket.broadcast.emit('user_joined', {
            user: userData,
            message: `${userData.name} đã tham gia chat`
        });
        
        // Send current users list
        socket.emit('users_list', Array.from(users.values()));
    });
    
    // Handle chat messages
    socket.on('chat_message', (data) => {
        const user = users.get(socket.id);
        if (user) {
            io.emit('chat_message', {
                user: user.name,
                message: data.message,
                timestamp: new Date().toISOString()
            });
        }
    });
    
    // Handle typing indicator
    socket.on('typing', (data) => {
        const user = users.get(socket.id);
        if (user) {
            socket.broadcast.emit('user_typing', {
                user: user.name,
                isTyping: data.isTyping
            });
        }
    });
    
    // Handle disconnect
    socket.on('disconnect', () => {
        const user = users.get(socket.id);
        if (user) {
            socket.broadcast.emit('user_left', {
                user: user.name,
                message: `${user.name} đã rời khỏi chat`
            });
            users.delete(socket.id);
        }
        console.log(`❌ User disconnected: ${socket.id}`);
    });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`🚀 Socket.io server đang chạy trên port ${PORT}`);
});
```

## Best Practices cho Node.js Network Programming

### 1. **Error Handling** 🛡️

```javascript
// Global error handler
process.on('uncaughtException', (error) => {
    console.error('Uncaught Exception:', error);
    process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};
```

### 2. **Environment Configuration** ⚙️

```javascript
// config.js
const config = {
    port: process.env.PORT || 3000,
    nodeEnv: process.env.NODE_ENV || 'development',
    database: {
        url: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp'
    },
    redis: {
        url: process.env.REDIS_URL || 'redis://localhost:6379'
    }
};

module.exports = config;
```

### 3. **Logging** 📝

```javascript
const winston = require('winston');

const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' })
    ]
});

if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.simple()
    }));
}
```

## Kết luận

Node.js đã **thay đổi hoàn toàn** cách chúng ta xây dựng ứng dụng mạng:

- ✅ **JavaScript Everywhere** - Cùng ngôn ngữ cho frontend và backend
- ✅ **Event-driven Architecture** - Hiệu quả cho I/O intensive applications
- ✅ **Rich Ecosystem** - NPM với hàng triệu packages
- ✅ **Real-time Capabilities** - WebSocket, Socket.io
- ✅ **Microservices Ready** - Dễ dàng scale và deploy

Trong bài tiếp theo, chúng ta sẽ tìm hiểu về **WebSocket với JavaScript** - công nghệ tạo ra các ứng dụng real-time tuyệt vời! 🚀

---

**Tài liệu tham khảo:**
- [Node.js Documentation](https://nodejs.org/docs/)
- [Express.js Guide](https://expressjs.com/)
- [Socket.io Documentation](https://socket.io/docs/)
- [WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
