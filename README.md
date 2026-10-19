<div align="center">

# ₿ Seed Gift Forge

**Turn a Bitcoin wallet into a beautiful, printable, encrypted gift — fully offline, in a single HTML file.**

**เปลี่ยนกระเป๋าบิตคอยน์ให้เป็น “การ์ดของขวัญ” ที่สวยงาม พร้อมพิมพ์ และเข้ารหัส — ทำงานออฟไลน์ 100% ในไฟล์ HTML ไฟล์เดียว**

`v1.0.1` · Single-file · Offline-first · Zero network · AES-256 PDF · BIP39 / BIP32 / BIP84 · Tails OS ready

*Don't Trust, Verify.*

![Seed Gift Forge — wedding, birthday and Songkran designs](docs/preview-gallery.png)

<sub>ตัวอย่างใช้ seed ทดสอบสาธารณะ (`abandon … about`, `zoo … vote`) เท่านั้น — ห้ามส่งเงินเข้า address ในภาพ</sub>

</div>

---

## 📌 TL;DR

**EN —** Seed Gift Forge is a one-stop, air-gapped web app for *gifting bitcoin on paper*. Generate a seed (system CSPRNG or your own dice/coin/card entropy), set a BIP39 passphrase, derive a Native SegWit (BIP84) watch-only wallet, then export a gift-styled, **AES-256 encrypted PDF** — wedding, birthday, anniversary, New Year, Songkran, graduation or keepsake — ready to fold into an envelope. Everything runs in one HTML file with no network access, no storage, and a built-in self-test.

**TH —** Seed Gift Forge คือเว็บแอป “จบในที่เดียว” สำหรับ *มอบบิตคอยน์เป็นของขวัญบนกระดาษ* สร้าง Seed ได้ทั้งจากระบบหรือจาก Entropy ที่สุ่มเอง ตั้ง Passphrase คำนวณกระเป๋า Native SegWit (BIP84) พร้อม QR สำหรับ watch-only แล้วสร้าง **PDF เข้ารหัส AES-256** ในรูปแบบการ์ดของขวัญตามเทศกาล พิมพ์แล้วพับใส่ซองได้ทันที ทั้งหมดอยู่ในไฟล์ HTML ไฟล์เดียว ไม่มีการเชื่อมต่อเครือข่าย ไม่บันทึกข้อมูล และมี self-test ทุกครั้งที่เปิด

---

## 🆕 What's new in v1.0.1

อัปเดตด้านความปลอดภัยและความถูกต้อง หลังการตรวจสอบภายนอก **3 รอบ** — แก้ข้อที่อาจทำให้ Seed, Address และ QR บนกระดาษไม่สอดคล้องกัน ทุกข้อมี regression test ยืนยัน (22/22 + 19/19 + 11/11)

- 🔴 ป้องกันการ Export ข้อมูลกระเป๋าเก่าเมื่อเปลี่ยน Seed/Passphrase
- 🔴 QR Descriptor ใช้เส้นทางเดิมของกระเป๋า + ตรวจ Address round-trip
- 🔴 แก้การอ่าน UR multipath ที่ทำให้ Address ผิด
- 🔴 ตรวจเทียบกระเป๋า 5 มิติ (key · fingerprint · script · network · address)
- 🟠 ล็อก Export เมื่อข้อมูลนำเข้าผิดหรือกำลังอ่าน / Self-test ไม่ผ่าน / ตรวจเทียบหรือตรวจไขว้ไม่ตรง
- 🟠 ตรวจเทียบทั้ง receive และ change path · QR แบบ zpub เฉพาะเส้นทางมาตรฐาน · ปฏิเสธ multisig key
- 🟠 Parser descriptor เข้มงวด · Entropy โหมด SHA ต้องครบไบต์ · รหัสผ่าน PDF เป็น ASCII
- 🟡 เรียงแหล่ง Seed ใหม่: **กรอก Seed เอง (ค่าเริ่มต้น) → Real Entropy → สุ่มจากระบบ**

รายละเอียด: [Release notes v1.0.1](RELEASE-NOTES-v1.0.1.md) · [AUDIT-RESPONSE.md](AUDIT-RESPONSE.md) · [CHANGELOG.md](CHANGELOG.md)

---

## ✨ Features · ความสามารถ

### 01 · Seed Phrase (BIP39)
| แหล่งที่มา | รายละเอียด |
|---|---|
| ⌨ **กรอก Seed เอง** *(ค่าเริ่มต้น)* | ช่องกรอก 12–24 คำ พร้อม autocomplete จาก BIP39 wordlist และตรวจ checksum ทันที |
| ✋ **Real Entropy** | โยนเหรียญ · HEX dice · ลูกเต๋า 6 หน้า · ไพ่ 52 ใบ → **SHA-256 × 1 รอบ** (ค่าเริ่มต้น) หรือ **Raw** (เฉพาะเหรียญ/HEX) |
| 🎲 **สุ่มจากระบบ** | `crypto.getRandomValues` (CSPRNG ของระบบปฏิบัติการ) สร้าง entropy ตรงตามจำนวนบิต — ไม่มี modulo bias |

- เลือกได้ **12 / 15 / 18 / 21 / 24 คำ** (128–256 บิต)
- **Hard gate:** entropy ไม่ครบ = ไม่สร้าง Seed · checksum ไม่ผ่าน = export ไม่ได้
- แสดงทุกขั้นตอน (entropy hex → SHA-256 → checksum bits) ให้ตรวจย้อนกลับได้
- ลูกเต๋า 6 หน้า/ไพ่ hash จากข้อความ UTF-8 ของผลทอย — แนวทางเดียวกับ Coldcard dice

### 02 · Passphrase & Wallet (BIP32 / BIP84)
- ช่อง **BIP39 Passphrase** + ยืนยัน + ปุ่มแสดง/ซ่อน — **ใช้คำนวณเท่านั้น ไม่ถูกพิมพ์ลง PDF**
- คำนวณอัตโนมัติ: **Master fingerprint · Derivation path `m/84'/0'/0'` · xpub · zpub · Output Descriptor (BIP380 checksum) · Address #1–#3**
- **ตรวจเทียบกับกระเป๋าที่มีอยู่:** อัปโหลด QR/วาง descriptor ที่ export จาก Sparrow → เทียบครบ 5 มิติ (Account key, Fingerprint, Script, Network, Address #1–#3) ผลเป็น `MATCH` / `ตรวจได้ไม่ครบ` / `NOT MATCH` — ถ้าไม่ใช่ `MATCH` ระบบล็อกการ export จนกว่าจะแก้หรือล้างช่องตรวจเทียบ
- QR descriptor บนกระดาษสร้างจาก **เส้นทางเดิมของกระเป๋าทุกประการ** และตรวจซ้ำว่า Address ก่อน/หลังสร้าง QR ตรงกันก่อนพิมพ์
- โหมด **นำเข้า** (ไม่กรอก Passphrase ในเครื่องนี้): รองรับ Output Descriptor (`wpkh` / `sh(wpkh)` / `pkh` / `tr`), **UR** (`crypto-output`, `output-descriptor`, `crypto-hdkey`, `crypto-account`), **BBQr**, xpub/ypub/zpub/tpub/vpub — คำนวณ address ได้ทั้ง Native SegWit, Taproot, Nested SegWit และ Legacy
- ข้อมูลที่คำนวณไม่ได้ (เช่น fingerprint ของ zpub เปล่า) มีช่องกรอกเพิ่มแบบไม่บังคับ — ถ้าไม่กรอกจะไม่แสดงบน PDF

### 03 · โอกาส & คำอวยพร — 7 ธีม
💍 แต่งงาน · 🎂 วันเกิด · 💞 ครบรอบแต่งงาน · 🎆 ปีใหม่ · 💧 สงกรานต์ · 🎓 รับปริญญา · 🎁 ของที่ระลึก

แต่ละธีมมีภาพเวกเตอร์ สีประจำธีม คำอวยพรภาษาไทย/อังกฤษ และลูกเล่นแบบ Bitcoiner (เช่น *“≈ 52,596 BLOCKS AROUND THE SUN”*) — แก้หัวข้อ คำอวยพร ชื่อผู้รับ วันที่ และจำนวนปีได้ทั้งหมด

### 04 · ผู้มอบ & ลายเซ็น
- ลายเซ็น 3 แบบ: **ลายมือประดิษฐ์จากชื่อ** (ฟอนต์ Charm) · **วาดเองด้วยเมาส์/นิ้ว** (เก็บเป็นเวกเตอร์ คมชัดทุกขนาด) · **เว้นว่างเพื่อเซ็นปากกาจริง**
- ตราประทับ ₿ “Don't Trust · Verify”, CERT NO. และ GIFT ID ประจำใบ

### 05 · Export PDF เข้ารหัส
- **AES-256 (PDF Standard Security Handler Revision 6 — ISO 32000-2)** owner password สุ่ม 256 บิต · อนุญาตพิมพ์/คัดลอก ห้ามแก้ไข
- เปิดได้ด้วยโปรแกรมมาตรฐาน: Adobe Acrobat/Reader, Preview (macOS), Chrome, Firefox/Tor Browser (pdf.js), Document Viewer/Evince ใน Tails (poppler), Foxit, SumatraPDF
- เวกเตอร์ทั้งหน้า ฝังฟอนต์ไทย (Sarabun/Charm) พร้อมจัดวางสระ-วรรณยุกต์ถูกต้อง · ค้นหา/คัดลอกข้อความได้
- QR บนกระดาษเลือกได้: **Output Descriptor** (Sparrow, Nunchuk, Bitcoin Core) · **zpub** (BlueWallet, Electrum ฯลฯ) · **Address #1** (ให้ญาติเติม sats) · ไม่พิมพ์
- ตัวเลือก: แจ้งว่ามี Passphrase · ประหยัดหมึก · พิมพ์ด้านนอก 2 หน้า · แยกการ์ด watch-only อีกแผ่น
- ปุ่ม 👁 แสดง/ซ่อนรหัสผ่าน · ชื่อไฟล์ต่อท้ายวันที่-เวลาอัตโนมัติ เช่น `bitcoin-gift_2026-10-18_23-18-36.pdf`
- แสดง SHA-256 ของ PDF ที่สร้าง

### 🛡 Security by design
- **ไฟล์เดียว ไม่มี dependency ภายนอกตอนรัน** — ฟอนต์ wordlist และไลบรารีฝังอยู่ในไฟล์
- **CSP ล็อกเครือข่าย:** `default-src 'none'; connect-src 'none'` — เบราว์เซอร์จะบล็อกทุกการเชื่อมต่อออก
- **ไม่ใช้ storage ใด ๆ** (ไม่มี localStorage / IndexedDB / cookie) · ล้างฟิลด์อัตโนมัติเมื่อปิดหน้า · ปุ่ม ⌫ ล้างทั้งหมด
- **Self-test 13 รายการทุกครั้งที่เปิด** (SHA-256/512, RIPEMD-160, AES-256 FIPS-197, BIP39 vectors, BIP84, BIP86, descriptor parser + round-trip, UR multipath keypath, entropy encoding, descriptor checksum, wordlist hash, Thai shaper) — ไม่ผ่านแม้รายการเดียว = ปุ่ม export ถูกล็อกถาวรจนกว่าจะเปิดไฟล์ใหม่
- ถอด QR จากไฟล์ PNG ด้วยตัวถอดที่เขียนเอง **ไม่ผ่าน canvas** (Tor Browser บล็อกการอ่านพิกเซล canvas เพื่อกัน fingerprinting)

---

## 🚀 Quick Start · เริ่มใช้งาน

### สิ่งที่ต้องมี
- คอมพิวเตอร์ที่บูต **[Tails OS](https://tails.net)** (แนะนำ) หรือเครื่องที่ตัดอินเทอร์เน็ตแล้ว
- ไฟล์ `seed-gift-forge.html` จากหน้า [Releases](../../releases) — **ตรวจ SHA-256 และลายเซ็น PGP ก่อนใช้** (ดูหัวข้อ [Verify](#-verify-the-release--ตรวจสอบไฟล์))
- เครื่องพิมพ์ที่ **ไม่ต่ออินเทอร์เน็ต/Wi-Fi** และกระดาษหนา 120 แกรมขึ้นไป

### ขั้นตอนบน Tails
1. บูต Tails → ที่ Welcome Screen เปิด **Offline Mode** (ปิดเครือข่าย)
2. ตั้ง Tor Browser เป็นระดับความปลอดภัย **Standard** — ระดับ Safer/Safest ปิด JavaScript ทำให้เครื่องมือไม่ทำงาน
3. เปิดไฟล์ด้วย Tor Browser (ลากไฟล์ใส่หน้าต่าง หรือ `file:///…/seed-gift-forge.html`) — ถ้าขึ้นว่า Tor ไม่พร้อม ให้กด *Start Tor Browser anyway*
4. ตรวจแถบ **Preflight** ด้านบน: `OFFLINE` และ `SELF-TEST PASS 13/13` ต้องเป็นสีเขียว

---

## 🧭 วิธีใช้งานแบบละเอียด (Mission Walkthrough)

### ขั้นที่ 01 — สร้าง Seed
| ต้องการ | ทำอย่างไร |
|---|---|
| มี Seed อยู่แล้ว *(ค่าเริ่มต้น)* | **⌨ กรอก Seed เอง** → เลือกจำนวนคำ → พิมพ์ หรือวางทั้งชุดในช่องแรก |
| สุ่มเองด้วยมือ | เลือก **✋ Real Entropy** → เลือกประเภท (เหรียญ/HEX/เต๋า 6/ไพ่) → กรอกผลจนตัวนับขึ้น `✓ ครบ` |
| เร็วและปลอดภัย | เลือก **🎲 สุ่มจากระบบ** → เลือกจำนวนคำ → กด *สุ่ม Seed ใหม่* |

จำนวนครั้งที่ต้องทอย/โยนให้ครบตามจำนวนบิต:

| คำ | บิต | เหรียญ | HEX | เต๋า 6 หน้า | ไพ่ 52 ใบ* |
|---|---|---|---|---|---|
| 12 | 128 | 128 | 32 | 50 | 23 |
| 24 | 256 | 256 | 64 | 100 | 45 |

<sub>* ไพ่: **จั่ว 1 ใบ → จด → ใส่คืน → สับใหม่** ทุกครั้ง (ใบซ้ำได้ตามปกติ) — ถ้าจั่วต่อเนื่องโดยไม่ใส่คืน เอนโทรปีจะน้อยกว่าที่นับ และทั้งสำรับมีเพียง ≈ 225.6 บิต ไม่พอสำหรับ 24 คำ</sub>

> โหมด SHA-256 สำหรับเหรียญต้องป้อนจำนวนครั้งที่หารด้วย 8 ลงตัว และ HEX ต้องเป็นจำนวนหลักคู่ (ครบไบต์พอดี) — ป้องกันอินพุตต่างกันกลายเป็นไบต์ชุดเดียวกันก่อน hash

แถบสถานะต้องขึ้น `✓ CHECKSUM VALID` ก่อนไปขั้นต่อไป

### ขั้นที่ 02 — Passphrase & Wallet
1. กรอก **Passphrase** 2 ครั้งให้ตรงกัน — ระบบคำนวณกระเป๋า BIP84 ให้อัตโนมัติ (≈ 0.1–0.5 วินาที)
2. ตรวจผล: fingerprint, xpub/zpub, descriptor และ Address #1–#3
3. *(แนะนำ)* เปิด **🔁 ตรวจเทียบกับ QR** → อัปโหลดภาพ QR ที่ export จาก Sparrow (*Wallet → Settings → Export → Output Descriptor*) → ต้องได้ `✓ MATCH`
4. เลือก **QR ที่พิมพ์ลงบนกระดาษ**: Descriptor สำหรับ Sparrow/Nunchuk หรือ zpub สำหรับ BlueWallet

> 💡 **ไม่อยากพิมพ์ Passphrase ลงเครื่องนี้?** เลือกโหมด *นำเข้า QR / Descriptor* แทน — ระบบจะใช้ข้อมูลจาก QR ที่สร้างจากแอปอื่น

### ขั้นที่ 03 — เลือกโอกาส
เลือกธีม → ใส่ชื่อผู้รับ วันที่ (รูปแบบ เดือน/วันที่/ปี) → แก้หัวข้อหรือคำอวยพรได้ตามต้องการ (เว้นวรรคเพื่อให้ตัดบรรทัดสวย) · ดูผลได้ทันทีใน **Live Preview** ด้านขวา (seed ถูกซ่อนในพรีวิวเป็นค่าเริ่มต้น)

### ขั้นที่ 04 — ผู้มอบ & ลายเซ็น
ใส่ชื่อผู้มอบ ตำแหน่ง/ความสัมพันธ์ คำลงท้าย และเลือกรูปแบบลายเซ็น

### ขั้นที่ 05 — Export
1. ตั้ง **รหัสผ่าน PDF** (≥ 8 ตัว แนะนำ ≥ 70 บิตตามมาตรวัด) — **ใช้ได้เฉพาะตัวอักษร ASCII** (a-z A-Z 0-9 สัญลักษณ์) เพื่อให้เปิดได้ทุกโปรแกรม
2. กด **⬇ สร้าง PDF เข้ารหัส** → ไฟล์ถูกบันทึกในโฟลเดอร์ `Tor Browser` (อยู่ใน RAM ถ้าไม่ได้ใช้ Persistent Storage)
3. เปิดด้วย Document Viewer ใส่รหัสผ่าน → สั่งพิมพ์ **Actual size / 100%** (ห้าม Fit to page)
4. ลบไฟล์ PDF ทิ้งหลังพิมพ์ — **รหัสผ่านปกป้องไฟล์เท่านั้น กระดาษที่พิมพ์ออกมาไม่ได้เข้ารหัส**

---

## 📐 การพับ · Folding Guide

กระดาษ A4 แบ่ง 3 ส่วนเท่ากัน (99 มม.): **ปก/คำอวยพร → Seed → Watch-only + ลายเซ็น** มีขีดบอกตำแหน่งพับที่ขอบกระดาษ และพิมพ์คำแนะนำ ① ② ไว้ตรงรอยพับ

| | หน้าเดียว — **Z-fold** (ค่าเริ่มต้น) | 2 หน้า — **พับแบบจดหมาย** |
|---|---|---|
| การพิมพ์ | ด้านเดียว ปริ้นเตอร์ทั่วไป | 2 ด้าน · **Flip on long edge** |
| ① ขีดล่าง | ทบส่วนล่างขึ้น ให้ Wallet คว่ำทับ Seed | ทบส่วนล่างขึ้น |
| ② ขีดบน | ทบปกไปด้านหลัง (ทบออก) | ทบส่วนบนลงมาทับ (ทบเข้า) |
| ด้านนอก | ปกคำอวยพร + หลังกระดาษสีขาว | ปก “A GIFT FOR …” + ปกหลัง |
| ด้านใน | Seed + Watch-only หันเข้าหากัน | คำอวยพร → “อีกเพียงหนึ่งทบ…” → Seed |

ขนาดเมื่อพับ 21 × 9.9 ซม. — ใส่ซอง **DL (11 × 22 ซม.)** หรือซองขาวมาตรฐานแนวนอนได้พอดี

---

## 🎯 ทำไมต้องใช้ · Why it matters

- **มอบ “เงินที่ดี” ได้จริง** — ผู้รับได้ Bitcoin ใน self-custody ตั้งแต่วันแรก ไม่ใช่บัตรกำนัลหรือบัญชีบนแพลตฟอร์มของคนอื่น
- **ปลอดภัยด้วย 2 ปัจจัย** — Seed บนกระดาษ + Passphrase ที่บอกด้วยวาจา ใครเก็บกระดาษได้ก็ยังใช้เหรียญไม่ได้
- **ผู้รับเริ่มต้นง่าย** — บนกระดาษมีขั้นตอนรับของขวัญ, fingerprint และ address ให้ตรวจเทียบ และ QR สำหรับดูยอดแบบ watch-only
- **ญาติ ๆ เติม sats ต่อได้** — พิมพ์ QR Address #1 เพื่อให้ร่วมสมทบได้ภายหลัง
- **เครื่องมือสอน self-custody** — ทุกขั้นตอนตรวจได้ เหมาะใช้ในเวิร์กช็อปและชุมชน Bitcoiner
- **ตรวจสอบได้ทั้งหมด** — โค้ดอ่านได้ทุกบรรทัดในไฟล์เดียว ไม่ต้องเชื่อใคร *Don't Trust, Verify.*

---

## ⚠️ Security Notes · ข้อควรรู้ก่อนมอบของขวัญ

1. **ผู้มอบรู้ Seed + Passphrase** — แนะนำให้ผู้รับย้ายเหรียญไปกระเป๋าใหม่ที่สร้างเอง (บนกระดาษมีคำแนะนำนี้แล้ว)
2. **Passphrase ต้องแข็งแรงจริง** — บนกระดาษมี fingerprint/xpub/address ใครได้กระดาษไปสามารถเดา Passphrase แบบออฟไลน์ได้ ห้ามใช้วันเกิดหรือชื่อผู้รับ ถ้าต้องการลดความเสี่ยง ให้เลือก **แยกการ์ด watch-only อีกแผ่น**
3. **อย่าพิมพ์ Passphrase ไว้กับกระดาษ** — บอกด้วยวาจา หรือส่งแยกช่องทาง/แยกเวลา
4. **ไม่มี Passphrase = กระดาษคือกุญแจเต็ม** — ระบบจะเตือนถ้าเว้นว่าง
5. **ใช้เครื่องพิมพ์ที่ไม่ต่อเน็ต** — เครื่องพิมพ์สมัยใหม่หลายรุ่นเก็บสำเนางานพิมพ์ไว้ในหน่วยความจำ
6. **ใช้ Seed ทดสอบซ้อมก่อน** — พิมพ์จริง → สแกน QR เข้า Sparrow/BlueWallet/Nunchuk → Address #1 ต้องตรงกับที่พิมพ์
7. **ใช้งานบน mainnet ด้วยความรับผิดชอบของผู้ใช้เอง** — ซอฟต์แวร์นี้ให้มา “ตามสภาพ” (ดู LICENSE)

---

## 🔐 Verify the Release · ตรวจสอบไฟล์

```bash
# 1) ตรวจ SHA-256
sha256sum seed-gift-forge.html
# ต้องได้:
# 1eef9946b04894a73c7860dc7d0b3dd48fbe1d28d1981737b7f24b3f402ae5c0  seed-gift-forge.html

# 2) ตรวจลายเซ็น PGP
gpg --import chollatis-bitcoiner-pubkey.asc
gpg --verify SHA256SUMS.asc SHA256SUMS
sha256sum -c SHA256SUMS
```

PGP key: **Chollatis Maneewong - Bitcoiner** (ed25519)
Fingerprint: `EEFC F3F0 928D 0199 BA7E 56EC 2DB5 4085 AB23 3A47`

**ตรวจไลบรารีที่ฝังอยู่ (vendored):** แต่ละบล็อกระบุแหล่งที่มาและ SHA-256 ไว้ในหัวคอมเมนต์ คัดลอกเนื้อหาในแท็ก `<script>` ไปเทียบกับไฟล์จาก npm ได้ทันที

| Library | Version | License | SHA-256 (ไฟล์ต้นฉบับ) |
|---|---|---|---|
| [jsQR](https://github.com/cozmo/jsQR) `dist/jsQR.js` | 1.4.0 | Apache-2.0 | `bc40c8a15196236b2314db0856f72ca0b49980cd5413b8c852a7349f5fee0859` |
| [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) `dist/qrcode.js` | 2.0.4 | MIT | `79ec86f82856005b1c887905cfccfcfbec3821ca61c7fd5a952faa5f778f791c` |

ส่วนอื่นทั้งหมดเขียนขึ้นใหม่ ไม่มี dependency: SHA-256/384/512, RIPEMD-160, HMAC, PBKDF2, AES, secp256k1, BIP32, Base58Check, Bech32/Bech32m, BIP380 checksum, CBOR/UR/Bytewords, BBQr, PNG decoder, Thai text shaper, PDF writer + AES-256 R6

---

## 🧪 Testing · การทดสอบ

| การทดสอบ | ผล |
|---|---|
| Crypto test vectors (SHA-2, RIPEMD-160, HMAC, PBKDF2, AES เทียบ Node.js crypto; BIP32 vector 1; BIP39 Trezor vector; BIP44/49/84/86; BIP380) | **143 / 143 ผ่าน** |
| เทียบกับไลบรารีอิสระ `@scure/bip39`, `@scure/bip32`, `@scure/btc-signer` — กระเป๋าสุ่ม 20 ชุด (ทุกจำนวนคำ รวม Passphrase ภาษาไทย) | mnemonic, fingerprint, xpub, address #1–3 **ตรงกันทั้งหมด** |
| UR จากไลบรารีอ้างอิง `@keystonehq/bc-ur-registry` → QR PNG → ถอด → address | ✅ |
| Real Entropy (SHA-256 × 1) เทียบกับเครื่องมือ BIP39-Generator เดิม | ตรงกันทุกไบต์ |
| PDF เข้ารหัส: เปิดด้วย **qpdf**, **poppler** (Document Viewer ใน Tails), **pdf.js** (Firefox/Tor Browser) | ✅ เปิดได้ด้วยรหัสถูก · ปฏิเสธรหัสผิด |
| QR บน PDF ที่ render แล้ว → สแกนกลับ | ได้ค่าเดิม (descriptor + checksum / zpub) |
| Thai shaping เทียบ HarfBuzz (3,573 cluster × 4 ฟอนต์) | ตรงกัน 100% |
| External audit รอบ 1 (ChatGPT) — 8 ข้อ + ประเด็นรอง → แก้ทั้งหมด | **22 / 22 ผ่าน** |
| External audit รอบ 2 — แกนคำนวณผ่าน 767 ข้อทดสอบของผู้ตรวจ · 8 ข้อด้านสถานะการนำเข้า → แก้ทั้งหมด | **19 / 19 ผ่าน** |
| External audit รอบ 3 — 3 ข้อ + ข้อเสนอแนะ → แก้ทั้งหมด (ดู [`AUDIT-RESPONSE.md`](AUDIT-RESPONSE.md)) | **11 / 11 ผ่าน** |
| UI end-to-end (headless) | ✅ ไม่มี error |

> ⚠️ **ยังไม่ได้ทดสอบบน Tor Browser ใน Tails จริงในสภาพแวดล้อมของผู้พัฒนา** และยังไม่ได้ทดสอบการ import เข้า BlueWallet/Nunchuk จริง — ยินดีรับรายงานผลผ่าน Issues

---

## ❓ FAQ

<details>
<summary><b>ทำไมต้องเป็น Native SegWit (BIP84)?</b></summary>
เป็นมาตรฐานที่กระเป๋าเกือบทุกตัวรองรับ ค่าธรรมเนียมต่ำ และ address ขึ้นต้น <code>bc1q</code> คุ้นเคยสำหรับมือใหม่ ส่วนโหมดนำเข้ายังรองรับ Taproot, Nested SegWit และ Legacy
</details>

<details>
<summary><b>CERT NO. และ GIFT ID คืออะไร?</b></summary>
เลขประจำใบของขวัญ คำนวณจาก SHA-256 ของ <code>โอกาส | ชื่อผู้รับ | ชื่อผู้มอบ | วันที่ | fingerprint</code> — ไม่ซ้ำกันในแต่ละชิ้น คำนวณซ้ำได้ และย้อนกลับไปหา Seed/Passphrase ไม่ได้ ใช้จดทะเบียนของขวัญของตัวเองได้โดยไม่ต้องจดข้อมูลกระเป๋า
</details>

<details>
<summary><b>อัปโหลด QR แล้วอ่านไม่ได้?</b></summary>
ใช้ไฟล์ <b>PNG</b> (อ่านได้โดยไม่ผ่าน canvas) · ครอปให้เห็นเฉพาะ QR · ถ้าเป็น UR แบบหลายเฟรม (animated) ให้คัดลอก descriptor เป็นข้อความมาวางแทน · ถ้าเป็นไฟล์ JPEG แล้ว Tor Browser แจ้งเรื่อง canvas ให้กด Allow ที่แถบ URL
</details>

<details>
<summary><b>ทำไมโหมด Raw ใช้กับลูกเต๋า 6 หน้า/ไพ่ไม่ได้?</b></summary>
การแปลงเลขฐาน 6 หรือ 52 เป็นบิตตรง ๆ แล้วตัดบิตแรก ทำให้บิตต้น ๆ ไม่สุ่มเท่ากัน (bias) จึงบังคับใช้ SHA-256 ซึ่งกระจายเอนโทรปีได้สม่ำเสมอ (ความปลอดภัยจริงยังเท่ากับเอนโทรปีต้นทาง)
</details>

<details>
<summary><b>เปิดบนมือถือได้ไหม?</b></summary>
UI รองรับจอเล็ก แต่ <b>ไม่แนะนำ</b> ให้สร้าง Seed จริงบนมือถือหรืออุปกรณ์ที่ต่ออินเทอร์เน็ต ใช้มือถือได้เพื่อทดลองด้วย Seed ทดสอบเท่านั้น
</details>

---

## 📜 License & Credits

- **Seed Gift Forge** — MIT License © 2026 Chollatis Bitcoiner (ดู [`LICENSE`](LICENSE))
- **jsQR** — Apache License 2.0 © Cosmo Wolfe · **qrcode-generator** — MIT © Kazuhiko Arase
- ฟอนต์ **Sarabun** (Cadson Demak), **Charm** (Cadson Demak), **JetBrains Mono** (JetBrains) — SIL Open Font License 1.1 (ฝังแบบ subset)
- รายละเอียดทั้งหมดอยู่ใน [`THIRD-PARTY-NOTICES.md`](THIRD-PARTY-NOTICES.md)

---

<div align="center">

**© 2026 Chollatis Bitcoiner · [learning.chontit.win](https://learning.chontit.win)**

*Don't Trust, Verify.* · Powered by Claude AI

</div>
