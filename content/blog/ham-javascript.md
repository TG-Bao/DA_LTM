---
title: "Hàm JavaScript - Siêu năng lực của lập trình viên"
date: 2025-09-15T13:00:00+07:00
draft: false
description: "Khám phá siêu năng lực của hàm JavaScript và cách sử dụng chúng để tạo ra code tuyệt vời"
category: "JavaScript"
tags: ["JavaScript", "Hàm", "Siêu năng lực", "Lập trình viên"]
readingTime: "20 phút"
author: "TG-Bao"
---

## 🎯 Hàm trong JavaScript - Kiến thức cần thiết

Chào các bạn! Hôm nay tôi muốn chia sẻ với các bạn về hàm trong JavaScript - một trong những khái niệm quan trọng nhất trong lập trình. Hàm giúp code trở nên có tổ chức và dễ bảo trì hơn.

### 📝 Hàm - Tại sao lại quan trọng?

Hàm là một khối code có thể được gọi nhiều lần để thực hiện một tác vụ cụ thể. Sử dụng hàm giúp code dễ đọc, dễ bảo trì và tái sử dụng.

**Nguyên tắc DRY (Don't Repeat Yourself)**: Hàm giúp tái sử dụng code, giảm duplicate và tăng maintainability. Một hàm tốt nên có Single Responsibility - chỉ làm một việc duy nhất, dễ test và debug.

### 📝 Cách tôi học khai báo hàm

#### 1. **Function Declaration - Cách đầu tiên tôi học**

```javascript
// Cách đầu tiên tôi học viết hàm
function greet(name) {
  return `Xin chào ${name}!`;
}

// Gọi hàm
let message = greet("TG-Bao");
console.log(message); // "Xin chào TG-Bao!"

// Hàm với nhiều tham số - tôi đã học sau
function calculateArea(width, height) {
  return width * height;
}

let area = calculateArea(5, 3);
console.log(area); // 15

// Hàm không có tham số - rất hữu ích!
function getCurrentTime() {
  return new Date().toLocaleTimeString();
}

console.log(getCurrentTime()); // "14:30:25"
```

Nhìn nhanh lại: đây là cách mình đi từ cơ bản đến thực dụng khi học hàm. Mấu chốt là viết hàm ngắn, đặt tên rõ nghĩa và để mỗi hàm giải quyết đúng một nhiệm vụ. Khi thấy một khối code lặp lại từ 2 lần trở lên, mình trích nó thành hàm riêng để dễ test và tái sử dụng.

**Function Declaration vs Expression**: Function Declaration được "hoisted" - có thể gọi trước khi khai báo. Function Expression không được hoisted, phải khai báo trước khi sử dụng. Arrow Function (ES6) không có `this` binding và không thể dùng làm constructor.

#### 2. **Function Expression**

```javascript
// Gán hàm vào biến
const add = function (a, b) {
  return a + b;
};

console.log(add(5, 3)); // 8

// Hàm ẩn danh (anonymous function)
const multiply = function (x, y) {
  return x * y;
};
```

#### 3. **Arrow Function (ES6)**

```javascript
// Arrow function cơ bản
const greet = (name) => {
  return `Xin chào ${name}!`;
};

// Arrow function ngắn gọn (1 dòng)
const add = (a, b) => a + b;
const square = (x) => x * x;

// Arrow function không có tham số
const getRandom = () => Math.random();

// Arrow function với nhiều tham số
const calculate = (a, b, c) => a + b * c;
```

Khi chọn giữa Declaration/Expression/Arrow, mình dùng quy tắc đơn giản:

- Cần hoisting hoặc định nghĩa API ở top-level → dùng Function Declaration.
- Cần truyền hàm như giá trị, gán động → Function Expression.
- Callback ngắn gọn, không cần `this/arguments` → Arrow Function.

### 🔄 Tham số và đối số

#### 1. **Tham số mặc định**

```javascript
// Tham số mặc định
function greet(name = "Người dùng") {
  return `Xin chào ${name}!`;
}

console.log(greet()); // "Xin chào Người dùng!"
console.log(greet("TG-Bao")); // "Xin chào TG-Bao!"

// Nhiều tham số mặc định
function createUser(name, age = 18, isActive = true) {
  return {
    name: name,
    age: age,
    isActive: isActive,
  };
}

let user1 = createUser("TG-Bao");
let user2 = createUser("Alice", 25, false);
```

#### 2. **Rest Parameters**

```javascript
// Rest parameters - nhận nhiều tham số
function sum(...numbers) {
  let total = 0;
  for (let number of numbers) {
    total += number;
  }
  return total;
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// Kết hợp tham số thường và rest
function greet(greeting, ...names) {
  return `${greeting} ${names.join(", ")}!`;
}

console.log(greet("Xin chào", "TG-Bao", "Alice", "Bob"));
// "Xin chào TG-Bao, Alice, Bob!"
```

#### 3. **Arguments Object**

```javascript
function showArguments() {
  console.log("Số lượng tham số:", arguments.length);
  for (let i = 0; i < arguments.length; i++) {
    console.log(`Tham số ${i}:`, arguments[i]);
  }
}

showArguments("a", "b", "c");
// Số lượng tham số: 3
// Tham số 0: a
// Tham số 1: b
// Tham số 2: c
```

### 🎯 Callback Functions

```javascript
// Hàm callback đơn giản
function processArray(arr, callback) {
  let result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(callback(arr[i]));
  }
  return result;
}

// Sử dụng callback
let numbers = [1, 2, 3, 4, 5];

// Callback để nhân đôi
let doubled = processArray(numbers, function (x) {
  return x * 2;
});
console.log(doubled); // [2, 4, 6, 8, 10]

// Callback với arrow function
let squared = processArray(numbers, (x) => x * x);
console.log(squared); // [1, 4, 9, 16, 25]
```

### 🔄 Higher-Order Functions

#### 1. **Map, Filter, Reduce**

```javascript
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Map - biến đổi từng phần tử
let doubled = numbers.map((x) => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

// Filter - lọc phần tử theo điều kiện
let evenNumbers = numbers.filter((x) => x % 2 === 0);
console.log(evenNumbers); // [2, 4, 6, 8, 10]

// Reduce - tính tổng
let sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 55

// Reduce phức tạp - tìm số lớn nhất
let max = numbers.reduce((acc, curr) => (acc > curr ? acc : curr));
console.log(max); // 10
```

#### 2. **Tạo Higher-Order Function**

```javascript
// Hàm tạo hàm
function createMultiplier(factor) {
  return function (number) {
    return number * factor;
  };
}

let double = createMultiplier(2);
let triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Hàm tạo validator
function createValidator(min, max) {
  return function (value) {
    return value >= min && value <= max;
  };
}

let ageValidator = createValidator(18, 65);
console.log(ageValidator(25)); // true
console.log(ageValidator(70)); // false
```

### 🎮 Ví dụ thực tế: Calculator với hàm

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Calculator với Hàm</title>
    <style>
      .calculator {
        width: 300px;
        margin: 50px auto;
        padding: 20px;
        border: 2px solid #ccc;
        border-radius: 10px;
      }
      .display {
        width: 100%;
        height: 50px;
        font-size: 24px;
        text-align: right;
        margin-bottom: 10px;
      }
      .buttons {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 10px;
      }
      button {
        height: 50px;
        font-size: 18px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
      }
    </style>
  </head>
  <body>
    <div class="calculator">
      <input type="text" class="display" id="display" readonly />
      <div class="buttons">
        <button onclick="clearDisplay()">C</button>
        <button onclick="deleteLast()">⌫</button>
        <button onclick="appendToDisplay('/')">/</button>
        <button onclick="appendToDisplay('*')">*</button>

        <button onclick="appendToDisplay('7')">7</button>
        <button onclick="appendToDisplay('8')">8</button>
        <button onclick="appendToDisplay('9')">9</button>
        <button onclick="appendToDisplay('-')">-</button>

        <button onclick="appendToDisplay('4')">4</button>
        <button onclick="appendToDisplay('5')">5</button>
        <button onclick="appendToDisplay('6')">6</button>
        <button onclick="appendToDisplay('+')">+</button>

        <button onclick="appendToDisplay('1')">1</button>
        <button onclick="appendToDisplay('2')">2</button>
        <button onclick="appendToDisplay('3')">3</button>
        <button onclick="calculate()" rowspan="2">=</button>

        <button onclick="appendToDisplay('0')" colspan="2">0</button>
        <button onclick="appendToDisplay('.')">.</button>
      </div>
    </div>

    <script>
      // Hàm tiện ích
      const getDisplay = () => document.getElementById("display");

      const setDisplay = (value) => {
        getDisplay().value = value;
      };

      const getDisplayValue = () => getDisplay().value;

      // Hàm chính
      function appendToDisplay(value) {
        const currentValue = getDisplayValue();
        setDisplay(currentValue + value);
      }

      function clearDisplay() {
        setDisplay("");
      }

      function deleteLast() {
        const currentValue = getDisplayValue();
        setDisplay(currentValue.slice(0, -1));
      }

      function calculate() {
        try {
          const expression = getDisplayValue();
          const result = evaluateExpression(expression);
          setDisplay(result);
        } catch (error) {
          setDisplay("Error");
        }
      }

      // Hàm đánh giá biểu thức an toàn
      function evaluateExpression(expression) {
        // Chỉ cho phép số và các toán tử cơ bản
        const allowedChars = /^[0-9+\-*/.() ]+$/;
        if (!allowedChars.test(expression)) {
          throw new Error("Invalid expression");
        }

        return Function('"use strict"; return (' + expression + ")")();
      }

      // Xử lý phím Enter
      document.addEventListener("keydown", function (event) {
        if (event.key === "Enter") {
          calculate();
        }
      });
    </script>
  </body>
</html>
```

### 🎯 Ví dụ: Hệ thống quản lý người dùng

```javascript
// Hàm tạo người dùng
function createUser(name, age, email) {
  return {
    id: Date.now(), // ID đơn giản
    name: name,
    age: age,
    email: email,
    isActive: true,
    createdAt: new Date(),
  };
}

// Hàm thêm người dùng
function addUser(users, name, age, email) {
  const newUser = createUser(name, age, email);
  users.push(newUser);
  return newUser;
}

// Hàm tìm người dùng
function findUser(users, id) {
  return users.find((user) => user.id === id);
}

// Hàm lọc người dùng theo điều kiện
function filterUsers(users, condition) {
  return users.filter(condition);
}

// Hàm cập nhật người dùng
function updateUser(users, id, updates) {
  const user = findUser(users, id);
  if (user) {
    Object.assign(user, updates);
    return user;
  }
  return null;
}

// Hàm xóa người dùng
function deleteUser(users, id) {
  const index = users.findIndex((user) => user.id === id);
  if (index !== -1) {
    return users.splice(index, 1)[0];
  }
  return null;
}

// Sử dụng hệ thống
let users = [];

// Thêm người dùng
addUser(users, "TG-Bao", 25, "tgbao@example.com");
addUser(users, "Alice", 30, "alice@example.com");
addUser(users, "Bob", 28, "bob@example.com");

// Tìm người dùng
let user = findUser(users, users[0].id);
console.log("Tìm thấy:", user);

// Lọc người dùng theo tuổi
let youngUsers = filterUsers(users, (user) => user.age < 30);
console.log("Người dùng trẻ:", youngUsers);

// Cập nhật người dùng
updateUser(users, users[0].id, { age: 26, name: "TG-Bao Updated" });
console.log("Sau khi cập nhật:", users[0]);
```

### 🎓 Bài tập thực hành

```javascript
// Bài tập 1: Hàm tính giai thừa
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// Bài tập 2: Hàm Fibonacci
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Bài tập 3: Hàm sắp xếp mảng
function sortArray(arr, ascending = true) {
  return arr.sort((a, b) => (ascending ? a - b : b - a));
}

// Bài tập 4: Hàm debounce
function debounce(func, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Sử dụng debounce
const debouncedSearch = debounce(function (query) {
  console.log("Tìm kiếm:", query);
}, 300);
```

## 🧠 Lý thuyết cốt lõi về Hàm trong JavaScript

### 1) Execution Context và Scope

- Mỗi khi hàm chạy, JavaScript tạo một Execution Context chứa: biến cục bộ, tham số, `this`, và môi trường lexical.
- Scope là “vùng nhìn thấy” của biến. JS có function scope và block scope (với `let/const`).
- Chuỗi scope (scope chain) quyết định cách JS tìm biến: bắt đầu từ scope hiện tại đi ra ngoài (lexical scoping).

```javascript
function outer() {
  const a = 1;
  function inner() {
    // nhìn thấy a nhờ lexical scoping
    return a + 1;
  }
  return inner();
}
```

### 2) Hoisting

- Khai báo `function` (declaration) được hoisted toàn bộ, có thể gọi trước khi định nghĩa.
- Biến `var` được hoisted nhưng khởi tạo là `undefined`. `let/const` có “Temporal Dead Zone” (TDZ) trước khi khai báo.

```javascript
greet(); // OK (declaration hoisted)
function greet() {
  console.log("Hi");
}

console.log(x); // undefined (var hoisted)
var x = 10;

// console.log(y); // ReferenceError (TDZ)
let y = 20;
```

### 3) Closures

- Closure là khả năng hàm “ghi nhớ” môi trường nơi nó được tạo ra, ngay cả khi môi trường đó đã kết thúc.
- Ứng dụng: module pattern, ẩn thông tin, tạo factory functions, debounce/throttle.

```javascript
function makeCounter() {
  let count = 0; // private
  return () => ++count;
}
const next = makeCounter();
next(); // 1
next(); // 2
```

### 4) `this` binding — 4 quy tắc cốt lõi

- Gọi hàm thông thường: `this` là `undefined` (strict mode) hoặc `window` (non-strict).
- Gọi như phương thức: `obj.method()`, `this` là `obj`.
- `call/apply/bind`: `this` là đối tượng bạn chỉ định.
- Arrow function: không có `this` riêng; “mượn” `this` từ scope bên ngoài (lexical this).

```javascript
const user = {
  name: "Bao",
  hi() {
    console.log(this.name);
  },
};
user.hi(); // Bao
const hi = user.hi;
// hi(); // undefined trong strict mode
hi.call(user); // Bao
```

### 5) Tư duy Functional Programming (FP) trong JS

- Pure function: không phụ thuộc/đổi trạng thái bên ngoài, cùng input → cùng output.
- Immutability: ưu tiên không đột biến dữ liệu; dùng copy/shallow copy thay vì sửa trực tiếp.
- Composition: ghép các hàm nhỏ thành pipeline.

```javascript
const addVAT = (x) => x * 1.1;
const round = (x) => Math.round(x * 100) / 100;
const finalPrice = (x) => round(addVAT(x));
```

### 6) Đệ quy, stack và hiệu năng

- Đệ quy giúp biểu đạt ngắn gọn (tree/graph), nhưng có rủi ro tràn stack.
- JS hiện đại hỗ trợ tối ưu tail-call hạn chế; thực tế nên chuyển sang vòng lặp khi cần.

```javascript
function sumIter(arr) {
  let s = 0;
  for (const n of arr) s += n;
  return s;
}
```

### 7) Hàm bất đồng bộ: callback, Promise, async/await

- Callback dễ rơi vào “callback hell” khi lồng nhau.
- Promise chuẩn hóa trạng thái và chuỗi then/catch.
- `async/await` giúp viết theo phong cách đồng bộ; dùng `try/catch` để bắt lỗi, `Promise.all` để song song.

```javascript
async function load() {
  try {
    const [a, b] = await Promise.all([fetchA(), fetchB()]);
    return combine(a, b);
  } catch (e) {
    return fallback(e);
  }
}
```

### 8) Xử lý lỗi trong hàm

- Với hàm sync: ném `Error` có ý nghĩa, tránh return mã lỗi mơ hồ.
- Với hàm async: luôn bao `try/catch`; hoặc gắn `.catch()` và chuẩn hóa thông điệp lỗi.
- Đừng nuốt lỗi; log đủ ngữ cảnh khi cần.

### 9) Kiểm thử hàm hiệu quả

- Hàm nhỏ, pure functions → test dễ và nhanh.
- Tách side-effects (I/O, DOM) ra khỏi core logic.
- Dùng dependency injection/mocking cho API/clock/random.

---

### 🎯 Kết luận

Hàm là nền tảng của JavaScript: từ cấu trúc chương trình, đóng gói logic, đến lập trình hàm và xử lý bất đồng bộ. Nắm vững execution context, scope/hoisting, closures, `this`, và các pattern FP sẽ giúp bạn viết code rõ ràng, dễ test và dễ mở rộng.

Hãy thực hành nhiều, nhưng luôn đặt lý thuyết lên trước: code chỉ là ví dụ minh họa cho tư duy đúng. 🚀
