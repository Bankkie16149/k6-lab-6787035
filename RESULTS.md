# K6 lab results

## Part 0
- Step 0.1, http_req_duration: avg=278.9ms min=278.9ms med=278.9ms max=278.9ms p(90)=278.9ms p(95)=278.9ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5:
  - Total requests: 69
  - Request rate: 3.27/s
  - p(95): 616.88ms
  - เหตุผลที่ rate ไม่ถึง 5 requests/s ทั้งที่มี 5 VUs:
    เพราะในโค้ดมีคำสั่ง sleep(1) คือหยุดรอ 1 วินาทีในทุกรอบ รวมกับเวลาที่เครือข่ายและเซิร์ฟเวอร์ใช้ตอบกลับ (Response Time) ทำให้ 1 รอบการทำงานของผู้ใช้ 1 คน (VU) ใช้เวลารวมมากกว่า 1 วินาทีเสมอ ส่งผลให้ 1 VU ส่งคำขอได้ไม่ถึง 1 ครั้งต่อวินาที เมื่อรวม 5 VUs อัตราการส่งคำขอจึงอยู่ที่ประมาณ 3.5 - 4 ครั้งต่อวินาที และไม่ถึง 5 ครับ
