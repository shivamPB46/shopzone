# 🛒 ShopZone — Full Stack E-Commerce App (SQLite Edition)

A production-ready E-Commerce app built with **React + Node.js + Express + SQLite (via Sequelize)** and **Razorpay** payments.

> ✅ **No database server needed** — SQLite stores everything in a single `.db` file automatically created on first run.

---

## 🗂️ Folder Structure

```
ecommerce-app/
├── backend/
│   ├── config/
│   │   ├── db.js               ← SQLite + Sequelize connection
│   │   └── cloudinary.js       ← Image upload config
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── productController.js
│   │   ├── orderController.js
│   │   └── userController.js
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   ├── adminMiddleware.js
│   │   └── errorMiddleware.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Product.js
│   │   └── Order.js            ← Order + OrderItem
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── productRoutes.js
│   │   ├── orderRoutes.js
│   │   └── userRoutes.js
│   ├── database/               ← SQLite .db file lives here (auto-created)
│   ├── uploads/                ← Temp image storage
│   ├── .env.example
│   ├── package.json
│   └── server.js
│
├── frontend/
│   ├── src/
│   │   ├── api/axios.js
│   │   ├── components/
│   │   ├── context/
│   │   ├── pages/
│   │   └── App.jsx
│   ├── .env.example
│   └── package.json
└── README.md
```

---

## 🚀 Quick Setup (3 steps — no DB install needed!)

### Step 1 — Install dependencies

```bash
# Backend
cd backend
npm install

# Frontend
cd ../frontend
npm install
```

---

### Step 2 — Configure environment variables

```bash
# Backend
cd backend
cp .env.example .env
```

Edit `backend/.env`:

```env
PORT=5000
NODE_ENV=development

# SQLite file location (auto-created, no server needed)
DB_STORAGE=./database/shopzone.db

JWT_SECRET=your_long_secret_key_here
JWT_EXPIRE=30d

# Cloudinary (free account at cloudinary.com)
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Razorpay (free test account at razorpay.com)
RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=your_razorpay_secret

CLIENT_URL=http://localhost:5173
```

```bash
# Frontend
cd ../frontend
cp .env.example .env
```

Edit `frontend/.env`:
```env
VITE_API_URL=http://localhost:5000/api
VITE_RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
```

---

### Step 3 — Run the app

**Terminal 1 — Backend:**
```bash
cd backend
npm run dev
```
You'll see:
```
✅ SQLite connected — file: ./database/shopzone.db
✅ Database tables synced.
🚀 Server running on http://localhost:5000 [SQLite mode]
```
> The `shopzone.db` file and all tables are created automatically!

**Terminal 2 — Frontend:**
```bash
cd frontend
npm run dev
```
> App opens at **http://localhost:5173**

---

## 🛡️ Making Yourself Admin

Since there's no psql needed, use any SQLite GUI or this one-liner:

```bash
# Install sqlite3 CLI if needed
npm install -g better-sqlite3-cli

# Or use DB Browser for SQLite (free GUI): https://sqlitebrowser.org/
```

Or add this temporary route to server.js during dev:
```js
// TEMP: run once then remove
app.get('/make-admin/:email', async (req, res) => {
  const User = require('./models/User');
  await User.update({ role: 'admin' }, { where: { email: req.params.email } });
  res.json({ message: 'Done' });
});
// Visit: http://localhost:5000/make-admin/your@email.com
```

---

## 💳 Razorpay Payment Flow

1. User fills shipping form → clicks Pay
2. Backend creates Razorpay order → returns `razorpay_order_id`
3. Razorpay popup opens → user pays
4. Backend **verifies HMAC signature** → saves order to SQLite → deducts stock
5. User sees Order Success page ✅

**Test Card:**
- Number: `4111 1111 1111 1111`
- Expiry: Any future date | CVV: Any 3 digits | OTP: `1234`

---

## 🔌 API Endpoints

| Method | Route | Access |
|--------|-------|--------|
| POST | /api/auth/register | Public |
| POST | /api/auth/login | Public |
| GET | /api/products | Public |
| GET | /api/products/:id | Public |
| POST | /api/products | Admin |
| PUT | /api/products/:id | Admin |
| DELETE | /api/products/:id | Admin |
| POST | /api/orders/create-razorpay-order | Protected |
| POST | /api/orders | Protected |
| GET | /api/orders/my | Protected |
| GET | /api/orders | Admin |
| PUT | /api/orders/:id/status | Admin |

---

## 🌐 Deployment

### Backend → Render.com
- SQLite works on Render but note: **free tier resets the filesystem** on deploy
- For persistence on Render, upgrade to paid or switch to Railway + SQLite volume

### Frontend → Vercel
```bash
cd frontend
npm run build
# Deploy dist/ to Vercel
```

### For production with persistent data, consider:
- **Turso** — SQLite-compatible cloud DB (drop-in replacement)
- **PlanetScale / Supabase** — switch dialect to mysql/postgres in db.js

---

## 🏗️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite + Tailwind CSS |
| Backend | Node.js + Express.js |
| Database | **SQLite** via Sequelize ORM |
| Auth | JWT + bcryptjs |
| Payments | Razorpay |
| Images | Cloudinary + Multer |
| State | React Context API |
| Routing | React Router v6 |

---

## ⚠️ SQLite vs PostgreSQL — Key Differences in This App

| Feature | PostgreSQL | SQLite (this app) |
|---------|-----------|-------------------|
| JSON fields | `JSONB` native | `TEXT` + getter/setter |
| ENUM type | Native `ENUM` | `STRING` + `isIn` validator |
| Case-insensitive search | `Op.iLike` | `Op.like` |
| Setup required | Install PostgreSQL | ❌ None — file based |
| Good for | Production scale | Dev, small apps, demos |
