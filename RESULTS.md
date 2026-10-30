# K6 lab results

## Part 0
- Step 0.1, http_req_duration: avg=278.9ms min=278.9ms med=278.9ms max=278.9ms p(90)=278.9ms p(95)=278.9ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5:
  - Total requests: 69
  - Request rate: 3.27/s
  - p(95): 616.88ms
  - เหตุผลที่ rate ไม่ถึง 5 requests/s ทั้งที่มี 5 VUs:
    เพราะในโค้ดมีคำสั่ง sleep(1) คือหยุดรอ 1 วินาทีในทุกรอบ รวมกับเวลาที่เครือข่ายและเซิร์ฟเวอร์ใช้ตอบกลับ (Response Time) ทำให้ 1 รอบการทำงานของผู้ใช้ 1 คน (VU) ใช้เวลารวมมากกว่า 1 วินาทีเสมอ ส่งผลให้ 1 VU ส่งคำขอได้ไม่ถึง 1 ครั้งต่อวินาที เมื่อรวม 5 VUs อัตราการส่งคำขอจึงอยู่ที่ประมาณ 3.5 - 4 ครั้งต่อวินาที และไม่ถึง 5 ครับ
- Step 0.6, exit code of the normal run and of the run with P95=300:
  - Exit code (normal run): 0
  - Exit code (P95=300): 99
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
  - Rules that passed: ผ่านครบทุกกฎ (http_req_failed, p(95)<1000, p(99)<1500, checks)
  - Measured p(95): 396.82ms
  - Measured p(99): 413.9ms
  - Exit code after change to fail: 99
- Step 0.8, what was wrong, and how the checks showed it:
  - สิ่งที่ผิดพลาด: ค่าใน excludedIngredients และ excludedTools ในคำขอเริ่มต้นสะกดด้วยตัวพิมพ์เล็ก ('pepperoni' และ 'knife') ซึ่ง API ของ QuickPizza เป็น Case-sensitive และคาดหวังชื่อที่ขึ้นต้นด้วยตัวพิมพ์ใหญ่ ('Pepperoni' และ 'Knife') ตามที่ตรวจพบจาก explore.js ส่งผลให้ API ไม่ได้ตัดส่วนผสมและอุปกรณ์ดังกล่าวออก และยังคงสุ่มพิซซ่าที่มี Pepperoni หรือใช้ Knife มาให้ลูกค้า
  - ทำไม Load Test ถึงเจอบั๊กนี้ได้ดีกว่าการยิง Manual รอบเดียว: เพราะระบบสุ่ม (random) วัตถุดิบและอุปกรณ์ ถ้าเรากดทดสอบมือ (Manual) เพียง 1-2 ครั้ง เราอาจจะบังเอิญสุ่มได้พิซซ่าชนิดอื่นที่ไม่ใช่ Pepperoni ทำให้เข้าใจผิดว่าเงื่อนไข exclusion ทำงานถูกต้องแล้ว แต่เมื่อรันด้วย Load Test หลายๆ รอบพร้อมกัน (5 VUs, 20s) จะทำให้เกิดการสุ่มซ้ำๆ หลายสิบครั้ง จนพบเคสที่หลุด Pepperoni หรือ Knife ออกมา และ Check ฟ้องความผิดพลาดให้เห็นได้ทันทีครับ
