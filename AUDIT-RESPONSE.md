# Audit Response — Seed Gift Forge v1.0.0 → fixed in v1.0.1

**Audited version:** v1.0.0 (`3fc8620c…32e94e`) · **Fixed in:** v1.0.1 (`1eef9946…402ae5c0`)  
**Auditor:** ChatGPT (external review, 2026-10-18) · **Response:** 2026-10-19
**Method:** every finding was first reproduced as a failing automated test, then fixed, then re-tested. Regression suite (Node.js + jsdom, 22 automated tests) — **22 / 22 pass**.

> **สรุป:** ผลตรวจถูกต้องทั้ง 8 ข้อ (ยืนยันด้วยการทดสอบซ้ำได้ผลเดียวกับที่ผู้ตรวจรายงาน รวมถึง address ผิด `bc1qffez…` และ fingerprint `cdd489e3`) แก้ไขครบทุกข้อพร้อมประเด็นรอง และเพิ่ม self-test ในแอปจาก 10 เป็น 13 รายการ

| # | Finding | Reproduced | Fix | Test |
|---|---|---|---|---|
| 1 | Export ได้ข้อมูลกระเป๋าเก่า ถ้าเปลี่ยน Seed/Passphrase แล้วกดเร็ว | ✅ ปุ่ม export ยังเปิดระหว่าง debounce | เปลี่ยนข้อมูล → ล้างผลเดิมทันที + สถานะ `deriving` · ผลที่คำนวณเสร็จจะถูกทิ้งถ้าข้อมูลเปลี่ยนระหว่างคำนวณ · ตรวจ `derivedKey === curKey()` ทั้งตอนเปิดปุ่มและตอนกด · `exportPdf()` ตรวจซ้ำเองและตรึง snapshot ของข้อมูลก่อนสร้างไฟล์ | ปุ่มปิดภายใน 5 ms หลังเปลี่ยน · คลิกระหว่างคำนวณไม่ดาวน์โหลด · ได้ fp `cdd489e3` |
| 2 | QR descriptor ใช้ `/<0;1>/*` ตายตัว ไม่ตรงกับเส้นทางที่นำเข้า | ✅ | โมเดลเส้นทาง `steps` (index / `<a;b>`) เก็บจากต้นฉบับ · `descriptorString()` สร้างจากเส้นทางเดิม · ตรวจ round-trip: address ก่อน/หลังสร้าง QR ต้องตรงกัน ไม่งั้นหยุด | `/1/*`, `/7/0/*`, `/0/*`, `/<0;1>/*` round-trip ตรง |
| 3 | UR multipath pair ถูกตีความผิด → address ผิด | ✅ ได้ `bc1qffez…` ตรงตามรายงาน | เขียนตัวอ่าน keypath ตาม BCR-2020-007: index(+bool), wildcard(+bool), range(+bool), pair `[a,h,b,h]` (ไม่มี bool ตาม) · ตรวจชนิด/ช่วงของ index · wildcard ต้องอยู่ท้าย | UR pair → `bc1qcr8t…` ✅ (เพิ่มเป็น self-test ในแอป) |
| 4 | ขึ้น MATCH ทั้งที่ script/เส้นทางต่างกัน | ✅ `pkh(…/1/*)` ขึ้น MATCH | `compareWallets()` เทียบ 5 มิติ: account key · fingerprint · script · network · address #1–#3 → `MATCH` / `ตรวจได้ไม่ครบ` / `NOT MATCH` · โหมดนำเข้า: มีแค่ fingerprint = “ตรวจได้ไม่ครบ” ไม่ใช่ MATCH | pkh /1/* ไม่ MATCH · descriptor ถูกต้อง MATCH |
| 5 | นำเข้าข้อมูลผิดแล้วยัง export ได้ | ✅ | มีข้อความในช่องนำเข้า/ตรวจเทียบแต่อ่านไม่ได้ = ล็อก export จนกว่าจะแก้หรือล้างช่อง (แยกจากกรณีตั้งใจไม่ใส่ข้อมูลกระเป๋า) · ตรวจเทียบได้ `NOT MATCH` หรือไม่ครบ = ล็อก | ปุ่มปิด |
| 6 | Self-test ล้มเหลวแล้วปุ่ม export เปิดกลับได้ | ✅ | เก็บ `S.selfTestOk` ถาวร · ตรวจทั้งใน `exportErrors()` และ `exportPdf()` · เพิ่ม self-test: descriptor parser + round-trip, UR multipath keypath, entropy encoding (รวม 13 รายการ) | ทำให้ self-test พังโดยตั้งใจ → ปุ่มยังปิด |
| 7 | Descriptor ผิดรูปแบบถูกยอมรับ (`/*/*`, key เดี่ยวถูกเติม `/0/*`) | ✅ | Parser แบบเข้มงวด: `*` เฉพาะท้ายสุด · index ต้องเป็นจำนวนเต็ม 0..2³¹−1 (non-hardened) · `<a;b>` ได้ 1 ตำแหน่ง · descriptor ที่ไม่มี `/*` = ปฏิเสธ (ไม่เดาเส้นทาง) · xpub เปล่า (ไม่ใช่ descriptor) สมมติ `/<0;1>/*` แบบเปิดเผย พร้อมแจ้งบนหน้าจอ | `/*/*`, `/0/x/*`, key เดี่ยว, `/0'/*` ถูกปฏิเสธ |
| 8 | อินพุต entropy ต่างกันกลายเป็นไบต์เดียวกัน (เติม 0) | ✅ | โหมด SHA-256: เหรียญต้องครบ 8 บิต, HEX ต้องเป็นจำนวนหลักคู่ — ไม่เติม 0 อีกต่อไป · อินพุตที่ครบไบต์ให้ผล **เหมือนเดิมทุกไบต์** (เข้ากันได้กับ BIP39-Generator) | 129 บิต/33 หลักถูกปฏิเสธ · ผลอินพุตครบไบต์ตรงกับ SHA-256 อ้างอิง |

## Secondary items

| Item | Action |
|---|---|
| รหัสผ่าน PDF ใช้แค่ NFKC (ไม่ใช่ SASLprep เต็ม) | **จำกัดรหัสผ่าน PDF เป็น ASCII ที่พิมพ์ได้ (0x20–0x7E)** — ไม่มีอักขระที่ต้อง map ตาม SASLprep จึงเปิดได้ตรงกันทุกโปรแกรม |
| `masterFromSeed()` ไม่ตรวจ IL = 0 / ≥ n | เพิ่มการตรวจตาม BIP32 |
| UR ไม่มี parent fingerprint | แสดงหมายเหตุ “xpub metadata ไม่ครบ (address ยังถูกต้อง)” |
| ตัวกรอง Real Entropy ลบอักขระเงียบ ๆ | แจ้งจำนวนอักขระที่ถูกตัด + คำแนะนำ “ไพ่ 10 ให้พิมพ์ t” |
| QR หลายเฟรม | ระบุข้อจำกัดในกล่องอัปโหลด |

## Also fixed during this cycle (found internally)
- **ไพ่ 52 ใบ:** เดิมห้ามใบซ้ำแต่นับ 5.7 บิต/ใบ (ขัดกัน และทั้งสำรับมีแค่ ≈ 225.6 บิต) → ใช้การจั่วแบบ “ใส่คืน + สับใหม่” ให้ตรงกับสูตรนับบิต

## Re-verification after fixes
- Crypto vectors **143/143** · Independent `@scure` cross-check **20/20** wallets · Thai shaper **0 mismatch** · UI end-to-end ✅ · Encrypted PDF opens in qpdf/poppler/pdf.js ✅

---

# Round 2 — re-audit of the fixed build (2026-10-19)

**Auditor summary:** core cryptography & standard Seed + Passphrase → Descriptor path: **767 tests, no calculation errors** (80 wallets, 240 addresses, 12–24 words, all 4 script types, mainnet/testnet, Thai/Unicode passphrases, 8 real PDFs). All 9 round-1 items confirmed fixed. Remaining issues were in **import / verification / export state** — all reproduced and fixed. Regression suite: **19 / 19 pass**.

| # | Finding | Reproduced | Fix |
|---|---|---|---|
| R2-1 | แก้ข้อความนำเข้าแล้ว ระหว่างหน่วง 250 ms ยังใช้ผลเก่า + ผลอ่านที่เสร็จช้ากว่าเขียนทับผลใหม่ได้ | ✅ | `walletChanged()` ล้างผลกระเป๋า/ตรวจเทียบทันทีที่ข้อความเปลี่ยน · สถานะ `walletPending` ระงับ export · หมายเลขคำขอ `walletRev` รับเฉพาะผลล่าสุด · ตรวจว่าข้อความที่อ่านสำเร็จตรงกับข้อความปัจจุบัน — ใช้กับการอัปโหลดภาพ QR ด้วย |
| R2-2 | QR แบบ zpub ทำให้เส้นทางที่นำเข้าหาย (เช่น `/1/*`) | ✅ | `zpubIssue()` อนุญาต zpub/ypub เฉพาะ Native/Nested SegWit บน mainnet ที่ใช้เส้นทางมาตรฐาน `/<0;1>/*` หรือ `/0/*` และตรวจย้อนกลับว่า address ตรง — กรณีอื่นต้องใช้ Descriptor (Taproot/Legacy ใช้ zpub ไม่ได้) |
| R2-3 | ตรวจไขว้ได้ NOT MATCH แล้วยัง export ได้ · แก้ path หลังตรวจแล้วผลเดิมยังค้าง | ✅ | สถานะ `not_checked / checking / match / mismatch / incomplete / error` ผูกกับลายเซ็นของข้อมูลที่ใช้ตรวจ (seed + account key + fingerprint + path + script + ข้อความนำเข้า) — ข้อมูลเปลี่ยน = ผลหมดอายุ · `mismatch` ระงับ export |
| R2-4 | MATCH ไม่ตรวจเส้นทาง change (`/<0;1>` vs `/<0;7>`) | ✅ | ตาม BIP389: เปรียบเทียบ address ของทั้ง receive และ change เมื่อทั้งสองฝั่งระบุ multipath · ถ้า QR ไม่ระบุ change แสดงชัดว่า “ตรวจเฉพาะ receive” |
| R2-5 | Canvas fallback ใช้ object spread กับ `ImageData` ทำให้ `data` หาย (กระทบ JPEG) | ✅ (ตรวจโค้ด) | คืน `{ data, width, height }` จาก `ImageData` โดยตรง |
| R2-6 | อัปโหลดภาพที่อ่าน QR ไม่ได้ (ช่องข้อความว่าง) แล้วยัง export ได้ | ✅ | สถานะนำเข้าอิสระจากช่องข้อความ — `walletErr` ระงับ export เสมอ · เพิ่มปุ่ม **✕ ล้างข้อมูลนำเข้า** |
| R2-7 | Ypub/Zpub (multisig) ทำให้เกิด error ภายใน | ✅ `Cannot read properties of undefined` | ปฏิเสธตั้งแต่ parser: “Extended key นี้เป็นของ Multisig — รองรับเฉพาะ Single-sig” |
| R2-8 | UR multipath pair ไม่ตรวจ index (`-1`, `2³¹`, `[0,0]`) | ✅ | ตรวจทั้งสองสมาชิก: จำนวนเต็ม 0..2³¹−1, hardened flag เป็น boolean, a ≠ b · ปฏิเสธ range |

**Re-verification after round 2:** audit #1 regression 22/22 · audit #2 regression 19/19 · crypto vectors 143/143 · `@scure` cross-check 20/20 · self-test 13/13 · Thai shaper 0 mismatch · UI end-to-end ✅

---

# Round 3 — re-audit (2026-10-19)

**Auditor summary:** 767 calculation tests pass again; all 8 round-2 items confirmed fixed. Three reproducible issues remained — all reproduced and fixed. Regression suite: **11 / 11 pass**.

| # | Finding | Reproduced | Fix |
|---|---|---|---|
| R3-1 | ผลตรวจไขว้ของกระเป๋าเก่าถูกผูกกับกระเป๋าใหม่ (เปลี่ยนข้อมูลระหว่างรอ 30 ms → ขึ้น MATCH ให้กระเป๋าที่ไม่ได้ตรวจ) | ✅ | ตรึง seed, passphrase, กระเป๋า และลายเซ็น `sig0` **ก่อน** await · หมายเลขคำขอ `crossRev` · ตรวจซ้ำก่อนและหลังคำนวณ ถ้าข้อมูลเปลี่ยน → ทิ้งผลและแจ้งให้ตรวจใหม่ · บันทึกผลด้วย `sig0` เท่านั้น · การเปลี่ยนข้อมูลนำเข้ายกเลิกการตรวจที่ค้างอยู่ |
| R3-2 | ปุ่ม Export เปิดกลับได้ระหว่างกำลังสร้าง PDF | ✅ | สถานะ `S.exporting` (กันการเรียกซ้อน) · `disabled = exporting OR errors` · สร้าง snapshot ภายใน `try` และคืนสถานะใน `finally` เสมอ |
| R3-3 | PNG Grayscale/RGB ที่ใช้ tRNS (สีโปร่งใส) ถูกถอดเป็นพื้นเทา → อ่าน QR ไม่ได้ | ✅ พื้น `[129,129,129]` | รองรับ tRNS ของ Grayscale/RGB (เทียบค่าดิบก่อนลดบิต รองรับ 16 บิต) · ถ้าตัวถอด PNG ภายในหา QR ไม่เจอ ลอง canvas fallback ทุกขนาดภาพ · เพิ่ม timeout 4 วินาทีตอนโหลดภาพ |
| R3-+ | นำเข้า xpub เปล่าโดยไม่เลือก Script type แล้วยัง “พร้อมส่งออก” (ไม่มี Address) | ✅ | **บังคับเลือก Script type** เมื่อมีข้อมูลกระเป๋านำเข้า (หรือกด ✕ ล้างข้อมูลนำเข้า ถ้าไม่ต้องการพิมพ์ข้อมูลกระเป๋า) |

**Re-verification after round 3:** audit #1 22/22 · audit #2 19/19 · audit #3 11/11 · crypto vectors 143/143 · `@scure` 20/20 · self-test 13/13 · PNG decoder (5 color types + 2 tRNS) ✅ · Thai shaper 0 mismatch · UI end-to-end ✅

*Don't Trust, Verify.*
