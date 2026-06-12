# cdn-assets

CDN assets repository สำหรับรูปภาพและสื่อต่าง ๆ ของโปรเจกต์ — เสิร์ฟผ่าน
[jsDelivr](https://www.jsdelivr.com/) (free public CDN สำหรับ GitHub repo) แทนการ
copy ไฟล์เข้าไปในแต่ละ resource

- **Repo:** `ar-team-socool/cdn-assets`
- **Branch หลัก:** `main`

---

## โครงสร้าง

```
cdn-assets/
  gta-vehicles/      # รูปรถ GTA V (738 ไฟล์) — ชื่อไฟล์ = spawn name ของรถ + .png
    adder.png
    banshee.png
    ...
```

ชื่อไฟล์ใน `gta-vehicles/` ตรงกับ **vehicle spawn name** (เช่น `adder`, `banshee2`,
`zentorno`) ต่อด้วย `.png` — ดึงรูปจาก model name ได้ตรง ๆ

---

## รูปแบบ URL ของ jsDelivr

```
https://cdn.jsdelivr.net/gh/<user>/<repo>@<version>/<path>
```

| ส่วน | ค่าในโปรเจกต์นี้ |
|------|------------------|
| `<user>/<repo>` | `ar-team-socool/cdn-assets` |
| `<version>` | branch (`main`), git tag (`v1.0.0`), หรือ commit hash |
| `<path>` | path ของไฟล์ในรีโป เช่น `gta-vehicles/adder.png` |

### ตัวอย่าง

```
# ระบุ branch (อัปเดตตามรีโปทันที — เหมาะกับ dev)
https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets@main/gta-vehicles/adder.png

# ระบุ tag เวอร์ชัน (ล็อกเวอร์ชัน — เหมาะกับ production)
https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets@v1.0.0/gta-vehicles/adder.png

# ระบุ commit hash (ล็อกแน่นที่สุด)
https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets@a1b2c3d/gta-vehicles/adder.png

# ไม่ระบุเวอร์ชัน = latest tag ถ้ามี ไม่งั้น default branch
https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets/gta-vehicles/adder.png
```

> **เลือก version ให้ถูก:**
> - `@main` — ได้ไฟล์ล่าสุดเสมอ แต่ jsDelivr cache ฝั่ง branch ได้ **นานสุด 7 วัน**
>   (ดู [Cache / Purge](#cache--purge) ด้านล่าง) — รูปใหม่/รูปแก้อาจไม่ขึ้นทันที
> - `@v1.0.0` / `@<hash>` — cache แบบ **permanent (immutable)** ตอบเร็วและไม่มีวันหมดอายุ
>   เพราะ jsDelivr รู้ว่า tag/commit ไม่เปลี่ยน — **แนะนำสำหรับ production**

---

## วิธีใช้ในโปรเจกต์ (FiveM / NUI)

### NUI (HTML/CSS/JS)

```html
<img src="https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets@main/gta-vehicles/adder.png">
```

```js
const base = 'https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets@main/gta-vehicles/';
const url  = base + vehicleModel + '.png';   // vehicleModel = spawn name ของรถ

// กัน model ที่ไม่มีรูป → ใส่ placeholder ผ่าน onerror
img.onerror = () => { img.src = 'nui://ar_nui/assets/images/placeholder.png'; };
```

> NUI ของ FiveM ทำงานบน CEF (Chromium 103) ซึ่งต่อ `https://` ออกเน็ตได้ปกติ —
> เรียก jsDelivr ได้ตรงจาก NUI โดยไม่ต้องผ่าน Lua

### Lua (ส่ง URL ให้ NUI)

```lua
local CDN_BASE <const> = 'https://cdn.jsdelivr.net/gh/ar-team-socool/cdn-assets@main/gta-vehicles/'

local function vehicleImage(model)
    return CDN_BASE .. model .. '.png'
end
```

> สำหรับรูปที่ใช้ใน inventory/UI ภายในเซิร์ฟเวอร์อยู่แล้ว ให้ใช้ asset ของ `ar_nui`
> ผ่าน `nui://ar_nui/...` (เสิร์ฟ local ไม่ต้องพึ่งเน็ต) — ใช้ jsDelivr เฉพาะ asset
> ก้อนใหญ่/ใช้นอกเกม (เว็บ, Discord bot, เอกสาร) หรือรูปที่ไม่อยากแพ็คเข้า resource

---

## Cache / Purge

- jsDelivr cache ไฟล์ที่อ้างด้วย **branch** ไว้ **สูงสุด 7 วัน** ส่วนไฟล์ที่อ้างด้วย
  **tag/commit hash** cache แบบถาวร
- เพิ่มหรือแก้ไฟล์แล้วอยากให้ `@main` เห็นทันที (ไม่รอ cache หมดอายุ) ให้ purge ผ่าน:

  ```
  https://purge.jsdelivr.net/gh/ar-team-socool/cdn-assets@main/gta-vehicles/<file>.png
  ```

  เปิด URL นี้ใน browser หรือ `curl` หนึ่งครั้งเพื่อล้าง cache ของไฟล์นั้น

- **วิธีที่ดีกว่าสำหรับ production:** push เสร็จแล้ว **สร้าง git tag เวอร์ชันใหม่**
  (`v1.1.0`, `v1.2.0`, …) แล้วชี้ URL ไปที่ tag นั้น — ได้ immutable cache + คุม
  เวอร์ชันชัดเจน ไม่ต้อง purge

  ```bash
  git tag v1.1.0
  git push origin v1.1.0
  ```

---

## การเพิ่มไฟล์ใหม่

1. วางไฟล์ในโฟลเดอร์ที่เหมาะสม (เช่น `gta-vehicles/`) ตั้งชื่อให้ตรง convention
   (รถ = spawn name ตัวพิมพ์เล็ก + `.png`)
2. `git add` → `git commit` → `git push origin main`
3. เข้าถึงได้ทันทีผ่าน `@main` (อาจต้อง purge ถ้า path เดิมเคยถูก cache)
4. (production) tag เวอร์ชันใหม่แล้วชี้ URL ไปที่ tag

> ขนาดไฟล์: jsDelivr รองรับไฟล์เดี่ยวสูงสุด ~50 MB ต่อไฟล์ ซึ่งเกินพอสำหรับรูป PNG
