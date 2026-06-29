# CLAUDE.md

model: claude-opus-4-8

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 Working Agreement (ข้อตกลงการทำงาน — บังคับทุกครั้ง)

กฎเหล่านี้ใช้กับ **ทุกคำสั่ง** ไม่ใช่เฉพาะงานใหญ่ — และ override พฤติกรรม default.

### 1. Investigate-first — แยก Fact ออกจาก Opinion ให้ชัด

เมื่อได้รับคำสั่ง ทำตามลำดับนี้ทุกครั้งก่อนลงมือแก้โค้ด:

1. **ตรวจสอบ** — อ่านไฟล์ / โครงสร้าง / โค้ดที่เกี่ยวข้องจริง
2. **หาความจริง** — ยืนยันข้อเท็จจริงจาก source (โค้ด / behavior) อย่าเดา
3. **ประมวลผล + วิเคราะห์**
4. **นำเสนอ โดยแบ่ง 2 ส่วนแยกกันชัดเจน:**
   - 🔵 **Fact** — สิ่งที่ตรวจสอบแล้วเป็นจริง อ้างอิงได้ (ไฟล์ / บรรทัด)
   - 🟡 **Opinion** — ข้อดี / ข้อเสีย + แผนการแก้ที่เสนอ + ความเห็นเพิ่มเติม
5. **รอเจ้าของตัดสินใจ** — ไม่ลงมือจนกว่าจะได้ไฟเขียว

**ข้อยกเว้น:** ถ้าคำสั่งเป็นไฟเขียวในตัว ("ทำเลย" / "จัดการเลย" / "ต่อเลย" / อนุมัติแผนใน plan mode) = ตัดสินใจแล้ว → ลงมือได้ทันที ไม่ต้องวนถามซ้ำ แต่ผลลัพธ์ยังต้องรายงานแบบแยก Fact / Opinion เสมอ

### 2. โค้ดสะอาด คอมเมนต์ให้น้อย — เอาแต่เนื้อโค้ด

- **อย่าใส่คอมเมนต์ `//` เยอะ** มันรกและเปลือง token. เขียนโค้ดให้อ่านรู้เรื่องด้วยตัวมันเอง (ชื่อ variable/function สื่อความหมาย) แทนการบรรยาย
- ใส่คอมเมนต์ **เฉพาะ "why" ที่ไม่ชัดเจน** — gotcha, workaround, ข้อจำกัดของ MediaPipe/canvas, สูตรเลขที่ตีความยาก. **ห้ามบรรยาย "what" รายบรรทัด** (โค้ดบอกอยู่แล้ว)
- รักษาสไตล์เดิมของไฟล์: โทนสี ink/paper/pulse, ฟอนต์ ui-rounded / Noto Sans Thai, โครงสร้างไฟล์เดียว
- ถ้าเจ้าของสั่งให้อธิบายจุดที่แก้ "ในแชต" ก็ตอบในแชต ไม่ต้องยัดเป็นคอมเมนต์ในโค้ด

### 3. iPad / กล้อง / performance-first

กลุ่มผู้ใช้คือผู้ป่วยกายภาพที่เล่นผ่านกล้อง iPad ก่อนแตะอะไรที่กระทบ loop / การตรวจจับมือ / การวาด canvas ต้องคำนึงถึง:

- **เฟรมเรตคงที่บน iPad** — หลีกเลี่ยงงานหนักต่อเฟรม, อย่าตรวจจับมือถ้าเฟรมกล้องยังไม่อัปเดต, จับมือเท่าที่โหมดต้องการ (`numHands`)
- **รองรับการเคลื่อนไหวที่ช้ากว่าคนปกติ** — timing/threshold ต้องเผื่อผู้ป่วยที่ช้ามาก
- **เล่นต่อเนื่องนาน ๆ ต้องเสถียร** — ไม่มี memory leak (ปิด landmarker เก่าก่อนสร้างใหม่), ไม่มีคะแนนเฟ้อ

ถ้าจุดที่จะแก้กระทบ performance หรือความเสถียรในทางลบ → แจ้งทันทีตามรูปแบบข้อ 1 (แยก Fact / Opinion) ก่อนทำ

## Overview

เกมฝึกการเคลื่อนไหว (rehab) ภาษาไทย — ผู้เล่นเอื้อมมือไปแตะเป้าหมายบนจอผ่านกล้อง iPad. ตรวจจับมือด้วย **MediaPipe Tasks Vision (HandLandmarker)**.

ทั้งโปรเจกต์อยู่ใน **`index.html` ไฟล์เดียว** — ไม่มี build step, ไม่มี dependency นอกจาก MediaPipe ที่ `import` จาก jsDelivr CDN (`@mediapipe/tasks-vision@0.10.14`). รันโดยเปิดไฟล์ในเบราว์เซอร์ (ต้องมีกล้อง + HTTPS/localhost ให้ `getUserMedia` ทำงาน). โค้ดเป็น English, UI strings เป็นไทย.

## Architecture (ภายใน index.html)

ไฟล์เดียวแบ่งเป็น 3 ส่วน: `<style>` (ธีม + layout), markup (stage/HUD/menu/summary), และ `<script type="module">` (เกมทั้งหมด).

- **Config + menu** — `config = { mode, diff, players, time, ballTimeTouched }`. `DIFF` กำหนด r/margin/life/twoHandChance ต่อความยาก. `wireOpts()` ผูกปุ่มเลือกในเมนู
- **MediaPipe** — `ensureModel(numHands)` สร้าง/cache `handLandmarker` (1 คน = 2 มือ, 2 คน = 4 มือ); `ensureCamera()` เปิดกล้อง front
- **Game state** — `game` (running, startTime, duration, players, `p[]` stats, targets, pickSets). สถิติต่อผู้เล่น `p[i] = { score, reach, miss }`
- **Loop** — `loop()` รัน rAF: คิดเวลา → ตรวจจับมือ (เฉพาะเฟรมกล้องใหม่) → `updateBallMode` / `updatePickMode` → `drawHands` → `updateHud`
- **โหมด** — `reach` (แตะลูก, มี twoHand), `color2` (เขียวได้/แดงเสีย), `pick` (สุ่มชุด 4 สี แตะตามสีที่สั่ง)
- **Summary** — `endGame()` คำนวณผล; เดี่ยวแสดง accuracy = reach/(reach+miss)

## Invariants & Gotchas (พังง่ายถ้าลืม)

- **กล้อง mirror ด้วย CSS `transform: scaleX(-1)` แต่พิกัด landmark เป็น raw (unmirrored) video space.** เวลาวาดข้อความ/แบ่งฝั่งผู้เล่นต้องแปลงเอง:
  - `whichPlayer`: raw `x > W/2` = ผู้เล่น 0 (ฝั่งซ้ายที่ตามองเห็น) เพราะภาพถูก mirror
  - ข้อความ (pick banner, two-hand badge) ต้อง `setTransform`/mirror x กลับเพื่อให้ตัวอักษรไม่กลับด้าน
  - side `"L"` (raw x<0.5) โผล่ฝั่ง **ขวา** ของจอ และกลับกัน
- **กันคะแนนเฟ้อ** — target/set มี `armed` + `SPAWN_SAFE_MS`: นับการแตะได้เมื่อบริเวณนั้นปลอดนิ้วมาก่อน + พ้นช่วงปลอดการชนหลัง spawn. อย่าลบกลไกนี้
- **`numHands` เปลี่ยนต้องสร้าง landmarker ใหม่** — `ensureModel` cache ตาม `currentNumHands` และ `close()` ตัวเก่าก่อนสร้างใหม่ (กัน memory leak)
- **ตรวจจับมือเฉพาะเฟรมกล้องใหม่** (`video.currentTime !== lastVideoTime`) — เฟรมที่กล้องไม่อัปเดตให้วาดต่อด้วยโครงมือเดิม แต่ไม่ประมวลผลการชน (กันหน่วง + กันนับซ้ำ)
- **เวลาต่อลูก (`life`)** — มาจาก `effectiveLife()`: ถ้า `ballTimeTouched` ใช้ค่า slider (0 = ไม่จำกัด), ไม่งั้น default ตามความยาก. ลูก life>0 จึงวาด countdown ring; "ไม่จำกัด" ไม่วาด
- **miss accounting** — ลูก good ที่หมดเวลาต้องบวก `miss` ให้เจ้าของ (เดี่ยว=0, สองคนแยก L/R) ไม่งั้น accuracy เพี้ยน. ลูก bad (แดง) ที่หมดเวลา = หลบถูก ไม่นับ miss
- **ฟีเจอร์ที่ห้ามทำพัง** — โหมด 2 คน, การ mirror, ปุ่มเสร็จ/ยกเลิก, หน้าสรุปผล, countdown ก่อนเริ่ม

## Model guidance

โปรเจกต์ไฟล์เดียวเล็ก — ใช้ `opus` สำหรับงาน logic เกม / debug timing / performance loop. งาน docs / commit / หา-แทน string ใช้ `haiku` ได้. ล็อกเวอร์ชันแบบ reproducible ใช้ full ID เช่น `claude-opus-4-8` แทน alias.
