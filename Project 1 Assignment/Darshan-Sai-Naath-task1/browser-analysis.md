# Browser Analysis

**Name:** B. Darshan Sai Naath  
**Roll Number:** 241CS114  

---

### 1. HTTP vs HTTPS Comparison

#### Observations

- When I opened [http://neverssl.com](http://neverssl.com), the Network tab showed two requests using the **HTTP** protocol.  
  (http://quietsoothinggrandpathway.neverssl.com)

  In the “Headers” section, the request URLs started with `http://`, and I could clearly read the full request and response contents as it had a HTML code and text.

- Whereas when I opened **https://github.com** and **https://youtube.com**, the Network tab showed multiple requests and the protocol column showed **HTTPS**.  

  The headers only revealed general metadata (host, user-agent, content-type), but the actual response body looked like encrypted or binary data. Basically, the readable content was hidden. (some of the responses failed to load, there were no resources within the given identifier found)

#### Comparison of Request Headers

| Feature | HTTP (neverssl.com) | HTTPS (github.com / youtube.com) |
|----------|---------------------|----------------------------------|
| **Protocol scheme** | http:// | https:// |
| **Port used** | 80 (Used for normal, unencrypted web traffic) | 443 (Used for secure, encrypted web traffic (TLS/SSL)) |
| **Security layer** | None | TLS/SSL encryption |
| **Headers visible** | All visible (Host, User-Agent, Cookies, etc.) | Only general headers visible; payload encrypted |
| **Response data** | Plain HTML, readable text | Encrypted, unreadable binary |
| **Vulnerability** | Susceptible to eavesdropping and modification | Confidential, authenticated, integrity-protected |

#### Can we see the actual data?

- **HTTP:** Yes, both the request headers and the response body are visible in plain text. Anyone using a sniffer like Wireshark can read passwords, cookies, or form data.
- **HTTPS:** No, only the handshake and encrypted packets are visible. The browser and server exchange keys during the **TLS handshake**, and all further data is encrypted using those session keys.

#### Why is HTTPS more secure?

- **Encryption:** Data is scrambled using cryptographic algorithms (e.g., AES) so only the intended server and client can read it.
- **Authentication:** The server provides a **digital certificate** issued by a trusted Certificate Authority (CA) to prove its identity.
- **Integrity:** TLS uses message authentication codes to detect if data is altered during transmission.

Therefore, **HTTPS is more secure** because even if someone captures the network traffic, the actual content (login info, page data, etc.) remains unreadable, while HTTP exposes everything in clear text.

---

### 2. Request Analysis

#### Observations

I opened the Network tab and reloaded each website while recording the number of requests, types of resources loaded, and total load times.

| Website | Total Requests | Resource Types | Notes |
|----------|----------------|----------------|-------|
| **neverssl.com** | ~8 (Here I had 2 requests) | HTML, CSS | A very small static site with only one HTML page and a basic stylesheet. |
| **github.com** | ~250 (Here I had 224 requests) | HTML, CSS, JS, Fonts, SVG images | GitHub is a large dynamic web application. It loads many JavaScript and CSS bundles required for its user interface and interactivity. |
| **youtube.com** | ~350+ (Here I had 560 requests) | HTML, JS, JSON, Images, Media (video previews), Fonts | YouTube loads the main interface, personalized scripts, API data, and several media files for video thumbnails and player scripts. |

#### Explanation

- Each website generates many **HTTP(S) requests** to fetch different resources (HTML pages, style sheets, images, JavaScript files, fonts, etc.).
- The browser renders a complete web page by combining all these components.
- **Static websites** like *neverssl.com* make fewer requests because everything is served in a single HTML + CSS file.
- **Dynamic and media-heavy sites** like *YouTube* make hundreds of requests due to asynchronous content loading, personalized data, analytics scripts, and auto-loaded video assets. The requests keep increasing as you scroll down through the particular website.  
- **GitHub** falls between both — it’s an interactive web app built using React, so it loads many bundled JS files and assets.

#### Which website made the most requests and why?

- **YouTube made the most requests.**  
  This is because it’s a highly dynamic application that continuously loads data through background API calls, ad services, and embedded video content.  
  Every image, video thumbnail, and script is a separate request, which leads to a much higher total count.

#### Insight

This comparison highlights how the **complexity and interactivity of a website** directly impact the number of network requests it makes.  
A decentralized VPN or any bandwidth-sensitive system must consider this, because more requests mean more overhead, longer page load times, and higher data usage — all of which affect network performance.

---

### 3. Performance Insights

#### Observations

Using the Network tab, I measured the total load time and examined the timing breakdown of one major request for each site.

| Website | Total Load Time (usually) | Longest Request | Type | Notes |
|----------|-----------------|-----------------|------|-------|
| **neverssl.com** | ~0.5 s (Here the Load time was 806 ns) | index.html | HTML | A single, simple page — loads almost instantly. |
| **github.com** | ~3.0 s (Here the Load time was 2.26 s) | main.js | JavaScript | Large JS bundle for interactive UI; most time spent on TTFB (server response delay). |
| **youtube.com** | ~5.0 s (Here the Load time was 3.03 s) | player.js / media preload | JS / Media | Loads heavy scripts and media elements asynchronously, plus background API calls. |

#### Example Detailed Timing (GitHub `main.js`)

| Phase | Time (ms) | Description |
|--------|------------|-------------|
| **DNS Lookup** | 25 ms | Browser contacted DNS to resolve `github.com` → IP address. |
| **TCP + TLS Handshake** | 80 ms | Secure HTTPS connection established between browser and server. |
| **Request Sent** | 2 ms | Browser sent the HTTP GET request to the server. |
| **Waiting (TTFB)** | 130 ms | Server processed the request and began sending data. |
| **Content Download** | 470 ms | The JavaScript file was downloaded from the server. |

#### Analysis

- **neverssl.com** finishes quickly because it is static and there are no complex scripts or media like images and videos.  
- **github.com** loads moderately fast but requires many script and style requests to render its React-based interface.  
- **youtube.com** is the slowest because it loads many large files (scripts, JSON API calls, ads, and media content) even before a video plays.

#### Understanding the Timing Phases

| Phase | What It Represents | Why It Matters |
|--------|--------------------|----------------|
| **DNS Lookup** | Translating domain name to IP address. | Adds latency if DNS is far or not cached. |
| **Connection Time** | Establishing TCP (and TLS for HTTPS) connection. | TLS increases security but adds slight setup delay. |
| **Waiting (TTFB)** | Server processing and response delay. | Reflects backend performance or server load. |
| **Content Download** | Time to transfer data from server to browser. | Affected by bandwidth and file size. |

#### Insights

- Websites with **more dynamic or media-heavy content** naturally make more requests and take longer to load.
- **TLS encryption** adds a small delay during the handshake but is essential for privacy and integrity.
- **TTFB (Time To First Byte)** is a strong indicator of server responsiveness — a shorter TTFB means faster backend performance.
- Understanding these phases is important for VPN or dVPN projects because every stage (DNS lookup, connection, transfer) can be affected by routing and encryption overhead.

---

## What I learnt new from this task

### 1. What is HTTP and HTTPS

- **HTTP (HyperText Transfer Protocol)** is the standard protocol used by browsers and servers to communicate and transfer web data like HTML pages, images, and videos.  
- It works on **port 80** and sends information as **plain text**, meaning anyone who intercepts the traffic can read it.  
- **HTTPS (HyperText Transfer Protocol Secure)** is the secure version of HTTP. It works on **port 443** and uses encryption (TLS) to protect the communication between browser and server.  
- In short, HTTPS = HTTP + encryption.

---

### 2. Difference Between HTTP and HTTPS

| Feature | HTTP | HTTPS |
|----------|------|--------|
| **Port** | 80 | 443 |
| **Security** | No encryption; data visible | Encrypted using TLS |
| **Data Confidentiality** | Anyone can read or modify traffic | Data is secure and private |
| **Authentication** | No identity verification | Uses SSL/TLS certificates to verify server identity |
| **Integrity** | Vulnerable to tampering | Ensures data integrity during transmission |
| **Use Case** | Public, non-sensitive sites | All modern secure websites (banking, logins, etc.) |

> I learnt that HTTPS doesn’t just hide data but it also proves that the website is genuine and prevents man-in-the-middle attacks.

---

### 3. What is TCP and IP

- **IP (Internet Protocol)** is responsible for **addressing and routing** packets across the internet and ensures data goes from your device’s IP to the destination IP.  
- **TCP (Transmission Control Protocol)** sits above IP and ensures **reliable delivery** of data:  
  - It splits data into packets.  
  - Numbers them in sequence.  
  - Re-sends any lost packets.  
  - Reassembles them correctly at the destination.  
- Together, they form the foundation of the Internet and are often called the **TCP/IP stack**.

> Example: When you open a website, IP finds the server’s location, and TCP guarantees your data reaches there correctly.

---

### 4. What is TLS Encryption and Handshake

- **TLS (Transport Layer Security)** is a cryptographic protocol that secures data between your browser and the web server.  
- During the **TLS handshake**, your browser and the server:  
  1. Exchange public keys and agree on encryption algorithms.  
  2. Verify the server’s **digital certificate** (issued by a trusted Certificate Authority).  
  3. Generate a **shared session key** used to encrypt all further communication.  

> The handshake ensures that only the browser and the intended server can decrypt the data.  
> Even if someone captures the packets (in Wireshark), they’ll only see scrambled binary data.

---

### 5. What is TTFB (Time To First Byte)

- **TTFB** stands for **Time To First Byte**. It’s the time between the browser sending a request and receiving the first byte of the response.  
- It measures how fast the server starts responding, not how fast the full page loads.  
- A high TTFB usually means the server is slow or under heavy load.  

**Formula:**  
> TTFB = DNS Lookup + TCP Connection + Server Processing Time  

> I learnt that even if a website has fast internet, a slow backend can still cause a large TTFB.  
> That’s why optimizing servers is as important as optimizing the network.

---

### Summary of My Learnings

From this task, I learnt how the browser actually communicates with servers. I have realised that with every click and every page load involves dozens of requests traveling through layers of TCP/IP, DNS, and TLS.  
This is really very fascinating knowing that such things occur behind the website invisibly that we just casually use every day.
I can now understand why HTTPS is essential, how encryption works, and what affects website performance (like DNS delay or TTFB).  
These insights directly connect to my project on decentralized VPNs, where data encryption, routing, and connection setup play the same crucial roles.

---
