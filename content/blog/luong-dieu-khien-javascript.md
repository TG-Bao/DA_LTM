---
title: "Luồng điều khiển JavaScript - Nghệ thuật lập trình thông minh"
date: 2024-10-15T12:00:00+07:00
draft: false
description: "Khám phá nghệ thuật luồng điều khiển JavaScript và cách tạo ra logic thông minh"
category: "JavaScript"
tags: ["JavaScript", "Luồng điều khiển", "Nghệ thuật", "Logic"]
readingTime: "18 phút"
author: "TG-Bao"
---

## 🔀 Luồng điều khiển trong JavaScript

Chào các bạn! Hôm nay tôi muốn chia sẻ với các bạn về luồng điều khiển trong JavaScript - một khái niệm quan trọng giúp chương trình thực thi logic một cách thông minh.

### 🎯 Luồng điều khiển - Tại sao lại quan trọng?

Luồng điều khiển giúp chương trình thực thi các đoạn code khác nhau dựa trên điều kiện và lặp lại các thao tác. Đây là nền tảng để tạo ra các ứng dụng tương tác và thông minh.

**Tầm quan trọng của Control Flow**: Luồng điều khiển cho phép chương trình thực thi các đoạn code khác nhau dựa trên điều kiện và lặp lại các thao tác. Đây là nền tảng để tạo ra logic phức tạp và ứng dụng tương tác.

### 🎯 Thực thi có điều kiện

#### 1. **If/Else Statement**

```javascript
let age = 18;

// If đơn giản
if (age >= 18) {
    console.log("Bạn đã trưởng thành");
}

// If/Else
if (age >= 18) {
    console.log("Bạn có thể bầu cử");
} else {
    console.log("Bạn chưa đủ tuổi bầu cử");
}

// If/Else If/Else
let score = 85;

if (score >= 90) {
    console.log("Xuất sắc (A)");
} else if (score >= 80) {
    console.log("Giỏi (B)");
} else if (score >= 70) {
    console.log("Khá (C)");
} else if (score >= 60) {
    console.log("Trung bình (D)");
} else {
    console.log("Yếu (F)");
}
```

Mẹo của mình khi viết nhánh điều kiện:
- Sắp xếp điều kiện theo thứ tự loại trừ (từ chặt nhất đến rộng dần) để tránh trùng lặp logic.
- Khi thấy `if/else if` quá dài, xem có thể đổi sang `lookup table` (object/dictionary) cho gọn không.

#### 2. **Ternary Operator (Toán tử 3 ngôi)**

```javascript
let age = 20;
let status = age >= 18 ? "Trưởng thành" : "Chưa trưởng thành";
console.log(status); // "Trưởng thành"

// Nested ternary
let grade = score >= 90 ? "A" : 
            score >= 80 ? "B" : 
            score >= 70 ? "C" : "D";
```

Toán tử 3 ngôi chỉ nên dùng cho nhánh ngắn, nếu lồng nhau quá 1 cấp thì mình quay về `if/else` để dễ đọc.

#### 3. **Switch Statement**

```javascript
let day = 3;
let dayName;

switch (day) {
    case 1:
        dayName = "Thứ Hai";
        break;
    case 2:
        dayName = "Thứ Ba";
        break;
    case 3:
        dayName = "Thứ Tư";
        break;
    case 4:
        dayName = "Thứ Năm";
        break;
    case 5:
        dayName = "Thứ Sáu";
        break;
    case 6:
        dayName = "Thứ Bảy";
        break;
    case 7:
        dayName = "Chủ Nhật";
        break;
    default:
        dayName = "Không hợp lệ";
}

console.log(dayName); // "Thứ Tư"

// Switch với nhiều case
let month = 2;
let season;

switch (month) {
    case 12:
    case 1:
    case 2:
        season = "Mùa đông";
        break;
    case 3:
    case 4:
    case 5:
        season = "Mùa xuân";
        break;
    case 6:
    case 7:
    case 8:
        season = "Mùa hè";
        break;
    case 9:
    case 10:
    case 11:
        season = "Mùa thu";
        break;
    default:
        season = "Tháng không hợp lệ";
}
```

### 🔄 Vòng lặp (Loops)

#### 1. **For Loop**

```javascript
// Vòng lặp for cơ bản
for (let i = 0; i < 5; i++) {
    console.log(`Lần lặp thứ ${i + 1}`);
}

// Vòng lặp với mảng
let fruits = ["apple", "banana", "orange"];
for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i]);
}

// Vòng lặp ngược
for (let i = 5; i >= 1; i--) {
    console.log(`Đếm ngược: ${i}`);
}
```

#### 2. **While Loop**

```javascript
// While loop cơ bản
let count = 0;
while (count < 5) {
    console.log(`Count: ${count}`);
    count++;
}

// While với điều kiện phức tạp
let number = 1;
while (number <= 100) {
    if (number % 2 === 0) {
        console.log(`${number} là số chẵn`);
    }
    number++;
}
```

#### 3. **Do-While Loop**

```javascript
// Do-while luôn thực thi ít nhất 1 lần
let userInput;
do {
    userInput = prompt("Nhập số từ 1-10:");
} while (userInput < 1 || userInput > 10);

console.log(`Bạn đã nhập: ${userInput}`);
```

#### 4. **For-In và For-Of**

```javascript
// For-in: Duyệt qua keys của object
let person = {
    name: "TG-Bao",
    age: 25,
    city: "Hồ Chí Minh"
};

for (let key in person) {
    console.log(`${key}: ${person[key]}`);
}

// For-of: Duyệt qua values của array
let colors = ["red", "green", "blue"];
for (let color of colors) {
    console.log(color);
}

// For-of với string
let text = "JavaScript";
for (let char of text) {
    console.log(char);
}
```

### 🎮 Ví dụ thực tế: Game đoán số

```html
<!DOCTYPE html>
<html>
<head>
    <title>Game Đoán Số</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }
        .game-container {
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
            border: 2px solid #ccc;
            border-radius: 10px;
        }
        input, button {
            padding: 10px;
            margin: 5px;
            font-size: 16px;
        }
        #result {
            margin-top: 20px;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>🎯 Game Đoán Số</h1>
        <p>Đoán số từ 1 đến 100</p>
        <input type="number" id="guessInput" placeholder="Nhập số đoán">
        <button onclick="checkGuess()">Đoán</button>
        <button onclick="newGame()">Game Mới</button>
        <div id="result"></div>
        <div id="attempts"></div>
    </div>

    <script>
        let secretNumber = Math.floor(Math.random() * 100) + 1;
        let attempts = 0;
        let maxAttempts = 7;

        function checkGuess() {
            let guess = parseInt(document.getElementById('guessInput').value);
            let resultDiv = document.getElementById('result');
            let attemptsDiv = document.getElementById('attempts');
            
            if (isNaN(guess) || guess < 1 || guess > 100) {
                resultDiv.textContent = "Vui lòng nhập số từ 1 đến 100!";
                resultDiv.style.color = "red";
                return;
            }

            attempts++;
            attemptsDiv.textContent = `Lần thử: ${attempts}/${maxAttempts}`;

            if (guess === secretNumber) {
                resultDiv.textContent = `🎉 Chúc mừng! Bạn đã đoán đúng số ${secretNumber} sau ${attempts} lần thử!`;
                resultDiv.style.color = "green";
                document.getElementById('guessInput').disabled = true;
            } else if (attempts >= maxAttempts) {
                resultDiv.textContent = `😞 Game Over! Số bí mật là ${secretNumber}`;
                resultDiv.style.color = "red";
                document.getElementById('guessInput').disabled = true;
            } else if (guess < secretNumber) {
                resultDiv.textContent = "📈 Số quá nhỏ! Hãy thử số lớn hơn.";
                resultDiv.style.color = "orange";
            } else {
                resultDiv.textContent = "📉 Số quá lớn! Hãy thử số nhỏ hơn.";
                resultDiv.style.color = "orange";
            }

            document.getElementById('guessInput').value = '';
        }

        function newGame() {
            secretNumber = Math.floor(Math.random() * 100) + 1;
            attempts = 0;
            document.getElementById('result').textContent = '';
            document.getElementById('attempts').textContent = '';
            document.getElementById('guessInput').disabled = false;
            document.getElementById('guessInput').value = '';
        }

        // Xử lý phím Enter
        document.getElementById('guessInput').addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                checkGuess();
            }
        });
    </script>
</body>
</html>
```

### 🎯 Ví dụ: Bảng cửu chương

```javascript
// Tạo bảng cửu chương
function createMultiplicationTable() {
    console.log("📊 Bảng cửu chương:");
    console.log("==================");
    
    for (let i = 1; i <= 10; i++) {
        console.log(`\nBảng ${i}:`);
        for (let j = 1; j <= 10; j++) {
            console.log(`${i} x ${j} = ${i * j}`);
        }
    }
}

createMultiplicationTable();
```

### 🎯 Ví dụ: Tìm số nguyên tố

```javascript
function isPrime(number) {
    if (number < 2) return false;
    if (number === 2) return true;
    if (number % 2 === 0) return false;
    
    for (let i = 3; i <= Math.sqrt(number); i += 2) {
        if (number % i === 0) return false;
    }
    return true;
}

function findPrimesInRange(start, end) {
    let primes = [];
    
    for (let i = start; i <= end; i++) {
        if (isPrime(i)) {
            primes.push(i);
        }
    }
    
    return primes;
}

// Tìm số nguyên tố từ 1 đến 100
let primes = findPrimesInRange(1, 100);
console.log("Số nguyên tố từ 1-100:", primes);
```

### 🎓 Bài tập thực hành

```javascript
// Bài tập 1: Kiểm tra năm nhuận
function isLeapYear(year) {
    if (year % 4 === 0) {
        if (year % 100 === 0) {
            return year % 400 === 0;
        }
        return true;
    }
    return false;
}

// Bài tập 2: Tính giai thừa
function factorial(n) {
    let result = 1;
    for (let i = 1; i <= n; i++) {
        result *= i;
    }
    return result;
}

// Bài tập 3: Fibonacci
function fibonacci(n) {
    let a = 0, b = 1;
    for (let i = 0; i < n; i++) {
        console.log(a);
        let temp = a + b;
        a = b;
        b = temp;
    }
}
```

### 🎯 Kết luận

Luồng điều khiển là nền tảng quan trọng trong lập trình. Nó giúp bạn tạo ra các chương trình thông minh và tương tác.

**Hãy thực hành nhiều để nắm vững các khái niệm này!** 🚀
