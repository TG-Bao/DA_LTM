---
title: "Mẹo tối ưu performance trong JavaScript"
date: 2025-09-25T10:00:00+07:00
draft: false
tags: ["JavaScript", "Performance", "Optimization", "Mẹo hay"]
category: "Tips"
---

# Mẹo tối ưu performance trong JavaScript

Hiệu suất là một yếu tố quan trọng trong phát triển ứng dụng web hiện đại. Với JavaScript đóng vai trò ngày càng lớn trong các ứng dụng web, việc tối ưu hiệu suất JavaScript trở nên cực kỳ quan trọng. Bài viết này sẽ chia sẻ những mẹo giúp tối ưu hiệu suất trong JavaScript.

## 1. Tối ưu DOM Manipulation

Thao tác với DOM là một trong những hoạt động tốn kém nhất trong JavaScript. Mỗi lần bạn truy cập hoặc thay đổi DOM, trình duyệt phải tính toán lại layout, repaint và reflow.

### Giảm thiểu DOM access

```javascript
// Không tốt
for (let i = 0; i < 1000; i++) {
  document.getElementById('result').innerHTML += `<div>${i}</div>`;
}

// Tốt
let html = '';
for (let i = 0; i < 1000; i++) {
  html += `<div>${i}</div>`;
}
document.getElementById('result').innerHTML = html;
```

### Sử dụng DocumentFragment

```javascript
// Tốt hơn nữa - sử dụng DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = i;
  fragment.appendChild(div);
}
document.getElementById('result').appendChild(fragment);
```

### Tránh forced reflow

```javascript
// Không tốt - gây forced reflow
const element = document.getElementById('box');
element.style.width = '100px';
console.log(element.offsetWidth); // Gây reflow
element.style.height = '100px';
console.log(element.offsetHeight); // Gây reflow lần nữa

// Tốt - gom nhóm các thay đổi
const element = document.getElementById('box');
element.style.width = '100px';
element.style.height = '100px';
// Đọc sau khi đã thay đổi xong
console.log(element.offsetWidth);
console.log(element.offsetHeight);
```

## 2. Tối ưu vòng lặp

Vòng lặp là nơi code JavaScript thường dành nhiều thời gian thực thi nhất, đặc biệt khi làm việc với mảng lớn.

### Cache độ dài mảng

```javascript
// Không tốt
for (let i = 0; i < array.length; i++) {
  // array.length được tính lại mỗi lần lặp
}

// Tốt
const len = array.length;
for (let i = 0; i < len; i++) {
  // Độ dài được cache
}
```

### Sử dụng các phương thức mảng tích hợp

```javascript
// Tốt - sử dụng các phương thức mảng tích hợp
const result = array.filter(item => item > 10).map(item => item * 2);

// Tốt hơn - giảm số lần duyệt mảng
const result = array.reduce((acc, item) => {
  if (item > 10) {
    acc.push(item * 2);
  }
  return acc;
}, []);
```

### Sử dụng for...of thay vì forEach

```javascript
// forEach có overhead của function call mỗi lần lặp
array.forEach(item => {
  // Xử lý item
});

// for...of thường nhanh hơn với mảng lớn
for (const item of array) {
  // Xử lý item
}
```

## 3. Quản lý bộ nhớ hiệu quả

Rò rỉ bộ nhớ là một vấn đề phổ biến trong các ứng dụng JavaScript, đặc biệt là các ứng dụng SPA (Single Page Application).

### Xóa event listeners khi không cần thiết

```javascript
function setupListener() {
  const button = document.getElementById('myButton');
  const handler = () => {
    // Xử lý sự kiện
  };
  
  button.addEventListener('click', handler);
  
  // Trả về hàm để xóa listener khi không cần nữa
  return () => {
    button.removeEventListener('click', handler);
  };
}

const cleanup = setupListener();

// Khi component bị hủy hoặc không cần nữa
cleanup();
```

### Tránh closure không cần thiết

```javascript
// Không tốt - giữ tham chiếu đến bigData trong closure
function createProcessor() {
  const bigData = new Array(1000000).fill('some data');
  
  return function process() {
    return bigData.length;
  };
}

// Tốt - chỉ giữ những gì cần thiết
function createProcessor() {
  const bigDataLength = new Array(1000000).fill('some data').length;
  
  return function process() {
    return bigDataLength;
  };
}
```

### Sử dụng WeakMap và WeakSet

```javascript
// WeakMap và WeakSet cho phép GC thu hồi bộ nhớ khi key không còn tham chiếu
const cache = new WeakMap();

function processObject(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  
  const result = expensiveOperation(obj);
  cache.set(obj, result);
  return result;
}
```

## 4. Tối ưu JavaScript loading

Cách bạn tải JavaScript có thể ảnh hưởng lớn đến thời gian tải trang.

### Sử dụng defer và async

```html
<!-- Tải script nhưng không chặn parsing HTML -->
<script src="analytics.js" async></script>

<!-- Tải script sau khi parse HTML, thực thi trước DOMContentLoaded -->
<script src="app.js" defer></script>
```

### Code splitting

Chia nhỏ bundle JavaScript thành các chunk nhỏ hơn và chỉ tải khi cần thiết.

```javascript
// Sử dụng dynamic import (ES2020)
button.addEventListener('click', async () => {
  const module = await import('./feature.js');
  module.doSomething();
});
```

### Preload critical resources

```html
<link rel="preload" href="critical.js" as="script">
<link rel="preload" href="critical.css" as="style">
```

## 5. Tối ưu thuật toán và cấu trúc dữ liệu

Lựa chọn thuật toán và cấu trúc dữ liệu phù hợp có thể cải thiện hiệu suất đáng kể.

### Sử dụng Map thay vì Object cho lookup tables

```javascript
// Không tốt - Object không được tối ưu cho việc thêm/xóa thường xuyên
const userCache = {};
userCache[userId] = userData;

// Tốt - Map được tối ưu cho việc thêm/xóa và tra cứu
const userCache = new Map();
userCache.set(userId, userData);
```

### Sử dụng Set cho tìm kiếm nhanh

```javascript
// Không tốt - O(n)
const uniqueIds = [1, 2, 3, 4, 5];
if (uniqueIds.includes(searchId)) {
  // Tìm thấy
}

// Tốt - O(1)
const uniqueIds = new Set([1, 2, 3, 4, 5]);
if (uniqueIds.has(searchId)) {
  // Tìm thấy
}
```

### Tránh thuật toán O(n²)

```javascript
// Không tốt - O(n²)
function findDuplicates(array) {
  const duplicates = [];
  
  for (let i = 0; i < array.length; i++) {
    for (let j = i + 1; j < array.length; j++) {
      if (array[i] === array[j] && !duplicates.includes(array[i])) {
        duplicates.push(array[i]);
      }
    }
  }
  
  return duplicates;
}

// Tốt - O(n)
function findDuplicates(array) {
  const seen = new Set();
  const duplicates = new Set();
  
  for (const item of array) {
    if (seen.has(item)) {
      duplicates.add(item);
    } else {
      seen.add(item);
    }
  }
  
  return [...duplicates];
}
```

## 6. Tối ưu rendering

### Sử dụng requestAnimationFrame cho animation

```javascript
// Không tốt
function animate() {
  element.style.left = `${position++}px`;
  setTimeout(animate, 16); // ~60fps
}

// Tốt - đồng bộ với refresh rate của màn hình
function animate() {
  element.style.left = `${position++}px`;
  requestAnimationFrame(animate);
}
```

### Tránh layout thrashing

```javascript
// Không tốt - gây layout thrashing
function updateLayout() {
  for (const element of elements) {
    const height = element.offsetHeight; // Đọc
    element.style.height = `${height * 2}px`; // Viết
    // Mỗi lần lặp gây ra một reflow
  }
}

// Tốt - tách đọc và viết
function updateLayout() {
  // Đọc trước
  const heights = elements.map(element => element.offsetHeight);
  
  // Viết sau
  elements.forEach((element, i) => {
    element.style.height = `${heights[i] * 2}px`;
  });
}
```

### Sử dụng CSS transforms thay vì thay đổi position

```javascript
// Không tốt - gây reflow
element.style.left = `${x}px`;
element.style.top = `${y}px`;

// Tốt - chỉ gây repaint, không gây reflow
element.style.transform = `translate(${x}px, ${y}px)`;
```

## 7. Tối ưu network requests

### Sử dụng debounce và throttle

```javascript
// Debounce - chỉ thực hiện sau khi ngừng sự kiện một khoảng thời gian
function debounce(func, wait) {
  let timeout;
  
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

// Throttle - giới hạn tần suất thực hiện
function throttle(func, limit) {
  let inThrottle;
  
  return function executedFunction(...args) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Sử dụng
const debouncedSearch = debounce(searchAPI, 300);
searchInput.addEventListener('input', debouncedSearch);

const throttledScroll = throttle(handleScroll, 100);
window.addEventListener('scroll', throttledScroll);
```

### Sử dụng memoization

```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Sử dụng
const expensiveCalculation = memoize((a, b) => {
  console.log('Calculating...');
  return a * b;
});

console.log(expensiveCalculation(4, 2)); // Tính toán và cache
console.log(expensiveCalculation(4, 2)); // Lấy từ cache
```

## 8. Tối ưu với Web Workers

Web Workers cho phép bạn chạy JavaScript trong một luồng riêng biệt, không chặn UI thread.

```javascript
// main.js
const worker = new Worker('worker.js');

worker.onmessage = function(e) {
  console.log('Kết quả từ worker:', e.data);
};

worker.postMessage({
  data: largeArray,
  operation: 'process'
});

// worker.js
self.onmessage = function(e) {
  const { data, operation } = e.data;
  
  if (operation === 'process') {
    // Xử lý dữ liệu tốn nhiều CPU
    const result = processData(data);
    self.postMessage(result);
  }
};

function processData(data) {
  // Xử lý phức tạp không làm đơ UI
  return data.map(x => x * x).filter(x => x > 10);
}
```

## 9. Sử dụng công cụ đo lường hiệu suất

### Chrome DevTools Performance tab

Chrome DevTools cung cấp các công cụ mạnh mẽ để phân tích hiệu suất JavaScript:

- **Performance tab**: Ghi lại và phân tích hoạt động của trang
- **Memory tab**: Phân tích sử dụng bộ nhớ và phát hiện rò rỉ
- **Lighthouse**: Đánh giá tổng thể hiệu suất trang

### Sử dụng console.time

```javascript
console.time('operationName');
// Code cần đo thời gian
console.timeEnd('operationName'); // Hiển thị thời gian thực thi
```

## 10. Tối ưu bundle size

### Tree shaking

Đảm bảo bundler của bạn (như Webpack, Rollup) được cấu hình để loại bỏ code không sử dụng.

```javascript
// Sử dụng ES modules để hỗ trợ tree shaking
import { usedFunction } from 'library';
// unusedFunction sẽ không được đưa vào bundle
```

### Code splitting dựa trên routes

```javascript
// React với React Router và lazy loading
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Route, Switch } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Switch>
          <Route exact path="/" component={Home} />
          <Route path="/about" component={About} />
          <Route path="/dashboard" component={Dashboard} />
        </Switch>
      </Suspense>
    </BrowserRouter>
  );
}
```

## Kết luận

Tối ưu hiệu suất JavaScript là một quá trình liên tục và cần được tích hợp vào quy trình phát triển của bạn. Bằng cách áp dụng các mẹo trên, bạn có thể cải thiện đáng kể hiệu suất ứng dụng JavaScript của mình.

Hãy nhớ rằng, tối ưu sớm có thể là nguồn gốc của nhiều vấn đề. Luôn đo lường trước, tối ưu sau, và tập trung vào những điểm nóng thực sự ảnh hưởng đến trải nghiệm người dùng.