# KbizAPI

> Unofficial Node.js wrapper สำหรับ **K-Biz Internet Banking** (KBank Business)  
> ดึงยอดเงิน / ประวัติธุรกรรม โดยไม่ต้องเข้าเว็บ

---

## ⚠️ คำเตือน

- เป็น **Unofficial API** — reverse-engineered จาก KBIZ web
- ใช้กับบัญชี **K-Biz เท่านั้น** (ไม่ใช่ K-Plus ส่วนตัว)
- ผู้พัฒนาไม่รับผิดชอบความเสียหายที่เกิดขึ้นจากการใช้งาน

---

## 📋 ความต้องการ

- Node.js `v14+`
- บัญชี K-Biz (นิติบุคคล / ร้านค้า)

---

## 🚀 ติดตั้ง

```bash
git clone https://github.com/slumzick-dev/KbizAPI.git
cd KbizAPI
npm install
```

---

## ⚙️ ตั้งค่า

แก้ไขไฟล์ `index.js` ใส่ข้อมูลจริง:

```js
const config = {
  username: "ชื่อผู้ใช้ KBIZ",
  password: "รหัสผ่าน KBIZ",
  accountnumber: "เลขบัญชี KBank",
  ibId: "",   // รับอัตโนมัติจาก Login()
  token: "",  // รับอัตโนมัติจาก Login()
};
```

---

## 📖 การใช้งาน

### Basic

```js
const KbizAPI = require("./index");

(async () => {
  const kbiz = new KbizAPI(config);

  // 1. Login → รับ token + ibId
  const session = await kbiz.Login();
  console.log(session);

  // 2. เช็ค session ยังใช้ได้ไหม
  const alive = await kbiz.CheckSession(); // true / false

  // 3. ดูข้อมูลบัญชี + ยอดเงิน
  const info = await kbiz.GetInfoUser();

  // 4. ดึงประวัติธุรกรรม
  const txn = await kbiz.getTransactionList();
  console.log(txn);
})();
```

### Auto Poll ยอด (สำหรับ Payment Gateway)

```js
const KbizAPI = require("./index");

const config = {
  username: "YOUR_USERNAME",
  password: "YOUR_PASSWORD",
  accountnumber: "YOUR_ACCOUNT",
  ibId: "",
  token: "",
};

async function initSession(kbiz) {
  const session = await kbiz.Login();
  config.token = session.token;
  config.ibId = session.ibId;
  console.log("[KBIZ] Login OK");
}

async function startPolling(kbiz, intervalMs = 10000) {
  setInterval(async () => {
    try {
      const alive = await kbiz.CheckSession();
      if (!alive) {
        console.log("[KBIZ] Session หมด → Re-login...");
        await initSession(kbiz);
      }
      const txn = await kbiz.getTransactionList();
      console.log("[KBIZ] Latest TX:", txn[0]);
      // ใส่ logic match order ของระบบตรงนี้
    } catch (e) {
      console.error("[KBIZ] Error:", e.message);
    }
  }, intervalMs);
}

(async () => {
  const kbiz = new KbizAPI(config);
  await initSession(kbiz);
  await startPolling(kbiz, 10000); // poll ทุก 10 วิ
})();
```

---

## 🔧 Methods

| Method | คืนค่า | คำอธิบาย |
|---|---|---|
| `Login()` | `{ token, ibId }` | Login และรับ session |
| `CheckSession()` | `boolean` | เช็คว่า token ยังใช้ได้ |
| `GetInfoUser()` | `array` | ข้อมูลบัญชี + ยอดเงิน |
| `getTransactionList()` | `array` | ประวัติธุรกรรมล่าสุด |

---

## 📱 รันบน Termux

```bash
pkg update && pkg upgrade -y
pkg install git nodejs -y
git clone https://github.com/slumzick-dev/KbizAPI.git
cd KbizAPI
npm install
node index.js
```

---

## 📄 License

MIT — ดัดแปลงและใช้ได้ฟรี

---

> Original by [BossNz](https://github.com/BossNz) • Fork & maintained by [slumzick-dev](https://github.com/slumzick-dev)
