# Node.js HTTP Module Guide

## 1. What is the `http` Module?

- Built-in core Node.js module (no need to install).
- Allows you to create servers, handle requests, and send responses.
- Works at a lower level than Express (manual handling of requests).
- 👉 Express.js is built on top of this http module to make life easier.

To use it:
```js
const http = require('http');
```

---

## 🚀 2. How to Create a Basic HTTP Server

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.write('Hello World!');
  res.end();
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

- `req` → incoming request object.
- `res` → outgoing response object.

---

## 📬 3. Handling GET Requests with `http`

```js
const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Welcome to Home Page');
  }
});
```

- `req.method` gives the HTTP method (GET, POST, etc.).
- `req.url` gives the requested URL (/, /about, etc.).

---

## 📝 4. Handling POST Requests with `http`

```js
const server = http.createServer((req, res) => {
  if (req.method === 'POST' && req.url === '/submit') {
    let body = '';

    // Collect incoming data
    req.on('data', chunk => {
      body += chunk.toString();
    });

    // Finished receiving data
    req.on('end', () => {
      console.log('Received body:', body);
      res.end('Data received');
    });
  }
});
```

- `req.on('data')` fires whenever a new chunk of body arrives.
- `req.on('end')` fires when all chunks are received.

---

## 🔥 5. How to Parse Query Params Manually

Example URL: `http://localhost:3000/user?id=123&name=John`

```js
const url = require('url');

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true); // 'true' gives query as object
  console.log(parsedUrl.query); // { id: '123', name: 'John' }

  res.end('Check console for query');
});
```

- `url.parse()` breaks the URL into parts like pathname, query, etc.

---

## 🛤️ 6. How to Handle Path Params Manually

In Node's native http, there are no path params by default. You need to manually extract them.

Example for `/user/123`:

```js
const server = http.createServer((req, res) => {
  const parts = req.url.split('/'); // ['', 'user', '123']

  if (req.method === 'GET' && parts[1] === 'user' && parts[2]) {
    const userId = parts[2];
    console.log('User ID:', userId);
    res.end(`User ID is ${userId}`);
  }
});
```

- Here, `parts[2]` is `123`.

---

## 📚 7. Full Example: GET + POST + Query + Params + Body Parsing

```js
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true); // parse URL
  const pathParts = parsedUrl.pathname.split('/'); // split path

  if (req.method === 'GET' && parsedUrl.pathname === '/') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Welcome Home!');
  }

  else if (req.method === 'GET' && pathParts[1] === 'user' && pathParts[2]) {
    const userId = pathParts[2];
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: `User ID: ${userId}` }));
  }

  else if (req.method === 'GET' && parsedUrl.pathname === '/search') {
    const query = parsedUrl.query;
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ query }));
  }

  else if (req.method === 'POST' && parsedUrl.pathname === '/submit') {
    let body = '';
    req.on('data', chunk => {
      body += chunk.toString();
    });
    req.on('end', () => {
      const data = JSON.parse(body); // parse JSON body
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ message: 'Data received', data }));
    });
  }

  else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

