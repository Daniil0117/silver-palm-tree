import express from "express";
import helmet from "helmet";
import dotenv from "dotenv";
import Database from "better-sqlite3";
import crypto from "node:crypto";
import path from "node:path";
import fs from "node:fs";
import { fileURLToPath } from "node:url";

dotenv.config();
const app = express();
const PORT = Number(process.env.PORT || 3000);
const __dirname = path.dirname(fileURLToPath(import.meta.url));
const dataDir = process.env.DATABASE_DIR || path.join(__dirname, "data");
fs.mkdirSync(dataDir, { recursive: true });
const db = new Database(path.join(dataDir, "strikgold.sqlite"));
db.pragma("journal_mode = WAL");
db.pragma("busy_timeout = 5000");
db.exec(`
CREATE TABLE IF NOT EXISTS orders (
  id TEXT PRIMARY KEY, platform TEXT NOT NULL, pack TEXT NOT NULL,
  amount INTEGER NOT NULL, currency TEXT NOT NULL, delivery TEXT NOT NULL,
  contact TEXT NOT NULL, note TEXT DEFAULT '', status TEXT NOT NULL DEFAULT 'awaiting_payment',
  payment_session TEXT, created_at TEXT NOT NULL
);`);
const CURRENCY = (process.env.CURRENCY || "RUB").toUpperCase();
const packs = {
  starter: { name: "Стартовый", amount: Number(process.env.PRICE_STARTER || 49900) },
  advanced: { name: "Продвинутый", amount: Number(process.env.PRICE_ADVANCED || 99900) },
  ultimate: { name: "Максимальный", amount: Number(process.env.PRICE_ULTIMATE || 199900) }
};
const validPlatforms = new Set(["PC", "Xbox", "PlayStation"]);
const validDeliveries = new Set(["Через трансферный рынок", "Комфортный способ"]);
const sessions = new Map();
const paymentsConfigured = Boolean(process.env.YOOKASSA_SHOP_ID && process.env.YOOKASSA_SECRET_KEY && process.env.BASE_URL);

function clean(s, max=500) { return String(s ?? "").trim().slice(0, max); }
function rubles(kopecks) { return (kopecks / 100).toFixed(2); }
function paymentAuth() {
  return "Basic " + Buffer.from(`${process.env.YOOKASSA_SHOP_ID}:${process.env.YOOKASSA_SECRET_KEY}`).toString("base64");
}
function auth(req, res, next) {
  if (!process.env.ADMIN_PASSWORD || !process.env.SESSION_SECRET) return res.status(503).json({ error: "Админка не настроена. Заполните ADMIN_PASSWORD и SESSION_SECRET." });
  const token = req.get("x-admin-token");
  const expected = crypto.createHmac("sha256", process.env.SESSION_SECRET).update(process.env.ADMIN_PASSWORD).digest("hex");
  const got = Buffer.from(String(token || "").padEnd(64, " ").slice(0,64));
  if (!token || !crypto.timingSafeEqual(got, Buffer.from(expected))) return res.status(401).json({ error: "Не авторизован" });
  next();
}
async function notifyTelegram(text) {
  const token = process.env.TELEGRAM_BOT_TOKEN, chatId = process.env.TELEGRAM_CHAT_ID;
  if (!token || !chatId) return;
  try {
    const response = await fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
      method: "POST", headers: { "content-type": "application/json" },
      body: JSON.stringify({ chat_id: chatId, text, disable_web_page_preview: true })
    });
    if (!response.ok) console.error("Telegram notification failed with HTTP", response.status);
  } catch (e) { console.error("Telegram notification failed:", e.message); }
}

app.use(helmet({ contentSecurityPolicy: false }));
app.use(express.json({ limit: "30kb" }));
app.use(express.static(path.join(__dirname, "public")));

app.get("/api/health", (_req,res) => res.json({ ok: true }));
app.get("/api/config", (_req,res) => {
  res.json({ currency: CURRENCY, packs: Object.fromEntries(Object.entries(packs).map(([k,v])=>[k,{name:v.name,amount:v.amount}])), paymentsEnabled: paymentsConfigured, paymentProvider: paymentsConfigured ? "yookassa" : null });
});

app.post("/api/orders", async (req,res) => {
  const platform = clean(req.body.platform, 30), packKey = clean(req.body.pack, 30);
  const delivery = clean(req.body.delivery, 80), contact = clean(req.body.contact, 180), note = clean(req.body.note, 1000);
  if (!validPlatforms.has(platform)) return res.status(400).json({ error: "Выберите поддерживаемую платформу." });
  if (!packs[packKey]) return res.status(400).json({ error: "Неизвестный пакет." });
  if (!validDeliveries.has(delivery)) return res.status(400).json({ error: "Выберите способ доставки." });
  if (contact.length < 3) return res.status(400).json({ error: "Укажите контакт для связи." });
  if (!Number.isSafeInteger(packs[packKey].amount) || packs[packKey].amount <= 0) return res.status(500).json({ error: "Неверная цена пакета. Проверьте переменные PRICE_* на сервере." });
  if (CURRENCY !== "RUB") return res.status(500).json({ error: "Этот проект настроен для RUB. Установите CURRENCY=RUB." });
  const pack = packs[packKey], id = "SG-" + crypto.randomBytes(4).toString("hex").toUpperCase();
  db.prepare(`INSERT INTO orders (id,platform,pack,amount,currency,delivery,contact,note,status,created_at) VALUES (?,?,?,?,?,?,?,?,?,?)`)
    .run(id,platform,pack.name,pack.amount,CURRENCY,delivery,contact,note,paymentsConfigured ? "awaiting_payment" : "payment_not_configured",new Date().toISOString());

  if (!paymentsConfigured) {
    await notifyTelegram(`🆕 Новая заявка STRIKGOLD (онлайн-оплата не настроена)\nЗаказ: ${id}\nПлатформа: ${platform}\nПакет: ${pack.name}\nСумма: ${rubles(pack.amount)} RUB\nКонтакт: ${contact}\nДоставка: ${delivery}`);
    return res.status(201).json({ orderId:id, paymentConfigured:false, message:"Заявка сохранена. Онлайн-оплата ещё не настроена." });
  }

  try {
    const response = await fetch("https://api.yookassa.ru/v3/payments", {
      method: "POST",
      headers: { "Authorization": paymentAuth(), "Idempotence-Key": crypto.randomUUID(), "Content-Type": "application/json" },
      body: JSON.stringify({
        amount: { value: rubles(pack.amount), currency: "RUB" },
        capture: true,
        confirmation: { type: "redirect", return_url: `${process.env.BASE_URL.replace(/\/$/, "")}/?payment=return&order=${encodeURIComponent(id)}` },
        description: `STRIKGOLD — ${pack.name} (${platform}), заказ ${id}`,
        metadata: { orderId: id, platform, pack: pack.name }
      })
    });
    const payment = await response.json();
    if (!response.ok || !payment.id || !payment.confirmation?.confirmation_url) {
      console.error("YooKassa create payment failed:", response.status, JSON.stringify(payment).slice(0,1000));
      db.prepare("UPDATE orders SET status='payment_error' WHERE id=?").run(id);
      return res.status(502).json({ error:"ЮKassa не создала платёж. Проверьте Shop ID, секретный ключ и настройки магазина.", orderId:id });
    }
    db.prepare("UPDATE orders SET payment_session=? WHERE id=?").run(payment.id,id);
    await notifyTelegram(`🧾 Создан заказ STRIKGOLD\nЗаказ: ${id}\nПлатформа: ${platform}\nПакет: ${pack.name}\nСумма: ${rubles(pack.amount)} RUB\nСтатус: ожидает оплату`);
    return res.status(201).json({ orderId:id, paymentConfigured:true, checkoutUrl:payment.confirmation.confirmation_url, message:"Заказ создан. Сейчас откроется защищённая страница оплаты ЮKassa." });
  } catch (err) {
    console.error("YooKassa request failed:", err.message);
    db.prepare("UPDATE orders SET status='payment_error' WHERE id=?").run(id);
    return res.status(502).json({ error:"Не удалось связаться с ЮKassa. Проверьте настройки сервера.", orderId:id });
  }
});

// ЮKassa webhook: доверяем не входящему JSON, а повторно запрашиваем платёж по API.
app.post("/api/yookassa-webhook", async (req,res) => {
  if (!paymentsConfigured) return res.status(503).send("Payments not configured");
  const notification = req.body;
  if (notification?.event !== "payment.succeeded" || notification?.object?.id == null) return res.status(200).json({ received:true, ignored:true });
  try {
    const response = await fetch(`https://api.yookassa.ru/v3/payments/${encodeURIComponent(notification.object.id)}`, {
      headers: { "Authorization": paymentAuth(), "Accept": "application/json" }
    });
    if (!response.ok) {
      console.error("YooKassa payment verification failed with HTTP", response.status);
      return res.status(502).send("Could not verify payment");
    }
    const payment = await response.json();
    if (payment.status !== "succeeded" || payment.paid !== true) return res.status(200).json({ received:true, ignored:true });
    const orderId = clean(payment.metadata?.orderId, 50);
    const row = db.prepare("SELECT * FROM orders WHERE id=? AND payment_session=?").get(orderId, payment.id);
    if (!row) return res.status(200).json({ received:true, ignored:true });
    if (payment.amount?.currency !== row.currency || Number(payment.amount?.value) !== Number(rubles(row.amount))) {
      console.error("Payment amount/currency mismatch for order", row.id);
      return res.status(400).send("Payment mismatch");
    }
    const changed = db.prepare("UPDATE orders SET status='paid' WHERE id=? AND status!='paid'").run(row.id).changes;
    if (changed) await notifyTelegram(`✅ Оплата подтверждена ЮKassa\nЗаказ: ${row.id}\nПлатформа: ${row.platform}\nПакет: ${row.pack}\nСумма: ${rubles(row.amount)} ${row.currency}\nКонтакт: ${row.contact}`);
    return res.status(200).json({ received:true });
  } catch (err) {
    console.error("YooKassa webhook verification error:", err.message);
    return res.status(500).send("Webhook processing failed");
  }
});

app.post("/api/admin/login", (req,res) => {
  const password = clean(req.body.password, 300);
  if (!process.env.ADMIN_PASSWORD || !process.env.SESSION_SECRET) return res.status(503).json({error:"Настройте секреты в переменных окружения"});
  if (password !== process.env.ADMIN_PASSWORD) return res.status(401).json({error:"Неверный пароль"});
  const token = crypto.createHmac("sha256",process.env.SESSION_SECRET).update(password).digest("hex");
  sessions.set(token, Date.now()+8*60*60*1000);
  res.json({token});
});
function adminSession(req,res,next) {
  const token = req.get("x-admin-token"), expiry = token && sessions.get(token);
  if (!expiry || expiry < Date.now()) return res.status(401).json({error:"Сессия истекла. Войдите снова."});
  next();
}
app.get("/api/admin/orders", adminSession, (_req,res) => {
  const rows = db.prepare("SELECT id,platform,pack,amount,currency,delivery,contact,note,status,created_at FROM orders ORDER BY created_at DESC LIMIT 500").all();
  res.json({orders:rows});
});
app.patch("/api/admin/orders/:id", adminSession, (req,res) => {
  const allowed = new Set(["awaiting_payment","paid","processing","completed","cancelled","payment_not_configured","payment_error"]);
  const status = clean(req.body.status,40);
  if (!allowed.has(status)) return res.status(400).json({error:"Недопустимый статус"});
  const result = db.prepare("UPDATE orders SET status=? WHERE id=?").run(status,clean(req.params.id,50));
  if (!result.changes) return res.status(404).json({error:"Заказ не найден"});
  res.json({ok:true});
});
app.get("/api/admin/export", adminSession, (_req,res) => {
  const rows = db.prepare("SELECT * FROM orders ORDER BY created_at DESC").all();
  res.setHeader("content-type","application/json; charset=utf-8");
  res.setHeader("content-disposition",'attachment; filename="strikgold-orders.json"');
  res.send(JSON.stringify(rows,null,2));
});
app.get("*", (_req,res) => res.sendFile(path.join(__dirname,"public","index.html")));
app.listen(PORT, "0.0.0.0", () => console.log(`STRIKGOLD listening on port ${PORT}; database directory: ${dataDir}`));
