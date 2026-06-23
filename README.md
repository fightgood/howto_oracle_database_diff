# คู่มือการใช้งาน Database Diff
## เปรียบเทียบ Structure ฐานข้อมูล Development กับ Production ด้วย Oracle SQL Developer (v21)

> **หมายเหตุเกี่ยวกับภาพประกอบ:** ภาพในคู่มือนี้เป็น **ภาพจำลอง (mock-up)** ที่วาดขึ้นเพื่ออธิบายตำแหน่งและหน้าที่ของแต่ละองค์ประกอบใน Wizard เท่านั้น หน้าจอจริงของ Oracle SQL Developer อาจมีสี/การจัดวางต่างไปเล็กน้อยตาม Theme และ Patch ที่ติดตั้ง แต่ชื่อฟิลด์ ลำดับขั้นตอน และตรรกะการทำงานจะตรงกับเวอร์ชัน 21.x

---

## สารบัญ

1. [ภาพรวมและวัตถุประสงค์](#1-ภาพรวมและวัตถุประสงค์)
2. [สิ่งที่ต้องเตรียมก่อนเริ่มงาน](#2-สิ่งที่ต้องเตรียมก่อนเริ่มงาน)
3. [การเปิดใช้งาน Database Diff Wizard](#3-การเปิดใช้งาน-database-diff-wizard)
4. [Step 1: Source and Destination Connections](#4-step-1-source-and-destination-connections)
5. [Step 2: Object Types to Compare](#5-step-2-object-types-to-compare)
6. [Step 3: Specify Objects](#6-step-3-specify-objects)
7. [Step 4: Summary](#7-step-4-summary)
8. [การอ่านผลลัพธ์ Diff Report](#8-การอ่านผลลัพธ์-diff-report)
9. [การดูรายละเอียดความแตกต่างระดับ Column/Source Code](#9-การดูรายละเอียดความแตกต่างระดับ-columnsource-code)
10. [การ Generate Sync Script (ALTER/CREATE)](#10-การ-generate-sync-script-altercreate)
11. [ข้อควรระวังเมื่อใช้งานกับ Production](#11-ข้อควรระวังเมื่อใช้งานกับ-production)
12. [ปัญหาที่พบบ่อยและวิธีแก้ไข](#12-ปัญหาที่พบบ่อยและวิธีแก้ไข)
13. [สรุปขั้นตอนแบบย่อ (Quick Reference)](#13-สรุปขั้นตอนแบบย่อ-quick-reference)

---

## 1. ภาพรวมและวัตถุประสงค์

**Database Diff** เป็นฟีเจอร์มาตรฐานที่มาพร้อมกับ Oracle SQL Developer (ไม่ต้องติดตั้งเพิ่ม ไม่มีค่าใช้จ่ายเพิ่มเติม) ใช้สำหรับเปรียบเทียบ **Structure ระดับ Object** ของ Schema สองฝั่ง เช่น Development กับ Production โดย**ไม่เปรียบเทียบข้อมูล (Data) ในตาราง** เปรียบเทียบเฉพาะ Definition/DDL เท่านั้น

วัตถุประสงค์หลักของการใช้งาน:

- ตรวจสอบว่า Object ใดถูกสร้างเพิ่มหรือถูกลบไปในแต่ละฝั่ง
- ตรวจสอบว่า Table/View ใดมี Column เปลี่ยนแปลง (เพิ่ม/ลด/เปลี่ยน Data Type/Nullable/Default)
- ตรวจสอบว่า Function/Procedure/Package มีการแก้ไข Source Code ที่ยังไม่ได้ Deploy ไปอีกฝั่ง
- สร้างสคริปต์สำหรับ Sync โครงสร้างให้ตรงกัน (ใช้เป็นจุดอ้างอิง ก่อนนำไป Review เอง)

---

## 2. สิ่งที่ต้องเตรียมก่อนเริ่มงาน

| รายการ | รายละเอียด |
|---|---|
| **Connection** | ต้องมี Database Connection ของทั้ง Dev และ Production ที่สร้างไว้ใน SQL Developer แล้ว (เห็นใน Connections Navigator ทางซ้าย) ถ้ามีแค่ Connection เดียว เมนู Database Diff จะถูก Disable |
| **สิทธิ์ผู้ใช้ (Privilege)** | แนะนำให้ขอ User แบบ **Read-only** ฝั่ง Production เพียงพอสำหรับการ Diff เช่น สิทธิ์ `SELECT` บน Data Dictionary หรือ Role `SELECT_CATALOG_ROLE` ถ้าต้องดู Object ของ Schema อื่นที่ไม่ใช่ Schema ของ User เอง ต้องมีสิทธิ์เพิ่มเติม (เช่น `SELECT ANY DICTIONARY`) |
| **Network/VPN** | เครื่องที่รัน SQL Developer ต้อง Connect ไปยัง Dev และ Production ได้พร้อมกันในเวลาเดียวกัน |
| **เวลาที่เหมาะสม** | หาก Schema มีขนาดใหญ่ การ Compare อาจใช้เวลาหลายนาที ควรเลือกเวลาที่ไม่กระทบช่วง Peak การใช้งานของระบบ Production |
| **Backup/Snapshot (ทางเลือก)** | หากต้องการ Generate Script ไปรันจริง ควรมี Backup หรือจุด Rollback ของ Production ก่อนเสมอ |

---

## 3. การเปิดใช้งาน Database Diff Wizard

จากเมนูหลักของโปรแกรม ไปที่ **Tools > Database Diff...**

<svg viewBox="0 0 800 260" xmlns="http://www.w3.org/2000/svg" font-family="Segoe UI, Arial, sans-serif">
  <rect x="0" y="0" width="800" height="260" fill="#ffffff" stroke="#9aa5b1" stroke-width="1"/>
  <rect x="0" y="0" width="800" height="34" fill="#2f3b52"/>
  <text x="14" y="22" fill="#ffffff" font-size="14">Oracle SQL Developer</text>
  <rect x="0" y="34" width="800" height="30" fill="#eef1f5" stroke="#d3d9e0"/>
  <text x="14" y="54" font-size="13" fill="#333">File</text>
  <text x="64" y="54" font-size="13" fill="#333">Edit</text>
  <text x="114" y="54" font-size="13" fill="#333">View</text>
  <text x="170" y="54" font-size="13" fill="#333">Navigate</text>
  <text x="250" y="54" font-size="13" fill="#333">Run</text>
  <text x="300" y="54" font-size="13" fill="#333">Versioning</text>
  <rect x="385" y="36" width="60" height="26" fill="#cfe0ff"/>
  <text x="395" y="54" font-size="13" font-weight="bold" fill="#10366b">Tools</text>
  <text x="460" y="54" font-size="13" fill="#333">Window</text>
  <text x="525" y="54" font-size="13" fill="#333">Help</text>
  <rect x="385" y="62" width="230" height="190" fill="#ffffff" stroke="#9aa5b1"/>
  <text x="398" y="84" font-size="12" fill="#333">Preferences...</text>
  <line x1="386" y1="92" x2="613" y2="92" stroke="#e5e7eb"/>
  <text x="398" y="110" font-size="12" fill="#333">SQL*Plus</text>
  <line x1="386" y1="118" x2="613" y2="118" stroke="#e5e7eb"/>
  <text x="398" y="136" font-size="12" fill="#333">Database Copy...</text>
  <line x1="386" y1="144" x2="613" y2="144" stroke="#e5e7eb"/>
  <rect x="386" y="148" width="227" height="24" fill="#2f6fed"/>
  <text x="398" y="165" font-size="12" font-weight="bold" fill="#ffffff">Database Diff...</text>
  <line x1="386" y1="172" x2="613" y2="172" stroke="#e5e7eb"/>
  <text x="398" y="190" font-size="12" fill="#333">Database Export...</text>
  <line x1="386" y1="198" x2="613" y2="198" stroke="#e5e7eb"/>
  <text x="398" y="216" font-size="12" fill="#333">Compare/Synchronize...</text>
  <polygon points="370,158 386,150 386,166" fill="#2f6fed"/>
</svg>

**คำอธิบาย:**

- ถ้าไม่เห็นเมนู **Database Diff...** หรือเมนูเป็นสีจาง (Disable) ให้ตรวจสอบว่ามี Connection อย่างน้อย 2 รายการอยู่ใน Connections Navigator แล้ว
- บางเวอร์ชัน/บาง Build อาจมีเมนูนี้ซ้อนอยู่ใต้ **Database Copy** หรือเรียงลำดับต่างไปเล็กน้อย แต่ชื่อเมนูจะเป็น "Database Diff..." เสมอ

เมื่อคลิกแล้ว จะเปิดหน้าต่าง **Database Diff Wizard** ซึ่งมีทั้งหมด 4 ขั้นตอน (Step) ก่อนจะเริ่มเปรียบเทียบจริง

---

## 4. Step 1: Source and Destination Connections

<svg viewBox="0 0 800 420" xmlns="http://www.w3.org/2000/svg" font-family="Segoe UI, Arial, sans-serif">
  <rect x="0" y="0" width="800" height="420" fill="#ffffff" stroke="#9aa5b1"/>
  <rect x="0" y="0" width="800" height="40" fill="#2f3b52"/>
  <text x="16" y="26" fill="#ffffff" font-size="15" font-weight="bold">Database Diff Wizard</text>
  <text x="650" y="26" fill="#c8d3e6" font-size="12">Step 1 of 4</text>
  <text x="16" y="62" font-size="13" fill="#555">Select the Source connection/schema and the Destination connection/schema to compare.</text>

  <text x="40" y="110" font-size="13" fill="#222">Source Connection:</text>
  <rect x="230" y="94" width="320" height="26" fill="#f4f6f8" stroke="#9aa5b1"/>
  <text x="240" y="112" font-size="12" fill="#10366b">DEV_CONNECTION</text>
  <polygon points="535,100 545,100 540,110" fill="#555"/>

  <text x="40" y="156" font-size="13" fill="#222">Source Schema:</text>
  <rect x="230" y="140" width="320" height="26" fill="#f4f6f8" stroke="#9aa5b1"/>
  <text x="240" y="158" font-size="12" fill="#10366b">APP_DEV</text>
  <polygon points="535,146 545,146 540,156" fill="#555"/>

  <line x1="40" y1="190" x2="760" y2="190" stroke="#e5e7eb"/>

  <text x="40" y="226" font-size="13" fill="#222">Destination Connection:</text>
  <rect x="230" y="210" width="320" height="26" fill="#f4f6f8" stroke="#9aa5b1"/>
  <text x="240" y="228" font-size="12" fill="#a3221c">PROD_CONNECTION</text>
  <polygon points="535,216 545,216 540,226" fill="#555"/>

  <text x="40" y="272" font-size="13" fill="#222">Destination Schema:</text>
  <rect x="230" y="256" width="320" height="26" fill="#f4f6f8" stroke="#9aa5b1"/>
  <text x="240" y="274" font-size="12" fill="#a3221c">APP_PROD</text>
  <polygon points="535,262 545,262 540,272" fill="#555"/>

  <rect x="40" y="312" width="700" height="50" fill="#fff7e6" stroke="#e3c576"/>
  <text x="52" y="332" font-size="11" fill="#8a6d1f">Tip: Source = ฝั่งที่ถือว่าเป็น "ของจริง/ต้นแบบ" (เช่น Dev ที่แก้ไขล่าสุด)</text>
  <text x="52" y="348" font-size="11" fill="#8a6d1f">Destination = ฝั่งที่จะถูกเทียบว่าขาด/เกิน/ต่างจาก Source อะไรบ้าง</text>

  <line x1="0" y1="378" x2="800" y2="378" stroke="#d3d9e0"/>
  <text x="20" y="404" font-size="12" fill="#888">Help</text>
  <rect x="560" y="386" width="60" height="24" fill="#e5e7eb" stroke="#aab2bd"/>
  <text x="568" y="402" font-size="11" fill="#999">&lt; Back</text>
  <rect x="628" y="386" width="60" height="24" fill="#2f6fed"/>
  <text x="638" y="402" font-size="11" fill="#fff">Next &gt;</text>
  <rect x="696" y="386" width="46" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="706" y="402" font-size="11" fill="#555">Finish</text>
  <rect x="750" y="386" width="40" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="758" y="402" font-size="11" fill="#555">Cancel</text>
</svg>

**คำอธิบายแต่ละช่อง:**

| ช่อง (Field) | ความหมาย | คำแนะนำการเลือก |
|---|---|---|
| **Source Connection** | Connection ของฝั่งที่ใช้เป็น "ต้นแบบ" ในการเทียบ | เลือก Connection ของ **Development** เพราะปกติ Dev คือฝั่งที่มีการแก้ไขล่าสุด |
| **Source Schema** | ชื่อ Schema/User ฝั่ง Source ที่จะดึง Object มาเทียบ | ต้องเป็น Schema ที่เก็บ Object ของแอปพลิเคชันจริง (เช่น `APP_DEV`) ไม่ใช่ Schema ระบบ เช่น `SYS`/`SYSTEM` |
| **Destination Connection** | Connection ของฝั่งที่จะถูกตรวจสอบว่าต่างจาก Source อย่างไร | เลือก Connection ของ **Production** |
| **Destination Schema** | ชื่อ Schema/User ฝั่ง Destination | ควรเป็นชื่อ Schema ที่สอดคล้องกับ Source (เช่น `APP_PROD`) — ชื่อ Schema ทั้งสองฝั่งไม่จำเป็นต้องเหมือนกัน แต่ต้องเก็บ Object กลุ่มเดียวกัน |

> **ข้อสังเกต:** ปุ่ม **Next >** จะ Enable ก็ต่อเมื่อเลือกครบทั้ง 4 ช่องแล้ว และทั้งสอง Connection ต้องสามารถเชื่อมต่อได้จริง (Login สำเร็จ) ไม่เช่นนั้นระบบจะแจ้ง Error ก่อนไปขั้นตอนถัดไป

จากนั้นกดปุ่ม **Next >**

---

## 5. Step 2: Object Types to Compare

<svg viewBox="0 0 800 420" xmlns="http://www.w3.org/2000/svg" font-family="Segoe UI, Arial, sans-serif">
  <rect x="0" y="0" width="800" height="420" fill="#ffffff" stroke="#9aa5b1"/>
  <rect x="0" y="0" width="800" height="40" fill="#2f3b52"/>
  <text x="16" y="26" fill="#ffffff" font-size="15" font-weight="bold">Database Diff Wizard</text>
  <text x="650" y="26" fill="#c8d3e6" font-size="12">Step 2 of 4</text>
  <text x="16" y="62" font-size="13" fill="#555">Select the object types you want to include in the comparison.</text>

  <g font-size="13" fill="#222">
    <rect x="50" y="86" width="16" height="16" fill="#2f6fed"/><text x="58" y="98" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="99">Tables</text>
    <rect x="50" y="116" width="16" height="16" fill="#2f6fed"/><text x="58" y="128" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="129">Views</text>
    <rect x="50" y="146" width="16" height="16" fill="#2f6fed"/><text x="58" y="158" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="159">Indexes</text>
    <rect x="50" y="176" width="16" height="16" fill="#2f6fed"/><text x="58" y="188" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="189">Sequences</text>
    <rect x="50" y="206" width="16" height="16" fill="#2f6fed"/><text x="58" y="218" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="219">Synonyms</text>
    <rect x="50" y="236" width="16" height="16" fill="#2f6fed"/><text x="58" y="248" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="249">Triggers</text>
    <rect x="50" y="266" width="16" height="16" fill="#2f6fed"/><text x="58" y="278" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="78" y="279">Database Links</text>

    <rect x="320" y="86" width="16" height="16" fill="#2f6fed"/><text x="328" y="98" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="348" y="99">Procedures</text>
    <rect x="320" y="116" width="16" height="16" fill="#2f6fed"/><text x="328" y="128" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="348" y="129">Functions</text>
    <rect x="320" y="146" width="16" height="16" fill="#2f6fed"/><text x="328" y="158" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="348" y="159">Package Specs</text>
    <rect x="320" y="176" width="16" height="16" fill="#2f6fed"/><text x="328" y="188" fill="#fff" font-size="11" text-anchor="middle">✓</text>
    <text x="348" y="189">Package Bodies</text>
    <rect x="320" y="206" width="16" height="16" fill="none" stroke="#9aa5b1"/>
    <text x="348" y="219" fill="#999">Types</text>
    <rect x="320" y="236" width="16" height="16" fill="none" stroke="#9aa5b1"/>
    <text x="348" y="249" fill="#999">Materialized Views</text>
    <rect x="320" y="266" width="16" height="16" fill="none" stroke="#9aa5b1"/>
    <text x="348" y="279" fill="#999">Materialized View Logs</text>
  </g>

  <rect x="560" y="86" width="190" height="200" fill="#f4f6f8" stroke="#d3d9e0"/>
  <text x="572" y="106" font-size="11" fill="#555">เลือกทั้งหมด:</text>
  <rect x="572" y="116" width="80" height="22" fill="#ffffff" stroke="#aab2bd"/>
  <text x="582" y="131" font-size="10" fill="#444">Select All</text>
  <rect x="658" y="116" width="80" height="22" fill="#ffffff" stroke="#aab2bd"/>
  <text x="668" y="131" font-size="10" fill="#444">Deselect All</text>
  <text x="572" y="160" font-size="10" fill="#888">งานเทียบ Structure ทั่วไป</text>
  <text x="572" y="174" font-size="10" fill="#888">แนะนำติ๊กทุกตัวยกเว้น</text>
  <text x="572" y="188" font-size="10" fill="#888">Materialized View ถ้าไม่ได้ใช้งาน</text>

  <line x1="0" y1="378" x2="800" y2="378" stroke="#d3d9e0"/>
  <text x="20" y="404" font-size="12" fill="#888">Help</text>
  <rect x="560" y="386" width="60" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="572" y="402" font-size="11" fill="#555">&lt; Back</text>
  <rect x="628" y="386" width="60" height="24" fill="#2f6fed"/>
  <text x="638" y="402" font-size="11" fill="#fff">Next &gt;</text>
  <rect x="696" y="386" width="46" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="706" y="402" font-size="11" fill="#555">Finish</text>
  <rect x="750" y="386" width="40" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="758" y="402" font-size="11" fill="#555">Cancel</text>
</svg>

**ความหมายของแต่ละ Object Type ที่เลือกได้ (Checkbox):**

| Object Type | คือ Object อะไร | ทำไมควรเลือกในงานเทียบ Dev/Prod |
|---|---|---|
| **Tables** | ตารางข้อมูล รวมถึง Column, Data Type, Constraint, Default Value | สำคัญที่สุด เพราะการเปลี่ยน Structure ตารางมีผลกระทบต่อแอปพลิเคชันโดยตรง |
| **Views** | View ที่สร้างจาก Query | ใช้ตรวจว่า Logic การดึงข้อมูลของ View ทั้งสองฝั่งตรงกันหรือไม่ |
| **Indexes** | Index ของตาราง (Unique/Non-unique, Column ที่ใช้ Index) | ตรวจ Performance-related object ที่อาจถูกลืม Deploy |
| **Sequences** | ตัวให้เลขรันอัตโนมัติ (Auto-increment) | ตรวจค่า Start/Increment ที่อาจตั้งต่างกันโดยไม่ได้ตั้งใจ |
| **Synonyms** | ชื่อ Alias ที่ใช้อ้างอิง Object อื่น (อาจอยู่ Schema อื่น) | สำคัญถ้าระบบมีการเรียกข้าม Schema |
| **Triggers** | Code ที่รันอัตโนมัติเมื่อมีการ Insert/Update/Delete | ตรวจว่า Trigger Enable/Disable หรือ Logic ตรงกันหรือไม่ |
| **Database Links** | การเชื่อมต่อไปยังฐานข้อมูลอื่น | ตรวจว่ามี Link หลงเหลือหรือขาดไปหรือไม่ |
| **Procedures / Functions** | PL/SQL Code แบบ Standalone | ตรวจว่า Source Code Deploy ตรงกันหรือไม่ (เทียบทีละบรรทัด) |
| **Package Specs / Package Bodies** | Package (กลุ่มของ Procedure/Function) แยกเป็นส่วน Spec (Interface) และ Body (Logic จริง) | มักพบกรณี Deploy Body ไม่ครบ หรือ Spec ไม่ตรงกับ Body |
| **Types** | Object Type/Custom Data Type ที่ผู้ใช้สร้างขึ้น | เลือกถ้าระบบมีการใช้ Object-Relational Type |
| **Materialized Views / Materialized View Logs** | View ที่เก็บข้อมูลจริงและ Log การเปลี่ยนแปลงสำหรับ Refresh | เลือกเฉพาะถ้าระบบมีการใช้งานจริง เพราะมักมีขนาดใหญ่ทำให้ Compare ช้า |

> **คำแนะนำ:** ถ้าต้องการเทียบทุกอย่างครั้งแรกเพื่อสำรวจภาพรวม ให้ติ๊กทุกตัว (**Select All**) แต่ถ้าต้องการตรวจประเด็นเฉพาะเจาะจง (เช่น สงสัยว่า Function บางตัว Deploy ไม่ครบ) ให้ติ๊กเฉพาะ Object Type ที่เกี่ยวข้อง จะช่วยให้ผลลัพธ์อ่านง่ายขึ้นและ Compare เร็วขึ้น

กดปุ่ม **Next >**

---

## 6. Step 3: Specify Objects

<svg viewBox="0 0 800 420" xmlns="http://www.w3.org/2000/svg" font-family="Segoe UI, Arial, sans-serif">
  <rect x="0" y="0" width="800" height="420" fill="#ffffff" stroke="#9aa5b1"/>
  <rect x="0" y="0" width="800" height="40" fill="#2f3b52"/>
  <text x="16" y="26" fill="#ffffff" font-size="15" font-weight="bold">Database Diff Wizard</text>
  <text x="650" y="26" fill="#c8d3e6" font-size="12">Step 3 of 4</text>
  <text x="16" y="62" font-size="13" fill="#555">Filter the specific objects to include for each object type (optional).</text>

  <rect x="40" y="80" width="120" height="26" fill="#2f6fed"/>
  <text x="56" y="98" font-size="12" fill="#fff">Tables</text>
  <rect x="160" y="80" width="120" height="26" fill="#f4f6f8" stroke="#d3d9e0"/>
  <text x="184" y="98" font-size="12" fill="#666">Views</text>
  <rect x="280" y="80" width="120" height="26" fill="#f4f6f8" stroke="#d3d9e0"/>
  <text x="296" y="98" font-size="12" fill="#666">Functions</text>

  <text x="40" y="132" font-size="12" fill="#222">Lookup:</text>
  <rect x="100" y="118" width="200" height="22" fill="#ffffff" stroke="#9aa5b1"/>
  <text x="108" y="134" font-size="11" fill="#10366b">EMP%</text>

  <text x="40" y="158" font-size="11" fill="#888">Available Objects</text>
  <rect x="40" y="166" width="280" height="160" fill="#ffffff" stroke="#9aa5b1"/>
  <text x="52" y="184" font-size="11" fill="#333">CUSTOMER</text>
  <text x="52" y="202" font-size="11" fill="#333">DEPARTMENT</text>
  <text x="52" y="220" font-size="11" fill="#333">ORDER_ITEM</text>
  <text x="52" y="238" font-size="11" fill="#333">PAYMENT_LOG</text>

  <polygon points="335,200 365,200 365,190 385,210 365,230 365,220 335,220" fill="#2f6fed"/>
  <text x="335" y="252" font-size="9" fill="#888">Move &gt;&gt;</text>

  <text x="420" y="158" font-size="11" fill="#888">Selected Objects</text>
  <rect x="420" y="166" width="280" height="160" fill="#eef4ff" stroke="#9aa5b1"/>
  <text x="432" y="184" font-size="11" fill="#10366b">EMPLOYEE</text>
  <text x="432" y="202" font-size="11" fill="#10366b">EMP_BONUS</text>
  <text x="432" y="220" font-size="11" fill="#10366b">EMP_HISTORY</text>

  <rect x="40" y="338" width="660" height="30" fill="#fff7e6" stroke="#e3c576"/>
  <text x="50" y="357" font-size="11" fill="#8a6d1f">ถ้าไม่เลือกรายการใดใน "Selected Objects" ระบบจะถือว่าเปรียบเทียบ "ทุก Object" ของชนิดนั้นโดยอัตโนมัติ</text>

  <line x1="0" y1="378" x2="800" y2="378" stroke="#d3d9e0"/>
  <text x="20" y="404" font-size="12" fill="#888">Help</text>
  <rect x="560" y="386" width="60" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="572" y="402" font-size="11" fill="#555">&lt; Back</text>
  <rect x="628" y="386" width="60" height="24" fill="#2f6fed"/>
  <text x="638" y="402" font-size="11" fill="#fff">Next &gt;</text>
  <rect x="696" y="386" width="46" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="706" y="402" font-size="11" fill="#555">Finish</text>
  <rect x="750" y="386" width="40" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="758" y="402" font-size="11" fill="#555">Cancel</text>
</svg>

**คำอธิบายแต่ละองค์ประกอบ:**

| องค์ประกอบ | หน้าที่ |
|---|---|
| **แท็บด้านบน (Tables / Views / Functions / ...)** | จะมี 1 แท็บต่อ Object Type ที่ติ๊กไว้ในขั้นตอนก่อนหน้า ใช้สลับไป Filter เฉพาะ Object ของแต่ละชนิด |
| **Lookup** | ช่องค้นหาแบบ Pattern โดยใช้ `%` แทนข้อความใดๆ เช่น พิมพ์ `EMP%` เพื่อกรองเฉพาะ Object ที่ชื่อขึ้นต้นด้วย EMP — มีประโยชน์มากเมื่อ Schema มี Object หลักร้อย/พัน |
| **Available Objects (ซ้าย)** | รายชื่อ Object ทั้งหมดของชนิดนั้นที่พบในฝั่ง Source ซึ่งยังไม่ถูกเลือก |
| **ปุ่มลูกศร (Move >> / Move <<)** | ใช้ย้าย Object จากซ้าย (Available) ไปขวา (Selected) หรือย้อนกลับ — ดับเบิลคลิกที่รายการก็สามารถย้ายได้เช่นกัน |
| **Selected Objects (ขวา)** | รายชื่อ Object ที่จะถูกนำไปเปรียบเทียบจริงในรอบนี้ |

> **กรณีพิเศษที่ควรรู้:** หาก**ไม่เลือกอะไรเลย**ใน Selected Objects ของ Object Type หนึ่งๆ ระบบจะตีความว่า "เทียบทุกตัว" ของชนิดนั้นโดยอัตโนมัติ ดังนั้นถ้าต้องการเทียบทั้ง Schema แบบครอบคลุม **สามารถข้าม Step นี้ได้เลยโดยไม่ต้องเลือกอะไร** แล้วกด Next ต่อได้ทันที — Step นี้มีไว้สำหรับกรณีที่ต้องการ "โฟกัสเฉพาะบาง Object" เท่านั้น

กดปุ่ม **Next >**

---

## 7. Step 4: Summary

<svg viewBox="0 0 800 380" xmlns="http://www.w3.org/2000/svg" font-family="Segoe UI, Arial, sans-serif">
  <rect x="0" y="0" width="800" height="380" fill="#ffffff" stroke="#9aa5b1"/>
  <rect x="0" y="0" width="800" height="40" fill="#2f3b52"/>
  <text x="16" y="26" fill="#ffffff" font-size="15" font-weight="bold">Database Diff Wizard</text>
  <text x="650" y="26" fill="#c8d3e6" font-size="12">Step 4 of 4</text>
  <text x="16" y="62" font-size="13" fill="#555">Review your selections, then click Finish to run the comparison.</text>

  <rect x="40" y="84" width="720" height="260" fill="#f9fafb" stroke="#d3d9e0"/>
  <text x="56" y="108" font-size="12" fill="#333" font-weight="bold">Source:</text>
  <text x="160" y="108" font-size="12" fill="#10366b">DEV_CONNECTION  /  Schema: APP_DEV</text>

  <text x="56" y="132" font-size="12" fill="#333" font-weight="bold">Destination:</text>
  <text x="160" y="132" font-size="12" fill="#a3221c">PROD_CONNECTION  /  Schema: APP_PROD</text>

  <text x="56" y="160" font-size="12" fill="#333" font-weight="bold">Object Types:</text>
  <text x="160" y="160" font-size="11" fill="#444">Tables, Views, Indexes, Sequences, Synonyms,</text>
  <text x="160" y="176" font-size="11" fill="#444">Triggers, Procedures, Functions, Package Specs,</text>
  <text x="160" y="192" font-size="11" fill="#444">Package Bodies</text>

  <text x="56" y="218" font-size="12" fill="#333" font-weight="bold">Filtered Objects:</text>
  <text x="160" y="218" font-size="11" fill="#444">Tables → EMPLOYEE, EMP_BONUS, EMP_HISTORY</text>
  <text x="160" y="234" font-size="11" fill="#444">Views, Indexes, ... → All Objects</text>

  <text x="56" y="266" font-size="12" fill="#333" font-weight="bold">Note:</text>
  <text x="160" y="266" font-size="11" fill="#444">การเปรียบเทียบจะใช้เวลาขึ้นกับขนาด Schema</text>
  <text x="160" y="282" font-size="11" fill="#444">และจำนวน Object ที่เลือกไว้</text>

  <line x1="0" y1="346" x2="800" y2="346" stroke="#d3d9e0"/>
  <text x="20" y="372" font-size="12" fill="#888">Help</text>
  <rect x="560" y="354" width="60" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="572" y="370" font-size="11" fill="#555">&lt; Back</text>
  <rect x="628" y="354" width="60" height="24" fill="#e5e7eb" stroke="#aab2bd"/>
  <text x="636" y="370" font-size="11" fill="#999">Next &gt;</text>
  <rect x="696" y="354" width="46" height="24" fill="#2f6fed"/>
  <text x="704" y="370" font-size="11" fill="#fff">Finish</text>
  <rect x="750" y="354" width="40" height="24" fill="#f4f6f8" stroke="#aab2bd"/>
  <text x="758" y="370" font-size="11" fill="#555">Cancel</text>
</svg>

**คำอธิบาย:**

หน้านี้เป็นเพียงหน้า**สรุปทบทวน**ก่อนรันจริง ไม่มีช่องให้กรอกเพิ่ม — ให้ตรวจทานว่า Source/Destination/Object Types/Filtered Objects ที่เลือกไว้ถูกต้องตามที่ต้องการหรือไม่ หากพบว่าผิด ให้กด **< Back** เพื่อย้อนกลับไปแก้ไขขั้นตอนก่อนหน้าได้

เมื่อตรวจสอบครบถ้วนแล้ว ให้กดปุ่ม **Finish** ระบบจะเริ่มกระบวนการเปรียบเทียบ (อาจใช้เวลาตั้งแต่ไม่กี่วินาทีถึงหลายนาที ขึ้นกับจำนวน Object และความเร็ว Network ไปยัง Production)

---

## 8. การอ่านผลลัพธ์ Diff Report

หลังจาก Compare เสร็จ จะเปิดหน้าต่าง **Diff Report** ขึ้นมาแสดงผลเป็นรายการ Object พร้อมสถานะ

<svg viewBox="0 0 800 460" xmlns="http://www.w3.org/2000/svg" font-family="Segoe UI, Arial, sans-serif">
  <rect x="0" y="0" width="800" height="460" fill="#ffffff" stroke="#9aa5b1"/>
  <rect x="0" y="0" width="800" height="36" fill="#2f3b52"/>
  <text x="14" y="24" fill="#ffffff" font-size="14" font-weight="bold">Diff Report : APP_DEV vs APP_PROD</text>

  <rect x="0" y="36" width="800" height="32" fill="#eef1f5" stroke="#d3d9e0"/>
  <circle cx="26" cy="52" r="10" fill="#2eb872"/>
  <text x="22" y="56" font-size="10" fill="#fff" text-anchor="middle">SQL</text>
  <text x="46" y="56" font-size="10" fill="#666">Generate Diff Script</text>
  <rect x="170" y="44" width="14" height="14" fill="none" stroke="#9aa5b1"/>
  <text x="190" y="55" font-size="11" fill="#555">Show Equal Objects</text>
  <text x="340" y="55" font-size="11" fill="#888">🔍 Filter:</text>
  <rect x="390" y="44" width="160" height="18" fill="#fff" stroke="#aab2bd"/>

  <rect x="0" y="68" width="800" height="26" fill="#dfe4ea"/>
  <text x="16" y="86" font-size="11" font-weight="bold" fill="#333">Object Name</text>
  <text x="260" y="86" font-size="11" font-weight="bold" fill="#333">Type</text>
  <text x="400" y="86" font-size="11" font-weight="bold" fill="#333">Status</text>
  <text x="560" y="86" font-size="11" font-weight="bold" fill="#333">Action</text>

  <rect x="0" y="94" width="800" height="30" fill="#ffe5e3"/>
  <text x="16" y="114" font-size="12" fill="#7a1f1a">DEPARTMENT</text>
  <text x="260" y="114" font-size="12" fill="#7a1f1a">TABLE</text>
  <text x="400" y="114" font-size="12" fill="#7a1f1a" font-weight="bold">Only in Source (Dev)</text>
  <text x="560" y="114" font-size="11" fill="#7a1f1a">Create in Prod</text>

  <rect x="0" y="124" width="800" height="30" fill="#fff3cd"/>
  <text x="16" y="144" font-size="12" fill="#7a5d00">EMPLOYEE</text>
  <text x="260" y="144" font-size="12" fill="#7a5d00">TABLE</text>
  <text x="400" y="144" font-size="12" fill="#7a5d00" font-weight="bold">Different</text>
  <text x="560" y="144" font-size="11" fill="#7a5d00">Alter columns</text>

  <rect x="0" y="154" width="800" height="30" fill="#dceeff"/>
  <text x="16" y="174" font-size="12" fill="#1a4a7a">BONUS_VIEW</text>
  <text x="260" y="174" font-size="12" fill="#1a4a7a">VIEW</text>
  <text x="400" y="174" font-size="12" fill="#1a4a7a" font-weight="bold">Only in Destination (Prod)</text>
  <text x="560" y="174" font-size="11" fill="#1a4a7a">Review before drop</text>

  <rect x="0" y="184" width="800" height="30" fill="#fff3cd"/>
  <text x="16" y="204" font-size="12" fill="#7a5d00">CALC_SALARY</text>
  <text x="260" y="204" font-size="12" fill="#7a5d00">FUNCTION</text>
  <text x="400" y="204" font-size="12" fill="#7a5d00" font-weight="bold">Different</text>
  <text x="560" y="204" font-size="11" fill="#7a5d00">Replace source code</text>

  <rect x="0" y="214" width="800" height="26" fill="#f3f5f7"/>
  <text x="16" y="232" font-size="11" fill="#9aa0a6">EMP_PK</text>
  <text x="260" y="232" font-size="11" fill="#9aa0a6">INDEX</text>
  <text x="400" y="232" font-size="11" fill="#9aa0a6">Equal</text>
  <text x="560" y="232" font-size="11" fill="#bbb">— (ไม่ต้องทำอะไร)</text>

  <rect x="40" y="260" width="720" height="180" fill="#f9fafb" stroke="#d3d9e0"/>
  <text x="54" y="282" font-size="11" font-weight="bold" fill="#333">รายละเอียดของ Object ที่เลือก (EMPLOYEE) — Source vs Destination</text>
  <text x="54" y="306" font-size="11" fill="#666">SOURCE (Dev)</text>
  <text x="420" y="306" font-size="11" fill="#666">DESTINATION (Prod)</text>
  <text x="54" y="326" font-size="11" fill="#333">EMP_ID        NUMBER(10)</text>
  <text x="420" y="326" font-size="11" fill="#333">EMP_ID        NUMBER(10)</text>
  <text x="54" y="346" font-size="11" fill="#a3221c" font-weight="bold">EMAIL         VARCHAR2(100)</text>
  <text x="420" y="346" font-size="11" fill="#999">(ไม่มี Column นี้)</text>
  <text x="54" y="366" font-size="11" fill="#333">HIRE_DATE     DATE</text>
  <text x="420" y="366" font-size="11" fill="#333">HIRE_DATE     DATE</text>
  <text x="54" y="386" font-size="11" fill="#b8860b" font-weight="bold">SALARY        NUMBER(12,2)</text>
  <text x="420" y="386" font-size="11" fill="#b8860b" font-weight="bold">SALARY        NUMBER(10,2)</text>
  <text x="54" y="412" font-size="10" fill="#888">สีแดง = Column มีฝั่ง Source แต่ไม่มีฝั่ง Destination</text>
  <text x="420" y="412" font-size="10" fill="#888">สีเหลือง/น้ำตาล = มีทั้งคู่แต่ Data Type ต่างกัน</text>
</svg>

**ความหมายของสี/สถานะในตาราง:**

| สี/สถานะ | ความหมาย | สิ่งที่ควรทำต่อ |
|---|---|---|
| 🔴 **Only in Source** | Object มีอยู่ใน Dev แต่**ไม่มีใน Production** | ต้อง Deploy/Create Object นี้ไปยัง Production (ถ้าเป็นของจริงที่ต้องใช้งาน) |
| 🔵 **Only in Destination** | Object มีอยู่ใน Production แต่**ไม่มีใน Dev** | ตรวจสอบก่อนว่าเป็น Object ที่ถูกสร้างเฉพาะ Production (เช่น Object ของทีม Infra) หรือเป็น Object เก่าที่ Dev ลบไปแล้วแต่ลืม Sync — **ห้าม Drop ทันทีโดยไม่ตรวจสอบ** |
| 🟡 **Different** | Object มีทั้งสองฝั่ง แต่ Definition ต่างกัน (Column, Data Type, Source Code ฯลฯ) | คลิกที่แถวนั้นเพื่อดู Detail เปรียบเทียบแบบ Side-by-side ในบานด้านล่าง |
| ⚪ **Equal** | Object เหมือนกันทั้งสองฝั่ง | ไม่ต้องทำอะไร (จะแสดงเฉพาะเมื่อติ๊ก **Show Equal Objects**) |

> **Show Equal Objects:** ปกติ Diff Report จะ**ซ่อน**รายการที่เหมือนกันทุกประการเพื่อให้ดูง่าย ถ้าต้องการเห็นรายการ Object ทั้งหมดแบบครบ (รวมที่เหมือนกัน) ให้ติ๊กช่องนี้

---

## 9. การดูรายละเอียดความแตกต่างระดับ Column/Source Code

เมื่อคลิกเลือกแถวที่มีสถานะ **Different** บานด้านล่างของหน้าต่าง Diff Report จะแสดงรายละเอียดแยกตามประเภท Object:

- **Table:** แสดงรายชื่อ Column ทั้งสองฝั่งแบบเทียบคู่กัน (Column Name, Data Type, Length, Nullable, Default) — Column ที่ขาดหรือ Data Type ต่างกันจะถูก Highlight สี
- **View:** แสดง Query Text ของ View ทั้งสองฝั่งแบบ Side-by-side เพื่อดูว่า Logic ต่างกันที่บรรทัดใด
- **Procedure/Function/Package:** แสดง Source Code (PL/SQL) ทั้งสองฝั่งแบบเทียบบรรทัดต่อบรรทัด คล้ายการทำ Diff โค้ดใน Git

ใช้ข้อมูลในส่วนนี้เพื่อตัดสินใจว่า "ความต่างที่เจอ" เป็นสิ่งที่ตั้งใจ (เช่น Production มี Tuning เพิ่มเติมเรื่อง Storage) หรือเป็นสิ่งที่ลืม Deploy จริงๆ

---

## 10. การ Generate Sync Script (ALTER/CREATE)

จากหน้า Diff Report คลิกไอคอน **SQL** สีเขียวที่มุมซ้ายบน (หรือคลิกขวาที่ Object ที่ต้องการ แล้วเลือก **Generate Diff Script**) ระบบจะ Generate DDL Script เป็น Script เดียวรวมทุก Object ที่เลือก

ตัวอย่างลักษณะ Script ที่จะได้ (เป็นเพียงตัวอย่างโครงสร้าง ไม่ใช่ Output จริงจากระบบ):

```sql
-- Generated by Database Diff: APP_DEV -> APP_PROD
-- Object: DEPARTMENT (TABLE) - Only in Source
CREATE TABLE APP_PROD.DEPARTMENT (
    DEPT_ID    NUMBER(10) NOT NULL,
    DEPT_NAME  VARCHAR2(100),
    CONSTRAINT DEPARTMENT_PK PRIMARY KEY (DEPT_ID)
);

-- Object: EMPLOYEE (TABLE) - Different
ALTER TABLE APP_PROD.EMPLOYEE ADD (EMAIL VARCHAR2(100));
ALTER TABLE APP_PROD.EMPLOYEE MODIFY (SALARY NUMBER(12,2));

-- Object: CALC_SALARY (FUNCTION) - Different
CREATE OR REPLACE FUNCTION APP_PROD.CALC_SALARY (...)
IS
BEGIN
    -- updated logic from Dev
    NULL;
END;
/
```

**สิ่งที่ต้องทำต่อหลัง Generate Script:**

1. คลิก **Save** เพื่อบันทึก Script เก็บไว้เป็นหลักฐาน (Audit Trail) ก่อนเสมอ
2. **ห้ามกด Run/Execute ทันทีบน Production** ให้เปิด Script นี้ใน SQL Worksheet แยก แล้ว Review ทุกบรรทัดก่อน
3. นำ Script ไปทดสอบรันบน **Test/UAT Environment** ก่อน เพื่อตรวจว่าไม่มี Error และไม่กระทบข้อมูลที่มีอยู่ (เช่น คำสั่ง `ALTER TABLE ... MODIFY` ที่ลด Length ของ Column อาจทำให้ข้อมูลเดิม Error ได้)
4. ตรวจสอบลำดับการรัน Statement (เช่น ต้องสร้าง Table ก่อนสร้าง Foreign Key ที่อ้างถึง Table นั้น)
5. เมื่อมั่นใจแล้ว จึงนำไปขอ Approve ตามขั้นตอน Change Management ของหน่วยงาน ก่อนรันบน Production จริง

---

## 11. ข้อควรระวังเมื่อใช้งานกับ Production

> ⚠️ ส่วนนี้สำคัญมากสำหรับการทำงานในหน่วยงานราชการ/องค์กรที่มีระบบ Production จริง

- **ใช้ User สิทธิ์อ่านอย่างเดียว (Read-only)** สำหรับ Connection ฝั่ง Production เสมอในขั้นตอน Diff — ไม่จำเป็นต้องให้สิทธิ์เขียนเพื่อแค่ "ดูความต่าง"
- **Database Diff ไม่ได้ Sync ข้อมูลอัตโนมัติ** มันสร้างได้แค่ Script DDL ให้ — การจะรันจริงหรือไม่ ผู้ใช้ต้องตัดสินใจและรับผิดชอบเอง
- คำสั่งบางประเภทที่ Generate มาอาจ**มีผลกระทบต่อข้อมูล** เช่น `DROP COLUMN`, `MODIFY` ที่ลด Size — ควร Backup หรือมี Point-in-time Recovery พร้อมก่อนรันจริงบน Production
- หลีกเลี่ยงการรัน Compare ที่ดึง Object จำนวนมากในช่วงเวลาที่ Production มีโหลดสูง เพราะการ Query Data Dictionary จำนวนมากอาจส่งผลต่อ Performance เล็กน้อย
- เก็บ Diff Report และ Sync Script ทุกครั้งไว้เป็นเอกสารประกอบการขอ Change (Change Request) เพื่อ Audit ย้อนหลังได้

---

## 12. ปัญหาที่พบบ่อยและวิธีแก้ไข

| ปัญหา | สาเหตุที่เป็นไปได้ | วิธีแก้ไข |
|---|---|---|
| เมนู **Database Diff** เป็นสีจาง กดไม่ได้ | มี Connection ใน Navigator น้อยกว่า 2 รายการ | สร้าง Connection เพิ่มให้ครบทั้ง Dev และ Prod |
| Compare แล้วค้างนาน/ไม่จบ | Schema มี Object จำนวนมาก หรือ Network ไปยัง Production ช้า/ไม่เสถียร | ลด Object Type ที่เลือก หรือใช้ Step 3 Filter เฉพาะ Object ที่สนใจ |
| ไม่เห็น Object บางตัว เช่น Object ของ Schema อื่น | User ที่ใช้ Connect ไม่มีสิทธิ์เห็น Object ของ Schema นั้น | ขอสิทธิ์ `SELECT ANY DICTIONARY` หรือ Role ที่เหมาะสมจาก DBA |
| Generate Script แล้ว Error ตอนรันจริง | ลำดับ Object ไม่ถูกต้อง (เช่น FK อ้างถึง Table ที่ยังไม่ถูกสร้าง) หรือ Object มี Dependency อื่น | จัดลำดับ Statement เองตาม Dependency หรือรันแยกเป็นกลุ่มทีละ Object Type |
| ผลลัพธ์ "Different" แต่ดูแล้ว Column เหมือนกันทุกอย่าง | อาจต่างกันที่ Storage/Tablespace Clause ซึ่งบางครั้งไม่เกี่ยวกับ Logic | ตรวจดู Detail ให้ละเอียด หากเป็นแค่ Storage Clause สามารถพิจารณาไม่ต้อง Sync ก็ได้ |

---

## 13. สรุปขั้นตอนแบบย่อ (Quick Reference)

1. เปิด **Tools > Database Diff...**
2. **Step 1:** เลือก Source Connection/Schema (Dev) และ Destination Connection/Schema (Prod)
3. **Step 2:** ติ๊กเลือก Object Type ที่ต้องการเทียบ (Tables, Views, Indexes, Functions, ...)
4. **Step 3:** (ถ้าต้องการ) Filter เฉพาะบาง Object ด้วย Lookup + ย้ายเข้า Selected Objects — ถ้าไม่เลือกจะถือว่าเทียบทุกตัว
5. **Step 4:** ตรวจทาน Summary แล้วกด **Finish**
6. อ่านผลลัพธ์ใน **Diff Report** — ดูสถานะ Only in Source / Only in Destination / Different / Equal
7. คลิกแถวที่ Different เพื่อดู Detail ระดับ Column/Source Code
8. คลิกไอคอน **SQL** เพื่อ **Generate Diff Script**
9. **Save Script → Review → ทดสอบบน Test/UAT → ขอ Approve → จึงนำไปรันบน Production**

---

*คู่มือนี้จัดทำขึ้นเพื่อใช้งานภายในหน่วยงาน อ้างอิงพฤติกรรมการทำงานของ Oracle SQL Developer ตระกูล 21.x — ชื่อเมนู/ตำแหน่งปุ่มอาจขยับเล็กน้อยตาม Patch แต่ลำดับขั้นตอนหลักยังคงเดิม*
