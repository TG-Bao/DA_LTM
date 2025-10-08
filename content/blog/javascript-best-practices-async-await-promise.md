---
title: "Best practices khi làm việc với async/await và Promise"
date: 2025-09-21T10:00:00+07:00
draft: false
tags: ["JavaScript", "Async", "Promise", "Best Practices", "Mẹo hay"]
category: "Tips"
---

# Best practices khi làm việc với async/await và Promise

Lập trình bất đồng bộ là một phần quan trọng trong JavaScript, đặc biệt khi phát triển ứng dụng web hiện đại. Với sự ra đời của Promise và sau đó là async/await, việc xử lý code bất đồng bộ đã trở nên dễ dàng hơn nhiều. Tuy nhiên, để sử dụng chúng hiệu quả, bạn cần tuân theo một số best practices. Bài viết này sẽ chia sẻ những kinh nghiệm quý báu khi làm việc với async/await và Promise.

## 1. Hiểu rõ cách Promise hoạt động

Trước khi đi sâu vào async/await, điều quan trọng là phải hiểu rõ cách Promise hoạt động, vì async/await chỉ là cú pháp "syntactic sugar" cho Promise.

```javascript
// Tạo một Promise
const myPromise = new Promise((resolve, reject) => {
  // Thực hiện công việc bất đồng bộ
  const success = true;
  
  if (success) {
    resolve('Operation completed successfully');
  } else {
    reject(new Error('Operation failed'));
  }
});

// Sử dụng Promise
myPromise
  .then(result => {
    console.log(result); // 'Operation completed successfully'
  })
  .catch(error => {
    console.error(error);
  });
```

### Promise states

Một Promise có thể ở một trong ba trạng thái:
- **Pending**: Trạng thái ban đầu, chưa hoàn thành hoặc bị từ chối
- **Fulfilled**: Hoạt động hoàn thành thành công
- **Rejected**: Hoạt động thất bại

## 2. Sử dụng async/await thay vì .then/.catch khi có thể

Async/await giúp code của bạn dễ đọc và dễ debug hơn, đặc biệt khi có nhiều thao tác bất đồng bộ liên tiếp.

```javascript
// Sử dụng .then/.catch
function getUserData(userId) {
  return fetchUser(userId)
    .then(user => {
      return fetchUserPosts(user.id)
        .then(posts => {
          return { user, posts };
        });
    })
    .catch(error => {
      console.error('Error fetching user data:', error);
    });
}

// Sử dụng async/await - dễ đọc hơn nhiều
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchUserPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Error fetching user data:', error);
  }
}
```

## 3. Xử lý lỗi đúng cách

Luôn bọc code async/await trong khối try-catch để xử lý lỗi một cách phù hợp.

```javascript
// Không tốt - không xử lý lỗi
async function fetchData() {
  const response = await fetch('https://api.example.com/data');
  const data = await response.json();
  return data;
}

// Tốt - xử lý lỗi đầy đủ
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error fetching data:', error);
    // Có thể xử lý lỗi thêm ở đây, ví dụ: hiển thị thông báo cho người dùng
    throw error; // Re-throw để caller có thể xử lý nếu cần
  }
}
```

## 4. Tránh await tuần tự khi các thao tác không phụ thuộc nhau

Nếu bạn có nhiều thao tác bất đồng bộ không phụ thuộc vào nhau, hãy thực hiện chúng song song để cải thiện hiệu suất.

```javascript
// Không tốt - thực hiện tuần tự
async function fetchAllData() {
  const users = await fetchUsers();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  
  return { users, posts, comments };
}

// Tốt - thực hiện song song
async function fetchAllData() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  
  return { users, posts, comments };
}
```

## 5. Xử lý lỗi với Promise.all

Khi sử dụng Promise.all, nếu một promise bị reject, toàn bộ Promise.all sẽ bị reject. Hãy xử lý trường hợp này một cách phù hợp.

```javascript
// Xử lý lỗi với Promise.all
async function fetchAllData() {
  try {
    const [users, posts, comments] = await Promise.all([
      fetchUsers(),
      fetchPosts(),
      fetchComments()
    ]);
    
    return { users, posts, comments };
  } catch (error) {
    console.error('One of the requests failed:', error);
    // Xử lý lỗi
  }
}

// Nếu bạn muốn tiếp tục ngay cả khi một số promise bị reject
async function fetchAllDataSafely() {
  const usersPromise = fetchUsers().catch(error => {
    console.error('Failed to fetch users:', error);
    return []; // Giá trị mặc định
  });
  
  const postsPromise = fetchPosts().catch(error => {
    console.error('Failed to fetch posts:', error);
    return []; // Giá trị mặc định
  });
  
  const commentsPromise = fetchComments().catch(error => {
    console.error('Failed to fetch comments:', error);
    return []; // Giá trị mặc định
  });
  
  const [users, posts, comments] = await Promise.all([
    usersPromise,
    postsPromise,
    commentsPromise
  ]);
  
  return { users, posts, comments };
}
```

## 6. Sử dụng Promise.allSettled khi cần

Từ ES2020, bạn có thể sử dụng Promise.allSettled để chờ tất cả các promise hoàn thành, bất kể thành công hay thất bại.

```javascript
async function fetchAllDataWithStatus() {
  const results = await Promise.allSettled([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  
  // Xử lý kết quả
  const users = results[0].status === 'fulfilled' ? results[0].value : [];
  const posts = results[1].status === 'fulfilled' ? results[1].value : [];
  const comments = results[2].status === 'fulfilled' ? results[2].value : [];
  
  return { users, posts, comments };
}
```

## 7. Tránh sử dụng async/await trong vòng lặp

Sử dụng async/await trong vòng lặp có thể dẫn đến hiệu suất kém vì các thao tác sẽ được thực hiện tuần tự.

```javascript
// Không tốt - thực hiện tuần tự
async function processItems(items) {
  const results = [];
  
  for (const item of items) {
    const result = await processItem(item); // Chờ từng item hoàn thành
    results.push(result);
  }
  
  return results;
}

// Tốt - thực hiện song song
async function processItems(items) {
  const promises = items.map(item => processItem(item));
  return Promise.all(promises);
}

// Nếu bạn cần giới hạn số lượng promise chạy đồng thời
async function processItemsWithConcurrencyLimit(items, concurrency = 5) {
  const results = [];
  const chunks = [];
  
  // Chia nhỏ mảng thành các chunk
  for (let i = 0; i < items.length; i += concurrency) {
    chunks.push(items.slice(i, i + concurrency));
  }
  
  // Xử lý từng chunk
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(item => processItem(item))
    );
    results.push(...chunkResults);
  }
  
  return results;
}
```

## 8. Sử dụng Promise timeout để tránh chờ đợi vô hạn

Đôi khi, một promise có thể không bao giờ resolve hoặc reject. Để tránh điều này, bạn có thể thêm timeout.

```javascript
function promiseWithTimeout(promise, timeoutMs) {
  // Tạo một promise sẽ reject sau timeoutMs
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Operation timed out after ${timeoutMs}ms`));
    }, timeoutMs);
  });
  
  // Trả về promise nào hoàn thành trước
  return Promise.race([promise, timeoutPromise]);
}

// Sử dụng
async function fetchWithTimeout() {
  try {
    const result = await promiseWithTimeout(
      fetch('https://api.example.com/data'),
      5000 // 5 giây timeout
    );
    return result.json();
  } catch (error) {
    console.error('Error or timeout:', error);
  }
}
```

## 9. Sử dụng async IIFE khi cần await ở top-level

Nếu bạn cần sử dụng await ở top-level (ngoài hàm async), bạn có thể sử dụng IIFE (Immediately Invoked Function Expression).

```javascript
// Không thể sử dụng await ở top-level (trước ES2022)
// await fetchData(); // Lỗi

// Sử dụng async IIFE
(async () => {
  try {
    const data = await fetchData();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
})();

// Từ ES2022, bạn có thể sử dụng top-level await trong module
// import { data } from './data.js';
// const result = await processData(data);
```

## 10. Tránh sử dụng async/await khi không cần thiết

Không phải mọi hàm đều cần async/await. Nếu bạn chỉ cần trả về một promise, không cần await, hãy trả về promise trực tiếp.

```javascript
// Không cần thiết
async function getData() {
  return await fetchData();
}

// Tốt hơn
async function getData() {
  return fetchData();
}

// Hoặc thậm chí tốt hơn nếu không cần xử lý gì thêm
function getData() {
  return fetchData();
}
```

## Kết luận

Async/await và Promise là những công cụ mạnh mẽ trong JavaScript hiện đại, giúp bạn xử lý code bất đồng bộ một cách dễ dàng và rõ ràng. Bằng cách tuân theo các best practices trên, bạn có thể viết code bất đồng bộ hiệu quả, dễ đọc và dễ bảo trì.

Hãy nhớ rằng, mặc dù async/await làm cho code trông giống như đồng bộ, nhưng bản chất của nó vẫn là bất đồng bộ. Hiểu rõ cách hoạt động của Promise và event loop trong JavaScript sẽ giúp bạn tránh được nhiều lỗi phổ biến khi làm việc với async/await.