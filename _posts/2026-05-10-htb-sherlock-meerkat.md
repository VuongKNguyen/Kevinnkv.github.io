---
title: "HTB Sherlock Write-up: Meerkat"
date: 2026-05-10 20:00:00 +0700
categories: [HackTheBox, Sherlock]
tags: [htb, sherlock, forensics, json, jq, blue-team]
description: "Phân tích forensics với file JSON — kỹ thuật dùng jq để xử lý và filter log dữ liệu."
---

## Thông tin Challenge

| Field | Info |
|---|---|
| **Name** | Meerkat |
| **Category** | Forensics |
| **Difficulty** | Easy |
| **Platform** | HackTheBox Sherlock |

---

## Tóm tắt (TL;DR)

> Challenge cung cấp file `.json` chứa log dữ liệu. Nhiệm vụ là dùng công cụ `jq` để parse, filter và trích xuất thông tin cần thiết nhằm trả lời các câu hỏi điều tra.

---

## Phân tích

### Bước 1: Xem cấu trúc file JSON

Trước tiên cần hiểu cấu trúc của file trước khi query:

```bash
jq . file.json
```

Lệnh này format JSON cho dễ đọc. Quan sát các field có trong mỗi object để biết mình đang làm việc với dữ liệu gì.

### Bước 2: Các lệnh jq cơ bản dùng trong challenge

**Lấy một field cụ thể:**
```bash
jq '.field' file.json
```

**Iterate qua array:**
```bash
jq '.[]' file.json
```

**Filter theo điều kiện:**
```bash
jq '.[] | select(.status == "failed")' file.json
```

**Raw output (không có dấu ngoặc kép):**
```bash
jq -r '.[] | .field' file.json
```

**Custom output — ghép nhiều field:**
```bash
jq -r '.[] | "\(.username) \(.ip)"' file.json
```

### Bước 3: Đếm tần suất — tìm anomaly

Kỹ thuật rất hay để phát hiện bất thường: đếm số lần xuất hiện của mỗi giá trị.

```bash
jq -r '.[] | .username' file.json | sort | uniq -c | sort -rn
```

Giải thích pipeline:
- `jq -r` → xuất raw text từng dòng
- `sort` → sắp xếp để các giá trị giống nhau nằm cạnh nhau
- `uniq -c` → đếm số lần xuất hiện liên tiếp
- `sort -rn` → sắp xếp giảm dần theo số đếm

**Kết quả mẫu:**
```
  47 admin
  12 user1
   3 guest
   1 root
```

→ Username `admin` xuất hiện 47 lần bất thường → dấu hiệu brute force.

### Bước 4: Filter theo thời gian hoặc IP

```bash
# Lọc theo IP cụ thể
jq '.[] | select(.src_ip == "192.168.1.100")' file.json

# Lọc các event failed login
jq '.[] | select(.event_type == "login" and .status == "failed")' file.json

# Đếm failed login theo IP
jq -r '.[] | select(.status == "failed") | .src_ip' file.json \
  | sort | uniq -c | sort -rn
```

---

## Lessons Learned

1. **jq là công cụ không thể thiếu** khi làm forensics với dữ liệu JSON — nắm vững cú pháp cơ bản sẽ tiết kiệm rất nhiều thời gian.
2. **`sort | uniq -c`** là combo kinh điển để phát hiện anomaly trong log — bất cứ giá trị nào xuất hiện với tần suất bất thường đều đáng điều tra.
3. **Pipeline** trong bash cho phép chain nhiều bước xử lý phức tạp thành một lệnh duy nhất.

---

## Tham khảo

- [jq Manual - Official Documentation](https://jqlang.github.io/jq/manual/)
- [HackTheBox Sherlock - Meerkat](https://app.hackthebox.com/sherlocks/Meerkat)
