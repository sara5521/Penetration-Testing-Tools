# 🌐 HTTP Requests Tools

This section covers tools that allow you to interact with web servers using HTTP/HTTPS requests. These are essential in both **network reconnaissance** and **web penetration testing** phases.

---

## 🎯 Purpose

These tools are used to:

* Interact with web servers
* Test API endpoints
* Download web content
* Analyze headers, cookies, and HTTP responses
* Send custom GET/POST requests

---

## 🛠️ Common HTTP Request Tools

| Tool         | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `curl`       | Versatile command-line tool to send HTTP requests            |
| `wget`       | Tool to download files and pages from HTTP/HTTPS/FTP servers |
| `httpie`     | User-friendly HTTP client with colored output                |
| `netcat`     | Can be used for manual HTTP requests via raw TCP             |
| `telnet`     | Older tool to manually send HTTP requests                    |
| `Burp Suite` | GUI tool for inspecting and modifying HTTP(S) traffic        |
| `Postman`    | GUI for testing RESTful APIs with custom HTTP requests       |

---

## 📋 curl Examples

### 🔎 Basic GET Request

```bash
curl http://demo.ine.local
```

### 📝 POST Request with Data

```bash
curl -X POST -d "user=admin&pass=123" http://demo.ine.local/login
```

### 🎭 Custom Headers

```bash
curl -H "User-Agent: Mozilla" http://demo.ine.local
```

### 💾 Save Output to File

```bash
curl http://demo.ine.local -o index.html
```

### 🔐 HTTPS Request Ignoring Certificate Warning

```bash
curl -k https://demo.ine.local
```

---

## 📋 wget Examples

### 📥 Download Webpage

```bash
wget http://demo.ine.local
```

### 📁 Mirror Entire Website

```bash
wget -m http://demo.ine.local
```

---

## 📋 httpie Examples

### 🔎 Simple GET

```bash
http http://demo.ine.local
```

### 📝 POST with JSON Data

```bash
http POST http://demo.ine.local/login user=admin pass=123
```

---

## 📋 netcat / telnet (Manual Requests)

### 🧪 Using Netcat

```bash
echo -e "GET / HTTP/1.1\r\nHost: demo.ine.local\r\n\r\n" | nc demo.ine.local 80
```

### 🧪 Using Telnet

```bash
telnet demo.ine.local 80
GET / HTTP/1.1
Host: demo.ine.local

```

---

## 🧠 Tips

* Use `-v` or `--verbose` in `curl` for debugging headers.
* Use `-A` in `wget` to specify a User-Agent.
* Combine `curl` with proxies like Burp using `--proxy`.

---

## 📚 Related Tools

* `Burp Suite`: Advanced GUI tool for testing and intercepting HTTP traffic
* `Postman`: Great for API testing

---

✅ **Use these tools to enumerate web services, identify vulnerabilities, and test how the server responds to various requests.**
