# V0X Security Blog

[![Jekyll](https://img.shields.io/badge/Jekyll-4.x-blue?style=flat-square)](https://jekyllrb.com)&nbsp;
[![Chirpy Theme](https://img.shields.io/badge/Theme-Chirpy-brightgreen?style=flat-square)](https://github.com/cotes2020/jekyll-theme-chirpy)&nbsp;
[![License](https://img.shields.io/github/license/Kevinnkv/Kevinnkv.github.io?style=flat-square)](LICENSE)

🔒 **An toàn thông tin · HackTheBox Write-ups · CTF Challenges**

Blog chia sẻ kiến thức An toàn thông tin, write-up HackTheBox, CTF solutions và công cụ pentest.

**Live:** [https://Kevinnkv.github.io](https://Kevinnkv.github.io)

---

## 📋 Nội dung Blog

### 🎯 Chủ đề Chính
- **HackTheBox Write-ups** - Phân tích chi tiết các machine sau khi retire
- **CTF Write-ups** - Lời giải các challenge từ các giải CTF
- **ATTT Fundamentals** - Kiến thức nền tảng về bảo mật
- **Web Security** - SQLi, XSS, SSRF, IDOR và các lỗ hổng web phổ biến
- **Privilege Escalation** - Kỹ thuật leo thang đặc quyền Linux/Windows
- **Tools & Scripts** - Công cụ và script tự viết hỗ trợ pentest

### 🛠️ Tech Stack
| Category | Tools |
|---|---|
| Recon | Nmap, Gobuster, Ffuf, Amass |
| Exploitation | Metasploit, BurpSuite, SQLmap |
| Post-Exploitation | LinPEAS, WinPEAS, BloodHound |
| Languages | Python, Bash, PowerShell |

---

## 🚀 Cách Sử Dụng

### Cài Đặt
```bash
git clone https://github.com/Kevinnkv/Kevinnkv.github.io.git
cd Kevinnkv.github.io
bundle install
```

### Chạy Locally
```bash
bundle exec jekyll serve
```
Truy cập: `http://localhost:4000`

### Thêm Bài Post Mới
Tạo file trong `_posts/` theo format:
```
_posts/YYYY-MM-DD-tieu-de.md
```

---

## 📁 Cấu Trúc Thư Mục
```
.
├── _config.yml          # Cấu hình Jekyll
├── _posts/              # Bài viết blog
├── _tabs/               # Trang About, Archives, Categories, Tags
├── assets/              # Ảnh, CSS, JS
│   ├── img/
│   │   └── avatar.svg   # Avatar (sidebar)
│   └── lib/             # Thư viện ngoài
├── README.md            # File này
└── index.html           # Trang chủ
```

---

## 📝 Viết Bài

Ví dụ Front Matter cho bài post:
```yaml
---
title: HackTheBox - Meerkat Write-up
date: 2026-05-10 20:00:00 +0700
categories: [HackTheBox, Sherlock]
tags: [forensics, json, jq, blue-team]
description: "Phân tích forensics với file JSON"
---
```

---

## 🔗 Liên Kết
- **GitHub:** [Kevinnkv](https://github.com/Kevinnkv)
- **HackTheBox:** [@Kevinnkv](https://app.hackthebox.com/profile/xxxx)
- **Blog:** [V0X Security](https://Kevinnkv.github.io)

---

## 📄 License

Published under [MIT License](LICENSE)

> *"The quieter you become, the more you are able to hear."* — Kali Linux motto
