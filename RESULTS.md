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

## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.50/s | 849.84ms | 964.41ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.02/s | 2.13s | 8.12s | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.20/s | 436.12ms | 1.14s | 0.00% | 100.00% |
| A4 | Spike | 5 | 1.07/s | 410.89ms | 1.14s | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.96/s | 449.63ms | 1.16s | 0.00% | 100.00% |

## Part B
1. **Fill in the results table for Steps A1 to A5:**
   - กรอกข้อมูลผลการทดสอบลงในตาราง Part A results ด้านบนเรียบร้อยแล้ว

2. **In Step 0.2, why was the request rate below 5 requests per second with 5 virtual users?**
   - เพราะในฟังก์ชัน default มีคำสั่ง `sleep(1)` (Think time) 1 วินาทีในทุกรอบการทำงาน บวกกับเวลาในการส่งข้อมูลไป-กลับผ่านเครือข่ายและเวลาที่เซิร์ฟเวอร์ใช้ประมวลผล (Network Latency + Server Response time) ทำให้ระยะเวลารวมในแต่ละรอบ (Iteration duration) เกินกว่า 1 วินาทีเสมอ ส่งผลให้ผู้ใช้ 1 คน (VU) ไม่สามารถส่งคำขอได้ถึง 1 request ต่อวินาที เมื่อรวม 5 VUs อัตราการยิงจึงอยู่ต่ำกว่า 5 requests/s (ได้ประมาณ 3.27 - 3.7 req/s)

3. **Did the response time grow from the average-load test to the stress test? Explain your answer with the operational envelope.**
   - จากผลการทดสอบ Response time ในการทดสอบ Stress test ไม่ได้เพิ่มขึ้นอย่างต่อเนื่องเมื่อเทียบกับขีดความสามารถของระบบ (ในรอบ A2 มีค่า p(95)=2.13s และ Max=8.12s ซึ่งเกิดจาก Internet Latency ชั่วคราว แต่ใน A3 เมื่อเพิ่มผู้ใช้เป็น 5 VUs ค่า p(95) อยู่ที่ 436.12ms และ Max=1.14s)
   - อธิบายด้วย **Operational Envelope**: QuickPizza ถูกโฮสต์บนระบบคลาวด์ที่มีขอบเขตการทำงานที่รองรับโหลดได้สูงในระดับสิบหรือร้อย VUs การที่เราทดสอบเพียง 3 ถึง 5 VUs จึงถือว่าเป็นการทดสอบที่อยู่ **ลึกเข้าไปข้างใน (Far inside)** ขอบเขตการทำงานของระบบ ยังห่างไกลจากขอบเขตจำกัด (Edge of envelope) ทำให้ระบบไม่เกิดอาการ Overload หรือแย่งทรัพยากรกัน ความเร็วในการตอบสนองจึงยังคงที่และรวดเร็ว

4. **In the spike test, how long did the response time take to return to normal after the spike?**
   - เมื่อตรวจสอบรายงานกราฟใน `reports/a4-spike.html` ในช่วงจังหวะที่โหลดพุ่งขึ้นอย่างรวดเร็วจาก 1 เป็น 5 VUs ภายใน 5 วินาที ระบบยังคงตอบสนองได้ดี โดยมีค่า p(95) อยู่ที่ 410.89ms และเมื่อโหลดลดกลับลงมาเหลือ 1 VU ระบบสามารถกลับคืนสู่ระดับเวลาปกติได้ทันทีแทบจะในทันที (ภายในเวลาไม่เกิน 5-10 วินาที) เนื่องจากจำนวน 5 VUs ไม่ทำให้เกิด Request queue สะสมที่เซิร์ฟเวอร์

5. **In the soak test, did the response time or the error rate get worse over time? Name one kind of defect that a soak test of several hours could reveal.**
   - จากการทดสอบ Soak test นาน 10 นาที (633 requests) ทั้ง Response time (p(95)=449.63ms) และ Error rate (0.00%) ไม่ได้แย่ลงตามเวลา ระบบยังคงมีความเสถียรดี
   - **ข้อบกพร่องที่ Soak test นานหลายชั่วโมงสามารถตรวจพบได้**:
     - **Memory Leak**: การที่แอปพลิเคชันจองหน่วยความจำเพิ่มขึ้นเรื่อยๆ ตลอดการทำงานและไม่ยอมคืนหน่วยความจำ (Garbage collection คืนไม่หมด) จนในที่สุดหน่วยความจำ (RAM) เต็ม และเซิร์ฟเวอร์เกิดข้อผิดพลาด Out of Memory (OOM Crash)
     - หรือ **Resource / Connection Leak** เช่น การเปิด Database Connection ค้างไว้แล้วไม่ปิดคืน Connection Pool

6. **Why do the thresholds in this lab use p(95) rather than the average?**
   - เพราะค่าเฉลี่ย (Average) จะถูกบดบัง (Masked) ด้วยคำขอส่วนใหญ่ที่ตอบสนองเร็ว ทำให้ไม่เห็นปัญหาของผู้ใช้ที่เจอความล่าช้า ในขณะที่ **p(95) (95th Percentile)** เป็นตัวชี้วัดที่บอกว่า 95% ของคำขอเร็วกว่าค่านั้น และมีผู้ใช้เพียง 5% (1 ใน 20 คน) ที่ช้ากว่า ซึ่งสะท้อนประสบการณ์ของผู้ใช้งานจริง (User Experience) และสามารถตรวจจับความล่าช้าผิดปกติ (Tail Latency / Outliers) ได้อย่างแม่นยำกว่าค่าเฉลี่ย

7. **Your results come from a shared public site reached over the internet. Give two reasons why the same tests against a system in a lab could give different numbers.**
   - 1. **Network Latency and Jitter**: การทดสอบผ่านเครือข่ายอินเทอร์เน็ตสาธารณะมีค่า Latency ผันผวนตามการเชื่อมต่อของผู้ให้บริการอินเทอร์เน็ต (ISP) สายเคเบิล และเราเตอร์ ในขณะที่การทดสอบใน Lab เครือข่ายเป็น Local Area Network (LAN) หรือ Localhost ซึ่งมีความหน่วงต่ำมาก (< 1 ms) และเสถียรสูงกว่ามาก
   - 2. **Shared Traffic / Noisy Neighbors**: เว็บสาธารณะมีผู้ใช้งานคนอื่นทั่วโลกเข้ามาทดสอบพร้อมๆ กัน ทำให้โหลดของเซิร์ฟเวอร์เปลี่ยนแปลงอยู่ตลอดเวลาตามการใช้งานของคนอื่น ในขณะที่ระบบใน Lab ปิด เราสามารถควบคุมสภาพแวดล้อมได้ 100% ไม่มีทราฟฟิกภายนอกมารบกวน

8. **Why must you never run a stress, spike or breakpoint test with hundreds of users against a site that you do not own?**
   - 1. **เสี่ยงต่อการถูกมองว่าเป็นการโจมตีแบบปฏิเสธการให้บริการ (Denial of Service - DoS)**: ซึ่งทำให้ระบบของผู้อื่นล่ม ผู้ใช้งานจริงไม่สามารถเข้าใช้บริการได้ และเป็นการกระทำที่ผิดกฎหมาย (เช่น พ.ร.บ. คอมพิวเตอร์)
   - 2. **สร้างภาระค่าใช้จ่ายที่ไม่พึงประสงค์ (Infrastructure Costs)** ให้กับเจ้าของเว็บไซต์ จากการคิดค่าบริการตามปริมาณ Bandwidth หรือการ Scale Server บน Cloud อัตโนมัติ
   - 3. **เสี่ยงต่อการถูกแบนหรือบล็อก IP Address**: ระบบป้องกันและไฟร์วอลล์ (WAF) จะตรวจจับพฤติกรรมผิดปกติและบล็อกการเชื่อมต่อทันที

9. **Write one vague performance requirement for an app you use every day. Rewrite it as a testable requirement, and show the K6 thresholds that check it.**
   - **Vague requirement (requirement ที่คลุมเครือ)**: "หน้าค้นหาและสั่งอาหารของแอป LINE MAN ต้องโหลดเร็วและไม่กระตุก"
   - **Testable requirement (ปรับให้วัดผลได้ชัดเจน)**: "เมื่อมีผู้ใช้งานพร้อมกัน 20 คน (20 concurrent VUs) ค้นหาและดูเมนูอาหาร (`/api/restaurants/search`) ต่อเนื่องเป็นเวลา 2 นาที อัตราการตอบสนอง 95% ของคำขอต้องเร็วกว่า 800 ms (p(95) < 800ms) และมีอัตราคำขอล้มเหลว (Failed rate) ไม่เกิน 1%"
   - **K6 thresholds ที่ใช้ตรวจสอบ**:
     ```javascript
     export const options = {
       vus: 20,
       duration: '2m',
       thresholds: {
         'http_req_duration': ['p(95)<800'],
         'http_req_failed': ['rate<0.01'],
         'checks': ['rate>0.99'],
       },
     };
     ```
