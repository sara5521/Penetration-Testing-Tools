
# 🛰️ TCP vs UDP

This file explains the difference between the two main transport layer protocols: **TCP** and **UDP**.

---

## 🔹 TCP (Transmission Control Protocol)

| Feature                    | Description |
|----------------------------|-------------|
| ✅ Reliable                | Ensures delivery with acknowledgments (ACKs). Retransmits if packets are lost. |
| 🔄 Connection-oriented     | Requires a 3-way handshake before sending data. |
| 🧱 Ordered delivery        | Delivers packets in the same order they were sent. |
| 🛡️ Suitable for critical data | Used in web browsing (HTTP/HTTPS), emails (SMTP), file transfers (FTP), etc. |
| 🐢 Slower than UDP         | Due to overhead of reliability and connection management. |

---

## 🔹 UDP (User Datagram Protocol)

| Feature                    | Description |
|----------------------------|-------------|
| ❌ Unreliable             | Sends data without checking if it was received. No retransmissions. |
| 🚫 Connectionless         | Does not require a handshake to start communication. |
| 🔀 Unordered delivery     | Packets may arrive out of order or not at all. |
| ⚡ Very fast              | Lightweight protocol with no delivery guarantees. |
| 🎮 Used for real-time apps | Ideal for streaming, VoIP, and online games. |

---

## 🧠 Real-Life Analogy

- **TCP**: Like sending a registered letter — the recipient must sign for it, and you get confirmation.
- **UDP**: Like shouting across a room — fast, but you’re not sure if they heard you.

