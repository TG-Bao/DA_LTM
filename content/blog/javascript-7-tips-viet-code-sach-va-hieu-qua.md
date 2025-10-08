---
title: "7 tips viết JavaScript sạch và hiệu quả"
date: 2025-09-18T10:00:00+07:00
draft: false
tags: ["JavaScript", "Clean Code", "Best Practices", "Mẹo hay"]
category: "Tips"  



---

# 7 tips viết JavaScript sạch và hiệu quả

JavaScript là một ngôn ngữ linh hoạt và mạnh mẽ, nhưng điều đó cũng có thể dẫn đến code khó đọc và khó bảo trì nếu không được viết cẩn thận. Bài viết này sẽ chia sẻ 7 tips giúp bạn viết code JavaScript sạch và hiệu quả hơn.

## 1. Sử dụng const và let thay vì var

Từ ES6, JavaScript đã giới thiệu `const` và `let` để thay thế cho `var`. Việc sử dụng chúng giúp code của bạn rõ ràng và an toàn hơn.

```javascript
// Không tốt
var name = 'John';
var age = 30;
var isActive = true;

// Tốt
const name = 'John'; // Không thay đổi
let age = 30; // Có thể thay đổi
const isActive = true;
```

### Lợi ích:

- `const` đảm bảo biến không bị gán lại, giúp tránh các lỗi không mong muốn
- `let` và `const` có phạm vi block, không bị hoisting như `var`
- Code dễ đọc hơn vì bạn biết ngay biến nào có thể thay đổi và biến nào không

## 2. Sử dụng Arrow Function

Arrow function không chỉ giúp code ngắn gọn hơn mà còn giải quyết vấn đề về `this` trong JavaScript.

```javascript
// Không tốt
function add(a, b) {
  return a + b;
}

// Tốt
const add = (a, b) => a + b;

// Xử lý this
// Không tốt
const user = {
  name: 'John',
  sayHi: function() {
    setTimeout(function() {
      console.log(`Hi, I'm ${this.name}`); // this không trỏ đến user
    }, 1000);
  }
};

// Tốt
const user = {
  name: 'John',
  sayHi() {
    setTimeout(() => {
      console.log(`Hi, I'm ${this.name}`); // this trỏ đến user
    }, 1000);
  }
};
```

## 3. Sử dụng Template Literals

Template literals giúp việc nối chuỗi trở nên dễ dàng và dễ đọc hơn.

```javascript
// Không tốt
const greeting = 'Hello, ' + name + '! You are ' + age + ' years old.';

// Tốt
const greeting = `Hello, ${name}! You are ${age} years old.`;

// Nhiều dòng
const html = `
  <div>
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;
```

## 4. Destructuring - Phân rã cấu trúc

Destructuring giúp trích xuất dữ liệu từ arrays và objects một cách ngắn gọn và rõ ràng.

```javascript
// Object destructuring
// Không tốt
const firstName = user.firstName;
const lastName = user.lastName;
const email = user.email;

// Tốt
const { firstName, lastName, email } = user;

// Array destructuring
// Không tốt
const first = items[0];
const second = items[1];

// Tốt
const [first, second] = items;

// Với tham số hàm
// Không tốt
function createUser(userData) {
  const name = userData.name;
  const age = userData.age;
  // ...
}

// Tốt
function createUser({ name, age }) {
  // Sử dụng trực tiếp name và age
}
```

## 5. Tránh Global Scope

Biến global có thể dẫn đến xung đột tên và khó debug. Hãy đóng gói code của bạn trong modules hoặc IIFE (Immediately Invoked Function Expression).

```javascript
// Không tốt
let count = 0;
function increment() {
  count++;
}

// Tốt - Sử dụng module
// counter.js
export let count = 0;
export function increment() {
  count++;
}

// main.js
import { count, increment } from './counter.js';

// Tốt - Sử dụng IIFE
const counter = (function() {
  let count = 0;
  
  return {
    increment() {
      count++;
    },
    getCount() {
      return count;
    }
  };
})();

counter.increment();
console.log(counter.getCount());
```

## 6. Sử dụng Array và Object Methods

JavaScript cung cấp nhiều phương thức hữu ích để làm việc với arrays và objects, giúp code ngắn gọn và dễ đọc hơn.

```javascript
const numbers = [1, 2, 3, 4, 5];

// Không tốt
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}

// Tốt
const doubled = numbers.map(num => num * 2);

// Không tốt
let sum = 0;
for (let i = 0; i < numbers.length; i++) {
  sum += numbers[i];
}

// Tốt
const sum = numbers.reduce((total, num) => total + num, 0);

// Không tốt
const evens = [];
for (let i = 0; i < numbers.length; i++) {
  if (numbers[i] % 2 === 0) {
    evens.push(numbers[i]);
  }
}

// Tốt
const evens = numbers.filter(num => num % 2 === 0);

// Object methods
const user = { name: 'John', age: 30 };

// Lấy keys
const keys = Object.keys(user); // ['name', 'age']

// Lấy values
const values = Object.values(user); // ['John', 30]

// Lấy entries
const entries = Object.entries(user); // [['name', 'John'], ['age', 30]]

// Clone object
const userCopy = { ...user };
```

## 7. Xử lý lỗi đúng cách

Xử lý lỗi đúng cách giúp ứng dụng của bạn ổn định và dễ debug hơn.

```javascript
// Không tốt
function fetchData() {
  fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => {
      // Xử lý data
    });
}

// Tốt
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Có lỗi khi lấy dữ liệu:', error);
    // Xử lý lỗi phù hợp
  }
}

// Sử dụng Promise.catch
fetchData()
  .then(data => {
    // Xử lý data
  })
  .catch(error => {
    // Xử lý lỗi
  })
  .finally(() => {
    // Luôn thực hiện, bất kể thành công hay thất bại
  });
```

## Bonus: Một số tips khác

### Sử dụng default parameters

```javascript
// Không tốt
function greet(name) {
  name = name || 'Guest';
  return `Hello, ${name}!`;
}

// Tốt
function greet(name = 'Guest') {
  return `Hello, ${name}!`;
}
```

### Sử dụng optional chaining (ES2020)

```javascript
// Không tốt
const street = user && user.address && user.address.street;

// Tốt
const street = user?.address?.street;
```

### Sử dụng nullish coalescing (ES2020)

```javascript
// Không tốt - Coi 0 và '' là falsy
const count = data.count || 10;

// Tốt - Chỉ coi null và undefined là falsy
const count = data.count ?? 10;
```

## Kết luận

Viết JavaScript sạch và hiệu quả không chỉ giúp code của bạn dễ đọc hơn mà còn giảm thiểu bug và tăng khả năng bảo trì. Bằng cách áp dụng 7 tips trên, bạn sẽ cải thiện đáng kể chất lượng code JavaScript của mình. Hãy nhớ rằng, mục tiêu cuối cùng là viết code mà cả bạn và đồng nghiệp đều có thể dễ dàng hiểu và làm việc cùng.