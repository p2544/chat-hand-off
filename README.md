# project-handoff

Skill สำหรับสร้างเอกสาร handoff อัตโนมัติเมื่อต้องการย้ายไป chat ใหม่

---

## Skill นี้มีไว้ทำอะไร

ในโปรเจคใหญ่ที่คุยกันยาวหลายร้อย turn — แชทจะเริ่มโหลดช้าและกินทรัพยากร วิธีแก้คือไปขึ้น chat ใหม่ แต่ปัญหาคือ Claude ตัวใหม่จะไม่รู้บริบท ไม่รู้กฎที่ตกลงกันไว้ ไม่รู้ว่าตัดสินใจอะไรไปแล้ว ต้องเล่าใหม่ทั้งหมด

Skill ตัวนี้แก้ปัญหานั้น — ให้ Claude สร้างไฟล์ `.md` สรุปทุกอย่างที่ Claude ตัวใหม่ต้องรู้:

- โปรไฟล์ผู้ใช้ + role structure
- กฎการสื่อสาร / ภาษา / style
- Tech stack + repos
- Architectural decisions ที่ปิดไปแล้ว (ห้าม revisit)
- งานล่าสุด + commit hashes
- Open bookmarks ที่ค้างไว้
- ข้อผิดพลาดที่เคยเกิด (เพื่อไม่ให้ทำซ้ำ)
- Day-1 instructions สำหรับ chat ใหม่

ไฟล์ที่ได้จะ portable — upload ไป chat ใหม่แล้ว Claude เริ่มงานต่อได้ทันทีโดยไม่ต้องเล่าใหม่

---

## วิธี trigger skill

Skill จะ trigger อัตโนมัติเมื่อพูดประโยคแนวนี้:

- "สรุปทุกอย่างให้หน่อย จะไปขึ้น chat ใหม่"
- "แชทนี้ยาวมาก สรุปไป chat ใหม่ที"
- "Create a handoff doc"
- "Summarize for the next chat"
- "เขียน handoff ให้ Claude ตัวต่อไป"

ไม่ต้องเรียกชื่อ skill ตรงๆ — Claude จะรู้เองว่าควรใช้

---

## วิธีติดตั้งใน Claude.ai

1. ดาวน์โหลด `project-handoff.zip` (มี SKILL.md ข้างใน)
2. เปิด Claude.ai (web หรือ desktop app)
3. คลิก **Customize** ที่เมนูซ้าย → **Skills**
4. คลิก **+** → **+ Create skill**
5. Upload ZIP file
6. เปิด toggle ของ skill เป็น **ON**

---

## วิธีติดตั้งใน Claude Code (terminal)

```bash
# unzip ไปไว้ที่ user-level skills folder
unzip project-handoff.zip -d ~/.claude/skills/

# verify
ls ~/.claude/skills/project-handoff/
# ควรเห็น: SKILL.md
```

เปิด session ใหม่ของ Claude Code — skill จะถูก discover อัตโนมัติ

---

## วิธีใช้งาน

**Step 1 — ตอน chat ปัจจุบันยาวเกินไป:**

พิมพ์ในแชท: "ช่วยสรุปทุกอย่างใน chat นี้ให้หน่อย จะไปขึ้น chat ใหม่ครับ"

Claude จะ:
1. ถามว่าครอบคลุม session ไหนถึงไหน
2. อ่าน transcript / conversation history
3. สร้างไฟล์ `.md` ในรูปแบบ handoff
4. ส่งไฟล์ให้ download

**Step 2 — เปิด chat ใหม่:**

1. ตั้งชื่อ chat ใหม่ตามต้องการ (เช่น "consult v2")
2. **ใน message แรก** → upload ไฟล์ handoff ที่ได้มา
3. พิมพ์: "นี่คือ handoff doc ของโปรเจคที่ทำต่อจาก chat เก่า · อ่านครบก่อน · ยืนยันเข้าใจ role structure ก่อนเริ่มงาน"

Claude ตัวใหม่จะอ่านแล้วเข้าใจบริบททั้งหมดทันที

---

## ใช้กับ AI ตัวอื่นได้ไหม

**ใช้ได้:** Claude Code · Gemini CLI · Cursor · GitHub Copilot · OpenAI Codex CLI · Antigravity · และ tools อื่น 30+ ตัวที่ adopt Agent Skills standard

**ใช้ไม่ได้โดยตรง (ต้อง paste content เข้าระบบของแต่ละค่าย):**
- ChatGPT GPTs — สร้าง custom GPT แล้ว paste body ของ SKILL.md เข้า "Instructions" (limit ~8,000 chars)
- Gemini Gems — สร้าง Gem ใหม่ แต่ต้อง trim เนื้อหาให้เหลือ ~4,000 chars

---

## License

ใช้งานส่วนตัว — ปรับแต่งได้ตามต้องการ
