# MERN Store – E-Commerce Management Platform

A full-stack e-commerce application built with **MongoDB, Express.js, React.js and Node.js**.
Customers browse, search, add to cart or wishlist, pay with Razorpay (test mode) or Cash on Delivery,
and track their orders. Admins manage products, categories, inventory and orders, and see sales analytics.

**Live demo:** _add your deployed URL here_  ·  **API:** _add your backend URL here_

## Features

| # | Requirement | Where it is implemented |
|---|---|---|
| 1 | JWT authentication and role-based authorization | `backend/middleware/auth.js` (`protect`, `authorize`), `controllers/authController.js` |
| 2 | Admin dashboard with sales analytics | `GET /api/admin/dashboard`, `frontend/src/pages/admin/Dashboard.jsx` |
| 3 | Product management (CRUD) | `controllers/productController.js`, `pages/admin/Products.jsx` |
| 4 | Categories and inventory management | `controllers/categoryController.js`, inline stock editor + low-stock filter |
| 5 | Product search, sorting and filtering | `GET /api/products` (search, category, price range, in-stock, sort, pagination) |
| 6 | Shopping cart and wishlist | `controllers/cartController.js`, `controllers/wishlistController.js` |
| 7 | Order placement and order tracking | `controllers/orderController.js`, `components/OrderTimeline.jsx` |
| 8 | Razorpay payment (test mode) | `config/razorpay.js`, `utils/razorpay.js` (frontend), signature verification |
| 9 | Order management and invoice generation | Admin status flow, PDF invoice with PDFKit (`utils/generateInvoice.js`) |
| 10 | Image upload and cloud storage | Multer (memory) + Cloudinary (`middleware/upload.js`, `config/cloudinary.js`) |
| 11 | Email notifications | Nodemailer: welcome, order placed, every status change (`utils/sendEmail.js`) |
| 12 | Advanced dashboard with sales reports | `GET /api/admin/reports/sales`, date range, day/month, top products, by category, CSV export |
| 13 | Secure REST APIs and backend validation | express-validator, helmet, CORS allow-list, rate limiting, central error handler |
| 14 | Responsive frontend | Tailwind CSS, mobile menu, mobile-friendly tables |
| 15 | Deployment | MongoDB Atlas + Render (API) + Vercel (frontend), see [Deployment](#deployment) |

### Advanced technical expectations

| Expectation | How |
|---|---|
| MVC architecture | `models/` (Mongoose), `controllers/` (logic), `routes/` (URL → controller); the React app is the view |
| Reusable React components | `ProductCard`, `Modal`, `Field`, `Pagination`, `StatusBadge`, `OrderTimeline`, `ProductFormModal`, `useShopActions` hook |
| Redux Toolkit | Slices for `auth`, `products`, `cart`, `wishlist`, `orders` with `createAsyncThunk` |
| Database relationships and aggregation | References between all collections; `$group`, `$unwind`, `$lookup`, `$dateToString` in reports |
| API security and error handling | See [Security](#security) |
| Environment variables | `.env.example` in both apps; secrets never committed |
| Git version control | `.gitignore` excludes `node_modules`, `.env`, `dist` |

## Tech stack

- **Frontend:** React 18, Redux Toolkit, React Router 6, Axios, Tailwind CSS 4, Recharts, Vite
- **Backend:** Node.js, Express 4, Mongoose 8, JWT, bcryptjs, express-validator, Multer, Cloudinary, Razorpay, Nodemailer, PDFKit
- **Security:** helmet, CORS allow-list, express-rate-limit, request size limit

## Project structure

```
shop/
├── backend/
│   ├── config/          db, cloudinary, razorpay
│   ├── models/          User, Category, Product, Cart, Order
│   ├── controllers/     auth, category, product, cart, wishlist, order, admin
│   ├── routes/          one file per resource
│   ├── middleware/      auth (JWT + roles), upload (multer), errorHandler
│   ├── validators/      express-validator rules
│   ├── utils/           pricing, emails, invoice PDF, constants, helpers
│   ├── app.js           Express app
│   ├── server.js        Entry point
│   └── seed.js          Sample data
└── frontend/
    └── src/
        ├── app/store.js           Redux store
        ├── api/axios.js           Axios instance + JWT interceptor
        ├── features/              auth, products, cart, wishlist, orders slices
        ├── components/            Layout, AdminLayout, ProductCard, Modal, ...
        ├── pages/                 Home, Products, ProductDetail, Cart, Checkout, Orders, ...
        ├── pages/admin/           Dashboard, Products, Categories, Orders, Reports
        └── utils/                 validation, formatting, Razorpay loader
```

## Getting started (local)

Requirements: Node.js 18+ and MongoDB (local, or a free MongoDB Atlas cluster).

### 1. Backend

```bash
cd backend
npm install
copy .env.example .env        # Windows   (macOS/Linux: cp .env.example .env)
```

Fill in `MONGO_URI` and `JWT_SECRET` in `backend/.env` (the other keys are optional, see below). Then:

```bash
npm run seed                  # admin, 3 customers, 6 categories, 24 products, 45 sample orders
npm run dev                   # API on http://localhost:5000
```

The terminal shows whether Razorpay, Cloudinary and email are configured.

### 2. Frontend

```bash
cd frontend
npm install
npm run dev                   # App on http://localhost:5173
```

### Demo logins (after `npm run seed`)

| Role | Email | Password |
|---|---|---|
| Admin | admin@store.com | Admin@123 |
| Customer | rahul@example.com | Customer@123 |

## Optional services

The app runs without these; each feature turns on when its keys are added.

| Service | Without keys | How to get keys |
|---|---|---|
| **Razorpay** (test mode) | Only Cash on Delivery works | Razorpay Dashboard → switch to **Test mode** → Account & Settings → API Keys → Generate Key |
| **Cloudinary** | Products use an image URL instead of upload | Cloudinary Console → Dashboard → API Keys |
| **Email (SMTP)** | Emails are printed in the backend terminal | Gmail: turn on 2-step verification → create an **App password** → `SMTP_HOST=smtp.gmail.com`, `SMTP_PORT=587` |

**Test payment:** in test mode use UPI ID `success@razorpay`, or a Razorpay test card from their documentation. No real money is charged.

## Environment variables

**backend/.env**

| Name | Required | Purpose |
|---|---|---|
| `PORT` | no (5000) | API port |
| `NODE_ENV` | no | `production` hides internal error messages |
| `MONGO_URI` | **yes** | MongoDB connection string |
| `JWT_SECRET` | **yes** | Signs login tokens |
| `JWT_EXPIRES_IN` | no (7d) | Token lifetime |
| `CLIENT_URL` | **yes in production** | Frontend URL allowed by CORS |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` | no | Online payments |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | no | Image upload |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `EMAIL_FROM` | no | Emails |

**frontend** (production only): `VITE_API_URL=https://<your-backend>/api`

## Database schema

```
User 1 ─── 1 Cart          (cart.user)
User 1 ─── * Order         (order.user)
User * ─── * Product       (user.wishlist: [Product])
Category 1 ─── * Product   (product.category)
Order * ─── * Product      (order.items[].product)
```

### users
| Field | Type | Notes |
|---|---|---|
| name | String | required |
| email | String | required, unique, lowercase |
| password | String | bcrypt hash, `select: false` |
| phone | String | |
| role | String | `customer` (default) or `admin` |
| wishlist | [ObjectId → products] | |

### categories
| Field | Type | Notes |
|---|---|---|
| name | String | required, unique |
| description | String | |

### products
| Field | Type | Notes |
|---|---|---|
| name, description, brand | String | |
| category | ObjectId → categories | required, indexed |
| price | Number | selling price |
| mrp | Number | original price, for the "% off" label |
| stock | Number | inventory, min 0 |
| sold | Number | units sold |
| images | [{ url, publicId }] | `publicId` is the Cloudinary id, used to delete the file |
| isActive | Boolean | hidden products are not shown in the store |
| isFeatured | Boolean | shown on the home page |

### carts
| Field | Type | Notes |
|---|---|---|
| user | ObjectId → users | unique (one cart per user) |
| items | [{ product → products, quantity }] | prices are read live from products |

### orders
| Field | Type | Notes |
|---|---|---|
| orderNumber | String | unique, e.g. `ORD-MWJAHRX0-RTKF` |
| user | ObjectId → users | indexed |
| items | [{ product, name, image, price, quantity }] | **snapshot** of the product at purchase time |
| shippingAddress | { fullName, phone, line1, city, state, pincode } | |
| itemsPrice, shippingPrice, totalPrice | Number | calculated on the server |
| paymentMethod | String | `razorpay` or `cod` |
| paymentStatus | String | `pending`, `paid`, `failed`, `refund_pending` |
| payment | { razorpayOrderId, razorpayPaymentId } | |
| status | String | `placed` → `confirmed` → `shipped` → `delivered`, or `cancelled` |
| statusHistory | [{ status, note, at }] | used for order tracking |
| paidAt, deliveredAt | Date | |

**Why the order keeps a copy of name and price:** if the admin later changes a product's price or deletes it,
old orders and invoices must still show what the customer actually paid.

## API documentation

Base URL: `http://localhost:5000/api` · Protected routes need `Authorization: Bearer <token>`

### Auth
| Method | Endpoint | Access | Body |
|---|---|---|---|
| POST | `/auth/register` | Public | `name, email, password` |
| POST | `/auth/login` | Public | `email, password` |
| GET | `/auth/me` | Logged in | – |
| PUT | `/auth/profile` | Logged in | `name?, phone?` |
| PUT | `/auth/password` | Logged in | `currentPassword, newPassword` |

### Categories
| Method | Endpoint | Access | Body |
|---|---|---|---|
| GET | `/categories` | Public | – (includes `productCount`) |
| POST | `/categories` | Admin | `name, description?` |
| PUT | `/categories/:id` | Admin | `name, description?` |
| DELETE | `/categories/:id` | Admin | – (blocked while products use it) |

### Products
| Method | Endpoint | Access | Body / Query |
|---|---|---|---|
| GET | `/products` | Public | `search, category, minPrice, maxPrice, inStock, featured, sort, page, limit` |
| GET | `/products/:id` | Public | – |
| GET | `/products/admin/all` | Admin | same as above + `lowStock`, includes hidden products |
| POST | `/products` | Admin | **multipart/form-data**: `name, description, category, price, stock, mrp?, brand?, isActive?, isFeatured?, imageUrl?, images[]` (max 5, 2 MB each) |
| PUT | `/products/:id` | Admin | same as POST, all optional, plus `removeImageIds` (comma separated) |
| PATCH | `/products/:id/stock` | Admin | `stock` |
| DELETE | `/products/:id` | Admin | – |

`sort`: `newest` (default), `popular`, `price_asc`, `price_desc`, `name_asc`

### Cart (customer)
| Method | Endpoint | Body |
|---|---|---|
| GET | `/cart` | – |
| POST | `/cart` | `productId, quantity?` (adds to existing quantity) |
| PUT | `/cart/:productId` | `quantity` (sets exact quantity, 1–10) |
| DELETE | `/cart/:productId` | – |
| DELETE | `/cart` | – (clear) |

Every cart response returns the full cart: `items, totalItems, itemsPrice, shippingPrice, totalPrice`.

### Wishlist (customer)
| Method | Endpoint | Description |
|---|---|---|
| GET | `/wishlist` | List |
| POST | `/wishlist/:productId` | Add, or remove if already there |

### Orders
| Method | Endpoint | Access | Body / Query |
|---|---|---|---|
| POST | `/orders` | Customer | `shippingAddress{fullName, phone, line1, city, state, pincode}, paymentMethod` |
| POST | `/orders/:id/pay` | Customer | – (new Razorpay order for an unpaid order) |
| POST | `/orders/:id/verify-payment` | Customer | `razorpay_order_id, razorpay_payment_id, razorpay_signature` |
| GET | `/orders/my` | Customer | `page, limit` |
| PUT | `/orders/:id/cancel` | Customer | – (only while `placed` or `confirmed`) |
| GET | `/orders/:id` | Owner or admin | – |
| GET | `/orders/:id/invoice` | Owner or admin | – (returns a PDF) |
| GET | `/orders` | Admin | `search, status, paymentStatus, page, limit` |
| PUT | `/orders/:id/status` | Admin | `status, note?` |

### Admin analytics
| Method | Endpoint | Query |
|---|---|---|
| GET | `/admin/dashboard` | – (revenue, orders by status, low stock, recent orders, last 30 days) |
| GET | `/admin/reports/sales` | `from=YYYY-MM-DD, to=YYYY-MM-DD, groupBy=day\|month` |

### Error format

```json
{ "success": false, "message": "Validation failed", "errors": [{ "field": "email", "message": "Enter a valid email" }] }
```

`400` bad request · `401` not logged in / bad token · `403` wrong role · `404` not found · `409` duplicate ·
`422` validation · `429` too many requests · `502` payment gateway error · `503` service not configured

## How the payment works

```
1. Customer clicks "Pay"          → POST /api/orders
2. Server checks stock, calculates the total, reserves stock,
   saves the order (status: placed, payment: pending),
   creates a Razorpay order for the same amount
3. Frontend opens the Razorpay popup with that Razorpay order id
4. Customer pays                  → Razorpay returns order_id, payment_id, signature
5. Frontend sends them            → POST /api/orders/:id/verify-payment
6. Server computes HMAC-SHA256(order_id + "|" + payment_id, KEY_SECRET)
   and compares it with the signature
7. Match → payment: paid, status: confirmed. No match → payment: failed
```

The amount always comes from the server, and an order is marked paid only after the signature check,
so a user cannot change the price or fake a payment from the browser. If the popup is closed, the order
stays unpaid and the customer can pay later from the order page.

## Order status flow

```
placed ──► confirmed ──► shipped ──► delivered
   │            │
   └────────────┴──► cancelled   (stock is returned; paid online orders become refund_pending)
```

- The server rejects any other move (e.g. `delivered` → `placed`).
- An online order cannot be confirmed or shipped until it is paid.
- Cash on Delivery orders are marked paid when delivered.
- Customers can cancel only while the order is `placed` or `confirmed`.

## Security

- Passwords hashed with bcrypt; the hash is never returned (`select: false`).
- Registration always creates a `customer`; the role is never read from the request.
- Every protected route checks the JWT and then the role on the server. Frontend route guards are only for UX.
- Prices, shipping and totals are calculated on the server from the database.
- Stock is reserved with an atomic update (`stock >= quantity` inside the update), so two buyers cannot both get the last item.
- Razorpay signatures are compared with `crypto.timingSafeEqual`.
- Customers can only read their own orders and invoices.
- express-validator on every input; search text is regex-escaped; sorting uses a whitelist.
- Uploads: images only (JPG, PNG, WEBP), 2 MB each, max 5, kept in memory and streamed to Cloudinary.
- helmet headers, CORS allow-list, rate limits (stricter on `/api/auth`), 10 kb JSON limit.
- HTML in emails is escaped.

## Deployment

### 1. Database – MongoDB Atlas
Create a free cluster → Database Access: add a user → Network Access: allow `0.0.0.0/0` →
Connect → Drivers → copy the connection string and add the database name: `.../mern_store`.
Run `npm run seed` locally once with this `MONGO_URI` to fill the live database.

### 2. Backend – Render
New → Web Service → connect the GitHub repo →
**Root directory:** `backend` · **Build:** `npm install` · **Start:** `npm start` →
add every backend environment variable (`NODE_ENV=production`; leave `CLIENT_URL` for step 4).
Copy the URL, e.g. `https://mern-store-api.onrender.com`.
Free Render services sleep after inactivity; the first request can take up to a minute.

### 3. Frontend – Vercel
New Project → import the repo → **Root directory:** `frontend` (Vite is detected) →
environment variable `VITE_API_URL=https://mern-store-api.onrender.com/api` → Deploy.
`vercel.json` sends every path to `index.html` so page refreshes work.

### 4. Connect them
On Render set `CLIENT_URL` to the Vercel URL (no trailing slash) and redeploy.
If `VITE_API_URL` changes later, redeploy the frontend: Vite puts it in the build.

## Screenshots

_Add: home, product list with filters, product page, cart, checkout, Razorpay popup, order tracking,
invoice PDF, admin dashboard, products, orders, sales report, mobile view._
