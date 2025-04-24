# Handling Incoming JSON Without `express.json()` in Node.js

When you send a POST request like:
```json
{ "name": "John", "age": 30 }
```

If you **don't** use `express.json()`, in Node.js the `req` object **does not** have `req.body` ready. Instead:

- The body arrives in **chunks** as binary buffer data.
- You must manually listen to `req.on('data')` events to collect it.

Example without `express.json()`:
```javascript
app.post('/user', (req, res) => {
  req.on('data', chunk => {
    console.log(chunk);
    // Output: <Buffer 7b 22 6e 61 6d 65 22 3a 22 4a 6f 68 6e 22 2c 22 61 67 65 22 3a 33 30 7d>
  });

  req.on('end', () => {
    res.send('Raw data received');
  });
});
```

This `<Buffer>` is a series of hexadecimal bytes (raw data). You **cannot directly work** with it unless you **manually convert it**.

Manual Parsing:
```javascript
let body = '';
req.on('data', chunk => {
  body += chunk.toString(); // Convert Buffer to string
});
req.on('end', () => {
  const parsed = JSON.parse(body); // Finally parse into JavaScript object
  console.log(parsed); // { name: 'John', age: 30 }
});
```

### Without Parsing | What Happens
| Request Type | Comes As |
| :----------- | :------- |
| JSON request | Buffer (binary data) |
| Form URL-encoded request | Raw string like `name=John&age=30` |
| Files (multipart) | Complex multipart boundary text |

> **Overall**: You must manually assemble and parse it.

## 👉 Conclusion
Without using parsers like `express.json()`, `express.urlencoded()`, or `multer`, you have to manually read raw data, assemble it, and decode it. **Painful and error-prone!** 😵

---

# Built-in Parsers in Express

## 1. `express.json()`

✅ **What it does**:
- Parses incoming requests where the `Content-Type` is `application/json`.
- Converts the raw JSON payload into a JavaScript object.
- Access parsed body via `req.body`.

✅ **Why needed**:
- When a client sends `{ "name": "John", "age": 30 }`, the server gets it as a raw JSON string.
- `express.json()` parses it into `{ name: 'John', age: 30 }`.

✅ **Example**:
```javascript
const express = require('express');
const app = express();

app.use(express.json()); // <-- parses JSON bodies

app.post('/user', (req, res) => {
  console.log(req.body); // { name: 'John', age: 30 }
  res.send('User received');
});
```

---

## 2. `express.urlencoded()`

✅ **What it does**:
- Parses requests with `Content-Type: application/x-www-form-urlencoded`.
- Usually used when submitting HTML form data.

✅ **Why needed**:
- Browsers send form data like `name=John&age=30`.
- Needs parsing into `{ name: 'John', age: 30 }`.

✅ **Options**:
- `{ extended: true }` uses `qs` library for parsing nested objects.
- `{ extended: false }` supports simple key-value pairs only.

✅ **Example**:
```javascript
app.use(express.urlencoded({ extended: true }));

app.post('/form', (req, res) => {
  console.log(req.body); // { name: 'John', age: '30' }
  res.send('Form data received');
});
```

---

## 3. `multer`

✅ **What it does**:
- Handles file uploads.
- Parses `multipart/form-data`.

✅ **Why needed**:
- Normal parsers (`express.json()`, `express.urlencoded()`) **cannot handle files**.
- `multer` saves uploaded files to disk, memory, or elsewhere.

✅ **Example**:
```javascript
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
  console.log(req.file); // file info: filename, path, size, etc.
  res.send('File uploaded');
});
```
- `upload.single('file')` expects a single file named `file` from the form.

✅ **Bonus**:
- `upload.array('photos', 12)` — multiple files.
- `upload.fields([{ name: 'avatar' }, { name: 'gallery' }])` — multiple fields.

---

## 4. `body-parser` (older)

✅ **What it does**:
- Originally a separate library to parse JSON and URL-encoded data.
- Now **built into Express** since version 4.16.0+.

✅ **History**:
- Before Express 4.16.0, needed separate installation:
```bash
npm install body-parser
```

✅ **Example (old way)**:
```javascript
const bodyParser = require('body-parser');

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
```

✅ **Modern way**:
```javascript
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

> ✨ **Summary**: `body-parser` is now built-in. Only needed separately for old projects.

