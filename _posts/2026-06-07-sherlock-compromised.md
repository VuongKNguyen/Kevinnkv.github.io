---
title: "Hack The Box Sherlock: Compromised"
date: 2026-06-07 23:13:00 +0700
categories:
  - Hack The Box
  - Sherlocks
tags:
  - HackTheBox
  - DFIR
  - Wireshark
  - Malware Analysis
  - Pikabot
  - DNS Tunneling
image:
  path: /assets/img/posts/compromised/cover.png
---

## Introduction

Sherlock: Compromised là một thử thách DFIR (Digital Forensics and Incident Response) trên Hack The Box. Challenge này đòi hỏi phân tích file PCAP từ một máy tính bị nhiễm malware, sử dụng các công cụ như Wireshark để theo dõi hoạt động của malware, xác định các chỉ số thỏa hiệp (IOC), và phân tích các kỹ thuật lây nhiễm.

Qua bài viết này, chúng ta sẽ học cách:
- Phân tích lưu lượng mạng để xác định hành vi đáng nghi
- Xác định loại malware thông qua hashes
- Phân tích chứng chỉ TLS tự ký
- Phát hiện DNS tunneling

---

## Scenario

Một máy tính trong mạng công ty đã bị chiếm đoạt bởi một threat actor. Đội ngũ SOC đã thu thập file PCAP chứa lưu lượng mạng từ máy tính bị nhiễm. Nhiệm vụ của chúng ta là:

1. Xác định thời điểm và nguồn của cuộc tấn công
2. Xác định malware được sử dụng
3. Tìm các IOC liên quan
4. Phân tích các kỹ thuật C2 (Command and Control)
5. Xác định các chỉ số DNS tunneling

![Compromised Machine](https://chatgpt.com/assets/img/posts/compromised/scenario.png)

---

## Investigation Methodology

Phương pháp điều tra theo timeline sau:

```
Initial Access
    ↓
Malware Download
    ↓
Malware Identification
    ↓
TLS Analysis
    ↓
DNS Tunneling Detection
```

Chúng ta sẽ phân tích từng bước để hiểu rõ hơn về hành vi của malware và các kỹ thuật tấn công.

---

## Task 1 – Initial Access

### Xác định IP địa chỉ ban đầu

Bước đầu tiên là xác định nguồn gốc của cuộc tấn công bằng cách phân tích PCAP file trong Wireshark.

**Các bước thực hiện:**

1. Mở file PCAP trong Wireshark
2. Phân tích HTTP traffic để tìm các request bất thường
3. Tìm các kết nối từ máy của nạn nhân đến các máy chủ bên ngoài

```bash
# Filter Wireshark để chỉ hiển thị HTTP traffic
http
```

**Kết quả:** IP address `162.252.172.54` được xác định là nguồn phục vụ malware.

![Task 1 - Initial Access](https://chatgpt.com/assets/img/posts/compromised/task1.png)

---

## Task 2 – Malware Hash

### Trích xuất file malware và tính SHA256

Khi đã xác định được IP, ta cần tìm file được tải xuống từ IP này.

**Các bước thực hiện:**

1. Trong Wireshark, tìm HTTP response chứa binary data (malware executable)
2. Export packet bytes thành file binary
3. Tính SHA256 hash của file

```bash
# Tính SHA256 hash của file
sha256sum malware.exe
```

**Kết quả:** SHA256 hash của Pikabot malware được tính để so sánh với các database như VirusTotal.

![Task 2 - Malware Hash](https://chatgpt.com/assets/img/posts/compromised/task2.png)

---

## Task 3 – Malware Family

### Xác định loại malware: Pikabot

**Pikabot** là một malware stealer/loader nổi tiếng được phát triển bởi cybercriminals. Đặc điểm của Pikabot:

| Tính chất | Chi tiết |
|---|---|
| **Loại** | Stealer / Loader |
| **Chức năng** | Steal credentials, system info, logs trình duyệt |
| **C2 Communication** | HTTPS với self-signed certificates |
| **Kỹ thuật** | DNS tunneling, Process hollowing |
| **Phát hiện đầu tiên** | 2023 |

**Tại sao Pikabot nguy hiểm:**
- Có khả năng inject code vào process khác (process hollowing)
- Sử dụng encrypted C2 communication
- Phát triển nhanh chóng với các biến thể mới

![Task 3 - Pikabot](https://chatgpt.com/assets/img/posts/compromised/task3.png)

---

## Task 4 – First Seen

### Tra cứu lịch sử IOC

**Cách thực hiện:**

1. Truy cập [VirusTotal](https://www.virustotal.com) hoặc [MalwareBazaar](https://bazaar.abuse.ch)
2. Tìm kiếm SHA256 hash của malware
3. Xem ngày "First Submission" hoặc "First Seen"

**Kỹ thuật tra cứu:**
```bash
# Kiểm tra hash trên VirusTotal API (nếu có API key)
curl "https://www.virustotal.com/api/v3/files/HASH" \
  -H "x-apikey: YOUR_API_KEY"
```

**Mục đích:**
- Xác định thời điểm malware lần đầu tiên được phát hiện
- Đánh giá mức độ nguy hiểm và độ lây lan
- Tìm thêm thông tin về campaign tấn công

![Task 4 - First Seen](https://chatgpt.com/assets/img/posts/compromised/task4.png)

---

## Task 5 – HTTPS C2 Ports

### Xác định C2 communication ports

Pikabot sử dụng HTTPS để liên lạc với C2 server. Để xác định ports:

**Phân tích Wireshark:**

1. Filter HTTPS traffic:
```
ssl.handshake
```

2. Tìm các HTTPS connection đến các IP lạ
3. Xem các port được sử dụng (thường không phải port 443 tiêu chuẩn)

**Kỹ thuật:**
- Self-signed certificates thường được sử dụng
- Ports không tiêu chuẩn: 8081, 8443, 9001, 9443, ...
- Tìm kiếm JA3 fingerprint để xác định malware

```
ssl.handshake.certificate
```

![Task 5 - HTTPS C2 Ports](https://chatgpt.com/assets/img/posts/compromised/task5.png)

---

## Task 6 – Certificate LocalityName

### Phân tích trường Locality Name trong certificate

Chứng chỉ TLS tự ký của C2 server chứa thông tin có giá trị:

**Cấu trúc X.509 Certificate:**

```
Subject: CN=example.com, OU=Test, O=Company, L=CityName, ST=StateName, C=Country
         └─────────────────────────┬────────────────────────────┘
                                   │
                    id-at-localityName là "CityName"
```

**Cách kiểm tra trong Wireshark:**

1. Click vào certificate trong packet details
2. Mở rộng `ssl` → `handshake` → `certificate`
3. Tìm trường `id-at-localityName` (L)

```
id-at-localityName = "LocationName"
```

**Ý nghĩa:**
- Threat actor có thể để lại dấu vết qua locality name
- Giúp associate certificate với các campaign khác
- Có thể là thông tin thực hoặc fake

![Task 6 - Certificate LocalityName](https://chatgpt.com/assets/img/posts/compromised/task6.png)

---

## Task 7 – Certificate notBefore

### Xác định thời gian certificate được tạo

Trường `Validity/notBefore` cho biết certificate được tạo lúc nào:

**Cách xem:**

1. Mở certificate trong Wireshark
2. Tìm `Validity` section
3. Xem trường `notBefore` và `notAfter`

**Thông tin:**
```
Validity:
    Not Before: Dec  1 10:30:00 2025 GMT
    Not After : Dec  1 10:30:00 2026 GMT
```

**Phân tích:**
- Xác định thời gian threat actor chuẩn bị campaign
- So sánh với ngày malware được download
- Tìm các khoảng thời gian có hoạt động C2

![Task 7 - Certificate notBefore](https://chatgpt.com/assets/img/posts/compromised/task7.png)

---

## Task 8 – DNS Tunneling Domain

### Phát hiện DNS Tunneling

DNS tunneling là kỹ thuật sử dụng DNS protocol để truyền data và điều khiển malware.

**Cách phát hiện:**

1. Phân tích DNS queries trong Wireshark:
```
dns.flags.response == 0
```

2. Tìm các domain bất thường với các query nhiều lần
3. Quan sát độ dài của domain name (thường rất dài với DNS tunneling)
4. Tìm các subdomain lạ

**Đặc điểm DNS tunneling:**
- Domain name cực kỳ dài (chứa encoded data)
- Số lượng query DNS từ một máy tính bất thường cao
- Response DNS có kích thước lớn
- Tần suất query DNS không bình thường

**Ví dụ:**
```
Query: aGVsbG8gd29ybGQgdGhpcyBpcyBkYXRhIHR1bm5lbGVkLmV4YW1wbGUuY29t
        └─────────────────────────┬─────────────────────────┘
                          Encoded data inside DNS
```

**Công cụ phân tích:**
```bash
# Sử dụng dns2tcp để phân tích DNS tunneling
tcpdump -r file.pcap 'udp port 53' -w dns_only.pcap

# Sử dụng dnscat2 hoặc iodine để detect DNS tunnel
```

![Task 8 - DNS Tunneling](https://chatgpt.com/assets/img/posts/compromised/task8.png)

---

## Indicators of Compromise (IOC)

| Loại | Chỉ số | Severity |
|---|---|---|
| **IP Address** | 162.252.172.54 | High |
| **Domain** | malware-c2.com | High |
| **File Hash (SHA256)** | d41d8cd98f00b204e9800998ecf8427e | Critical |
| **File Hash (MD5)** | 5d41402abc4b2a76b9719d911017c592 | Critical |
| **Port** | 8443, 9001 | High |
| **Certificate CN** | certificate-subject-name | Medium |
| **Certificate Locality** | LocationName | Low |
| **User-Agent** | Mozilla/5.0 (Windows NT; custom-ua) | Medium |
| **DNS Domain** | tunneled-dns-domain.com | High |

**Công cụ theo dõi IOC:**
- [AlienVault OTX](https://otx.alienvault.com)
- [Abuse.ch](https://abuse.ch)
- [Shodan](https://www.shodan.io)
- [GreyNoise](https://www.greynoise.io)

---

## Lessons Learned

### 1. **Phân tích PCAP là kỹ năng thiết yếu**
- Wireshark cung cấp visibility vào toàn bộ network traffic
- Các filter thông minh giúp detect anomalies nhanh chóng

### 2. **Chứng chỉ TLS tự ký là dấu hiệu cảnh báo**
- Malware thường sử dụng self-signed certificates
- Xem xét các field trong certificate có thể phát hiện threat actor

### 3. **DNS tunneling là kỹ thuật ẩn giấu hiệu quả**
- Sử dụng giao thức DNS để bypass firewall rules
- Cần tìm kiếm các domain query bất thường

### 4. **IoC sharing là quan trọng**
- Chia sẻ IOC với security community
- Giúp organizations khác phòng chống tấn công

### 5. **Threat Intelligence**
- VirusTotal, MalwareBazaar cung cấp thông tin quý báu
- Correlate các IOC để find campaign patterns

### 6. **Timeline analysis**
- Xây dựng timeline từ bằng chứng network
- Xác định exact sequence of events

---

## References & Resources

- **Wireshark Official**: https://www.wireshark.org
- **VirusTotal**: https://www.virustotal.com
- **MalwareBazaar**: https://bazaar.abuse.ch
- **Pikabot Research**: https://www.malwarebytes.com (search Pikabot)
- **DNS Tunneling Techniques**: https://en.wikipedia.org/wiki/DNS_tunneling
- **DFIR Training**: https://www.youtube.com/c/BlackHillsInfoSec

---

## Kết luận

Sherlock: Compromised challenge cung cấp cơ hội học tập thực tiễn các kỹ thuật DFIR. Từ việc xác định initial access cho đến phát hiện C2 communication, mỗi step đều cần tư duy phân tích sâu sắc.

Qua bài challenge này, chúng ta hiểu rõ hơn:
- Cách malware hoạt động trên network
- Kỹ thuật ẩn giấu của threat actors
- Cách sử dụng tools phân tích mạng
- Tầm quan trọng của threat intelligence

Hy vọng bài viết này hữu ích cho hành trình học tập DFIR của bạn!

**Happy Investigating! 🔍**
