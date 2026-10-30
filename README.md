# ITDS362 Software Quality Assurance and Testing
## Lab: Performance Testing with Grafana K6

Repository สำหรับส่งงานการทดลอง Performance Testing ด้วย Grafana K6 โดยทดสอบกับเว็บเดโม [QuickPizza](https://quickpizza.grafana.com)

---

### 📄 รายงานสรุปผลการทดลอง (Lab Results)
👉 **ดูผลการทดลองทั้งหมดและคำตอบ Reflection ได้ที่นี่: [RESULTS.md](./RESULTS.md)**

---

### 📂 โครงสร้างโปรเจกต์ (Repository Structure)

```text
k6-lab-6787035/
├── README.md           # ไฟล์แนะนำโปรเจกต์
├── RESULTS.md          # สรุปผลการทดลอง Part 0, ตาราง Part A และคำตอบ Part B
├── 01-first-test.js    # Step 0.1 - 0.3: การเขียนเทสต์แรก, ทดสอบ duration และ iterations
├── 04-pizza-api.js      # Step 0.4: การทดสอบ API และตรวจ JSON response
├── 05-no-token.js       # Step 0.5: การทดสอบความปลอดภัย (Expected status 401)
├── 06-thresholds.js     # Step 0.6: การกำหนด Thresholds และตรวจสอบ Exit code
├── 07-requirement.js    # Step 0.7: แปลง Requirement เป็น Thresholds
├── 08-bug-hunt.js       # Step 0.8: สคริปต์ค้นหาและแก้ไขบั๊ก (Case-sensitive parameters)
├── explore.js           # สคริปต์สืบค้น API metadata เพื่อวิเคราะห์บั๊ก
├── journey.js           # User Journey รวมของ QuickPizza (Home -> Quotes -> Recommendation)
├── a1-smoke.js          # Step A1: Smoke test (1 user, 30s)
├── a2-average.js        # Step A2: Average load test (ramp up 3 users, 3m)
├── a3-stress.js         # Step A3: Stress test (ramp up 5 users, 3m)
├── a4-spike.js          # Step A4: Spike test (พุ่งขึ้น 5 users ใน 5s)
├── a5-soak.js           # Step A5: Soak test (2 users, 11m)
└── reports/             # โฟลเดอร์เก็บรายงาน HTML Dashboard
    ├── a2-average.html  # รายงานกราฟ Average load test
    ├── a3-stress.html   # รายงานกราฟ Stress test
    ├── a4-spike.html    # รายงานกราฟ Spike test
    └── a5-soak.html     # รายงานกราฟ Soak test
```

---

### 🚀 วิธีการรันการทดสอบ (How to Run)

สามารถรันสคริปต์ต่างๆ ผ่านคำสั่ง K6 ในโฟลเดอร์ root ได้ดังนี้:

```bash
# 1. รัน Basic Tests (Part 0)
k6 run 01-first-test.js
k6 run 04-pizza-api.js
k6 run 05-no-token.js
k6 run 06-thresholds.js
k6 run 07-requirement.js
k6 run 08-bug-hunt.js

# 2. รัน Load Tests พร้อม Export รายงาน HTML (Part A)
k6 run a1-smoke.js
k6 run --out web-dashboard=export=reports/a2-average.html a2-average.js
k6 run --out web-dashboard=export=reports/a3-stress.html a3-stress.js
k6 run --out web-dashboard=export=reports/a4-spike.html a4-spike.js
k6 run --out web-dashboard=export=reports/a5-soak.html a5-soak.js
```
