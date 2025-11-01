# Vercel CORS Guide

## Giới thiệu

Hướng dẫn này cung cấp thông tin chi tiết về cách sử dụng Vercel và CORS (Cross-Origin Resource Sharing) để xây dựng các ứng dụng web hiện đại.

## Mục lục

1. [Vercel là gì?](#vercel-là-gì)
2. [Cách triển khai ứng dụng trên Vercel](#cách-triển-khai-ứng-dụng-trên-vercel)
3. [CORS là gì?](#cors-là-gì)
4. [Cấu hình CORS trên Vercel](#cấu-hình-cors-trên-vercel)
5. [Ví dụ thực tế](#ví-dụ-thực-tế)
6. [Troubleshooting](#troubleshooting)

## Vercel là gì?

Vercel là một nền tảng cloud hosting tối ưu hóa cho các ứng dụng web hiện đại. Nó cung cấp:

- **Deployment tự động** từ Git repositories
- **Serverless Functions** để chạy backend code
- **Edge Network** toàn cầu để tăng tốc độ
- **Automatic HTTPS** và bảo mật
- **Analytics** và monitoring

### Ưu điểm của Vercel

- Dễ sử dụng và thiết lập nhanh chóng
- Tích hợp tốt với Next.js
- Preview deployment cho mỗi pull request
- Hỗ trợ nhiều framework (React, Vue, Svelte, etc.)
- Chi phí thấp cho các dự án nhỏ

## Cách triển khai ứng dụng trên Vercel

### Bước 1: Chuẩn bị repository

1. Tạo một repository GitHub
2. Push code của bạn lên GitHub

### Bước 2: Kết nối với Vercel

1. Truy cập [vercel.com](https://vercel.com)
2. Đăng nhập hoặc tạo tài khoản
3. Nhấp vào "New Project"
4. Chọn repository GitHub của bạn
5. Cấu hình build settings (nếu cần)

### Bước 3: Deploy

Vercel sẽ tự động deploy khi bạn push code lên branch main.

```bash
# Đầu tiên, cài đặt Vercel CLI
npm i -g vercel

# Deploy trực tiếp từ terminal
vercel
```

## CORS là gì?

CORS (Cross-Origin Resource Sharing) là một cơ chế cho phép các ứng dụng web chạy trên một domain có thể yêu cầu tài nguyên từ một domain khác.

### Tại sao cần CORS?

- **Same-Origin Policy**: Trình duyệt web mặc định cấm các request từ domain A sang domain B
- **CORS** cho phép server chỉ định rõ ràng domain nào được phép truy cập

### CORS Headers

Các HTTP headers quan trọng:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 3600
```

## Cấu hình CORS trên Vercel

### Phương pháp 1: Sử dụng vercel.json

Tạo file `vercel.json` ở thư mục gốc của project:

```json
{
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, PUT, DELETE, PATCH, OPTIONS"
        },
        {
          "key": "Access-Control-Allow-Headers",
          "value": "Content-Type, Authorization"
        }
      ]
    }
  ]
}
```

### Phương pháp 2: Trong Serverless Function

Nếu sử dụng API routes (Next.js):

```javascript
// pages/api/data.js
export default function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') {
    res.status(200).end();
    return;
  }

  // Logic của API
  res.status(200).json({ message: 'Hello from Vercel!' });
}
```

### Phương pháp 3: Chỉ định domain cụ thể

Thay vì `*`, hãy chỉ định domain cụ thể:

```json
{
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "https://yourdomain.com"
        }
      ]
    }
  ]
}
```

## Ví dụ thực tế

### Ví dụ 1: Frontend React gọi API từ Vercel

**Frontend code:**

```javascript
import React, { useEffect, useState } from 'react';

function App() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('https://your-vercel-app.vercel.app/api/data')
      .then(res => res.json())
      .then(data => setData(data))
      .catch(err => console.error(err));
  }, []);

  return <div>{data && JSON.stringify(data)}</div>;
}

export default App;
```

**API Handler (Next.js):**

```javascript
export default function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', 'https://your-frontend.com');
  res.status(200).json({ data: 'Hello from API' });
}
```

### Ví dụ 2: Proxy request qua Vercel

```javascript
// pages/api/proxy.js
export default async function handler(req, res) {
  const { url } = req.query;

  try {
    const response = await fetch(url);
    const data = await response.json();

    res.setHeader('Access-Control-Allow-Origin', '*');
    res.status(200).json(data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Troubleshooting

### Lỗi: "Access to XMLHttpRequest has been blocked by CORS policy"

**Nguyên nhân:** Headers CORS không được cấu hình đúng

**Giải pháp:**
1. Kiểm tra file `vercel.json`
2. Đảm bảo API handler đã set headers
3. Xóa cache của trình duyệt
4. Restart development server

### Preflight request thất bại

**Nguyên nhân:** Server không xử lý OPTIONS request

**Giải pháp:**
```javascript
if (req.method === 'OPTIONS') {
  res.status(200).end();
}
```

### Domain bị từ chối

**Nguyên nhân:** Domain không được liệt kê trong Access-Control-Allow-Origin

**Giải pháp:** Thêm domain vào danh sách cho phép hoặc sử dụng `*`

## Best Practices

1. **Không sử dụng `*` trong production** - Hãy chỉ định domain cụ thể
2. **Sử dụng HTTPS** - Vercel tự động cung cấp SSL
3. **Validate requests** - Kiểm tra origin, method, headers
4. **Cache settings** - Sử dụng `Access-Control-Max-Age` để giảm preflight requests
5. **Credentials** - Nếu gửi cookies, đặt `Access-Control-Allow-Credentials: true`

## Tài liệu tham khảo

- [Vercel Documentation](https://vercel.com/docs)
- [MDN - CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [Next.js API Routes](https://nextjs.org/docs/api-routes/introduction)

## Contributor

Tạo bởi nhucuong010

## License

MIT License
