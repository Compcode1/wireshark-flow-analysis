**Wireshark Network Flow Analysis Project: Full Summary & Methodology**

**🔹 Project Objective**
The goal of this project was to efficiently analyze how a website (Adidas.com) was accessed from the moment of the initial request to the establishment of an encrypted HTTPS session.

Instead of following traditional network analysis (which starts with DNS and moves forward), we optimized our workflow by:
✅ Starting with the first actual connection attempt (IP discovery)
✅ Working backward to find the DNS resolution that led to that IP
✅ Moving forward through TCP, TLS, and HTTPS to confirm how the session was established

This approach eliminated unnecessary manual tracking of irrelevant DNS queries and focused directly on the IP and its related traffic.

**🔹 Step-by-Step Summary of the Analysis**

1️⃣ Identifying the First Connection Attempt
📌 Filter Used:
tcp.flags.syn == 1 or udp.port == 443
🔹 This filter was applied to find the first actual network connection attempt (either via TCP SYN for HTTPS or UDP for QUIC).
🔹 The first result (Packet 1374) showed an IPv6 connection to www.adidas.com.

**📌 Key Takeaways:**
✅ The website was contacted over IPv6, meaning an AAAA record must have been returned.
✅ The connection used TCP over port 443, meaning Adidas was not using QUIC (HTTP/3).

2️⃣ Finding the Exact DNS Resolution
📌 Filter Used:

dns.aaaa == 2600:1406:2e00:28::17cb:a617
🔹 This filter jumped directly to the DNS response that provided the IPv6 address used in the connection.
🔹 Instead of manually searching for Adidas DNS queries, we worked backward from the IP itself.

**📌 Key Takeaways:**
✅ www.adidas.com did not resolve directly to an IP—it first went through a CNAME chain via Akamai’s CDN.
✅ The final resolved address was 2600:1406:2e00:28::17cb:a617, which matched the IP used in the TCP handshake.

**3️⃣ Moving Forward to Analyze the TLS & HTTPS Session**

📌 Filter Used:
ip.addr == 2600:1406:2e00:28::17cb:a617
🔹 This displayed all traffic between the system and Adidas.com, including:

The TCP handshake (confirming the three-way handshake).

The TLS handshake (Server Hello, Certificate Exchange, etc.).

The encrypted HTTPS data exchange.

📌 Key Takeaways:
✅ Adidas.com uses TLS over TCP, not QUIC.
✅ The full session was encrypted, confirming HTTPS was successfully established.
✅ No additional unexpected hosts were involved—everything was routed through Akamai’s CDN servers.

**🔹 Final Strategy for All Future Network Flow Analysis**
This method should be the standard for all future Wireshark-based network analysis:

🚀 The Optimized Three-Step Process
1️⃣ Start With the First Connection (Find the IP)

📌 Use this filter to find the first real connection attempt:

tcp.flags.syn == 1 or udp.port == 443

✅ This immediately tells us:
Whether the connection used IPv4 (A record) or IPv6 (AAAA record)
Whether it was TCP (TLS) or UDP (QUIC)

**2️⃣ Work Backward to Find the DNS Resolution**
📌 Once the IP is known, apply one of these filters to find its DNS response:
dns.aaaa == [IPv6 Address]
dns.a == [IPv4 Address]
✅ This instantly reveals the exact DNS resolution instead of manually tracking all queries.

**3️⃣ Move Forward Through TCP, TLS, and HTTPS**

📌 Use this filter to isolate all activity related to that IP:
ip.addr == [Resolved IP]

✅ This displays:
The TCP handshake (SYN, SYN-ACK, ACK)
The TLS handshake (Client Hello, Server Hello, Encryption Setup)
The actual encrypted HTTPS session

**🔹 Why This Approach Is Better Than Traditional Methods**
✅ Eliminates unnecessary manual tracking of multiple DNS queries.
✅ Directly finds the actual IP used in the connection instead of guessing.
✅ Works backward efficiently to find the exact DNS response.
✅ Moves forward through all network layers (TCP, TLS, HTTPS) without confusion.
✅ Drastically reduces time spent searching for packets in large captures.

**🔹 Final Confirmation: We Met All Project Objectives**
✅ We successfully identified how Adidas.com was accessed.
✅ We confirmed that IPv6 was used.
✅ We confirmed DNS resolution via Akamai’s CDN.
✅ We confirmed TCP was used (not QUIC).
✅ We identified the TLS handshake and encrypted session.
✅ We established a clear, repeatable methodology for future network flow analysis.

