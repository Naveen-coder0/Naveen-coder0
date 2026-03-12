# MELINI — Full Project Context

## What Is This Project?

MELINI is a full-stack, production-ready Indian fashion e-commerce platform. It sells premium comfort wear across three seasonal categories — Summer Wear, Semi-Winter Wear, and Winter Wear. The storefront is a React 18 + TypeScript SPA, backed by a Node.js/Express REST API connected to MongoDB, with Razorpay as the payment gateway and Cloudinary for image hosting.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend framework | React 18 + TypeScript |
| Build tool | Vite |
| Styling | Tailwind CSS + PostCSS |
| UI components | shadcn/ui (30+ components) + Radix UI primitives |
| Animation | Framer Motion |
| State management | React Context API (5 contexts) |
| HTTP / data fetching | TanStack React Query v5 |
| Forms & validation | React Hook Form + Zod |
| Payment gateway | Razorpay (INR) |
| 3D graphics | Three.js + React Three Fiber (dependency available) |
| Notifications | Sonner + shadcn toast |
| Backend runtime | Node.js + Express |
| Database | MongoDB (via Mongoose) |
| Image hosting | Cloudinary |
| Authentication | JWT (8-hour admin sessions) |
| Package manager | Bun |
| Testing | Vitest + Testing Library |
| Linting | ESLint |
| Deployment | Vercel (frontend + serverless API functions) |

---

## Repository Structure

```
MELINI/
├── api/                        # Vercel serverless API handlers
│   ├── [...path].ts            # Catch-all proxy to Express backend
│   └── create-order.ts         # Razorpay order creation endpoint
├── backend/
│   ├── index.js                # Full Express server (all API routes)
│   └── package.json
├── public/
│   └── robots.txt
├── src/
│   ├── App.tsx                 # Root component — providers + routing
│   ├── main.tsx                # Entry point
│   ├── index.css               # Global styles
│   ├── assets/                 # Static images / SVGs
│   ├── data/
│   │   └── products.ts         # Product/Category TypeScript interfaces + static data
│   ├── lib/
│   │   ├── auth.ts             # JWT token helpers + adminFetch wrapper
│   │   ├── productApi.ts       # REST calls for products (CRUD)
│   │   └── utils.ts            # cn() class-name utility
│   ├── hooks/
│   │   ├── use-mobile.tsx      # Mobile breakpoint hook
│   │   └── use-toast.ts        # Toast notification hook
│   ├── contexts/
│   │   ├── CartContext.tsx     # Cart state + localStorage persistence
│   │   ├── ProductContext.tsx  # Product list from API + localStorage cache
│   │   ├── AuthContext.tsx     # Admin JWT auth state
│   │   ├── WishlistContext.tsx # Wishlisted product IDs + localStorage
│   │   └── ConfigContext.tsx   # Live site configuration from API
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.tsx      # Sticky header, nav, cart icon, wishlist icon
│   │   │   ├── Footer.tsx      # Newsletter form, footer links, social links
│   │   │   └── Layout.tsx      # Wraps all public pages
│   │   ├── home/
│   │   │   ├── HeroSection.tsx
│   │   │   ├── CategoryShowcase.tsx
│   │   │   ├── NewArrivalsSection.tsx
│   │   │   ├── BestSellersSection.tsx
│   │   │   ├── FeaturedProducts.tsx
│   │   │   ├── PromoBanner.tsx
│   │   │   ├── SeasonalCollection.tsx
│   │   │   ├── BrandStory.tsx
│   │   │   └── CustomerStats.tsx
│   │   ├── shop/
│   │   │   ├── ProductCard.tsx  # Card for grid/list display
│   │   │   └── ProductFilters.tsx # Category, size, price, stock filters
│   │   ├── admin/
│   │   │   ├── DashboardTab.tsx
│   │   │   ├── ProductsTab.tsx
│   │   │   ├── OrdersTab.tsx
│   │   │   ├── MarketingTab.tsx
│   │   │   ├── ContentTab.tsx
│   │   │   ├── ReviewsTab.tsx
│   │   │   └── SettingsTab.tsx
│   │   ├── ui/                  # shadcn/ui component library (30+ files)
│   │   ├── AnnouncementBar.tsx  # Dismissible top announcement banner
│   │   ├── BackToTop.tsx        # Floating back-to-top button
│   │   ├── CartDrawer.tsx       # Slide-in cart panel from header
│   │   ├── NavLink.tsx          # Active-state nav link wrapper
│   │   ├── ProtectedAdminRoute.tsx # Redirects to /admin-login if not authed
│   │   ├── ProtectedRoute.tsx   # Generic protected route guard
│   │   ├── RecentlyViewed.tsx   # Horizontal scroll of last-viewed products
│   │   ├── ScrollProgressBar.tsx # Thin progress bar fixed at top of screen
│   │   ├── ScrollToTop.tsx      # Scrolls window to top on route change
│   │   └── SizeGuideModal.tsx   # Size chart dialog (XS–XXL measurements)
│   ├── pages/
│   │   ├── Index.tsx            # Home page
│   │   ├── Shop.tsx             # Product catalog
│   │   ├── ProductDetails.tsx   # Single product view
│   │   ├── Category.tsx         # Category-filtered product list
│   │   ├── Cart.tsx             # Full cart page
│   │   ├── Checkout.tsx         # Checkout + Razorpay + WhatsApp confirmation
│   │   ├── Search.tsx           # Full-text product search
│   │   ├── About.tsx            # Brand story + timeline + values
│   │   ├── Contact.tsx          # Contact form + contact info
│   │   ├── Admin.tsx            # Admin dashboard shell
│   │   ├── AdminLogin.tsx       # Admin login form
│   │   └── NotFound.tsx         # 404 page
│   └── test/
│       ├── example.test.ts
│       └── setup.ts
├── vercel.json                  # SPA rewrite rules for Vercel
├── vite.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## Routing

All public pages are wrapped in the `<Layout>` component (Header + Footer). Admin pages are outside Layout.

| Route | Component | Auth Required |
|---|---|---|
| `/` | `Index` | No |
| `/shop` | `Shop` | No |
| `/product/:slug` | `ProductDetails` | No |
| `/category/:category` | `Category` | No |
| `/cart` | `Cart` | No |
| `/checkout` | `Checkout` | No |
| `/about` | `About` | No |
| `/contact` | `Contact` | No |
| `/search` | `Search` | No |
| `/admin-login` | `AdminLogin` | No |
| `/admin` | `Admin` | Yes — JWT token via `ProtectedAdminRoute` |
| `*` | `NotFound` | No |

---

## State Management — React Contexts

### CartContext
- Stores `CartItem[]` (id, name, price, image, size, color, quantity, slug)
- Persisted to `localStorage` under key `melini-cart`
- Methods: `addItem`, `removeItem`, `updateQuantity`, `clearCart`
- Computed: `totalItems`, `totalPrice`

### ProductContext
- Fetches products from `GET /api/products` on mount
- Falls back to `localStorage` cache (`melini_products`) when API is unavailable
- Syncs across tabs via `storage` event listener
- Methods: `refreshProducts`, `addProduct`, `updateProduct`, `deleteProduct`, `getProductBySlug`, `getProductsByCategory`, `getFeaturedProducts`

### AuthContext
- Stores admin JWT in `localStorage` under key `melini_admin_token`
- Methods: `login(token)`, `logout()`
- `isAuthenticated` derived from whether a valid token is stored

### WishlistContext
- Stores an array of product IDs
- Persisted to `localStorage` under key `melini_wishlist`
- Methods: `toggleWishlist(id)`, `isWishlisted(id)`
- Computed: `count`

### ConfigContext
- Fetches live site configuration from `GET /api/site-config` on mount
- Provides: `heroTitle`, `heroSubtitle`, `heroBadge`, `announcement`, `storeName`, `tagline`, `contactEmail`, `whatsapp`, `instagram`, `freeShippingThreshold`, `currency`, `promoBannerUrl`, `promoLink`
- Defaults: store name "MELINI", threshold ₹999, currency "INR"
- `refreshConfig()` re-fetches and propagates changes globally

---

## Pages — Detail

### Home (`/`)
Composed of the following sections in order:
1. **HeroSection** — Large branded banner with dynamic `heroTitle`, `heroSubtitle`, `heroBadge` from `ConfigContext`
2. **CategoryShowcase** — Cards for Summer, Semi-Winter, Winter
3. **NewArrivalsSection** — Products filtered by `isNewProduct = true`
4. **BestSellersSection** — Products filtered by `isBestSeller = true`
5. **FeaturedProducts** — Curated featured products grid
6. **PromoBanner** — Dynamic promotional banner from admin-configured `promoBannerUrl`
7. **SeasonalCollection** — Seasonal-themed product display
8. **BrandStory** — Copy about the brand's origins
9. **CustomerStats** — Social proof stats (customer count, orders, etc.)

### Shop (`/shop`)
- Pulls product list from `ProductContext`
- URL-parameter-driven filtering: `?category=`, `?sizes=`, `?minPrice=`, `?maxPrice=`, `?inStock=`, `?sort=`
- Sort options: newest, best-selling, price-low, price-high
- View modes: grid (2–3 columns) or list
- Desktop: sidebar filters; Mobile: slide-out `Sheet` with filters
- Shows loading spinner and "no products found" empty state

### Product Details (`/product/:slug`)
- Looks up product by slug via `ProductContext`
- Image gallery with thumbnail strip; active images change per selected color
- Image hover zoom with mouse-tracking
- Size selector — required before adding to cart
- Color selector with per-color image sets
- Per-size pricing support (price and originalPrice change by selected size)
- Discount percentage calculated from `originalPrice`
- Price range display if sizes have varying prices
- Stock count — shows "Low Stock" badge when `stockCount < 5`
- Wishlist (heart) toggle
- Add to cart validates size selection before adding
- Breadcrumb navigation
- `ScrollProgressBar` at top
- `SizeGuideModal` accessible via link
- `RecentlyViewed` section at bottom (up to 6 products from localStorage)
- Tabs: Description, Materials & Care, Features, Customer Reviews
- Related products section (same category, up to 4)

### Cart (`/cart`)
- Lists all cart items with image, name, size, color, quantity controls, price
- Inline quantity increment/decrement (decrement to 0 removes item)
- Per-item removal button
- "Clear Cart" button
- Order Summary sidebar: subtotal, shipping (free above configurable threshold), total
- Accepted payment icons (Visa, Mastercard, UPI)
- Empty state with link back to shop

### Checkout (`/checkout`)
Multi-step flow: `details` → `payment` → `success`

**Details step:**
- Form fields: First name, Last name, Email, Phone, Address, City, State, Pincode
- Coupon code input — calls `POST /api/coupons/validate` with the order amount; shows discount line in summary when applied
- Order summary sidebar with subtotal, coupon discount, shipping, total
- Shipping threshold from `ConfigContext`

**Payment step:**
- Calls `POST /api/create-order` to create a Razorpay order
- Opens Razorpay checkout modal
- On success handler:
  1. Saves full order to database via `POST /api/orders`
  2. Redirects user to WhatsApp with pre-filled order details to the shop owner's configured WhatsApp number

### Search (`/search`)
- Full-text search across product name, description, category, and material fields
- Query driven via `?q=` URL parameter
- Real-time filtering as user types

### About (`/about`)
- Brand story in a hero image section
- Mission statement
- Four value pillars (Crafted with Love, Sustainable Fashion, Community First, Premium Quality)
- Timeline of the brand from 2018 to 2024

### Contact (`/contact`)
- Contact form (name, email, phone, subject, message) — currently simulates submission with a 1.5 s delay + toast
- Contact info cards: address, phone (from config), email (from config), working hours

### Admin Login (`/admin-login`)
- Dark glassmorphism design
- Calls `POST /api/admin/login` with username + password
- On success: stores JWT in localStorage

---

## Admin Panel (`/admin`)

Protected by `ProtectedAdminRoute` (redirects to `/admin-login` if no valid token). Responsive sidebar navigation with 7 tabs.

### Dashboard Tab
- KPI cards: Total Revenue, Orders, Products, Out-of-Stock count
- Area chart: Revenue for last 7 days (Recharts)
- Pie chart: Inventory breakdown by category (Recharts)
- Recent orders list

### Products Tab
- Full product list with search
- Add product form / Edit modal with all fields:
  - Name, slug, article no, description, short description
  - Category (summer / semi-winter / winter)
  - Price, original price (for discount display)
  - Per-size pricing overrides (`sizePricing[]`)
  - Images (Cloudinary upload)
  - Colors with per-color image sets
  - Sizes (XS/S/M/L/XL/XXL), stock count
  - Material, care instructions, features
  - SEO: meta title, meta description, tags
  - Flags: isNewProduct, isBestSeller, inStock
- Delete product with confirmation

### Orders Tab
- Lists all orders with customer name, items, total, date, status
- Status filter: all / pending / processing / shipped / delivered / cancelled
- Search by customer name or order ID
- Inline status update dropdown per order
- Status badges with distinct color coding

### Marketing Tab
- Coupon management
- Create coupon form: code (uppercase), discount type (percentage / fixed), discount value, min order amount, max discount, expiry date, usage limit
- Toggle coupon active/inactive
- Delete coupon
- Usage count displayed per coupon

### Content Tab
- Edit homepage content: hero badge, hero title, hero subtitle
- Edit announcement bar text
- Edit promotional banner (image URL + link)
- Changes saved via `PUT /api/admin/site-config` and propagated via `refreshConfig()`

### Reviews Tab
- Lists all customer reviews
- Filter: All / Pending / Approved
- Per-review: approve/unapprove toggle, delete
- Displays: reviewer, star rating, date, product name, comment

### Settings Tab
- Store identity: store name, tagline
- Announcement bar text
- Free shipping threshold (₹)
- Currency (INR / USD)
- Contact email
- WhatsApp number
- Instagram handle

---

## Backend API — Express Server (`backend/index.js`)

### MongoDB Schemas

| Schema | Key Fields |
|---|---|
| **Product** | name, slug, price, originalPrice, sizePricing[], images[], colors[{name, value, images[]}], sizes[], category, inStock, isBestSeller, isNewProduct, material, careInstructions[], features[], articleNo, metaTitle, metaDescription, tags[], stockCount |
| **Order** | orderItems[], shippingAddress, paymentMethod, paymentResult, customer{name,email,phone}, itemsPrice, shippingPrice, discountAmount, coupon, totalPrice, isPaid, isDelivered, status (pending/processing/shipped/delivered/cancelled), orderNotes |
| **Coupon** | code (uppercase unique), discountType (percentage/fixed), discountValue, minOrderAmount, maxDiscountAmount, usageLimit, usedCount, expiresAt, isActive |
| **Review** | product (ref), user, rating (1–5), comment, images[], isApproved |
| **SiteConfig** | heroTitle, heroSubtitle, heroBadge, announcement, promoBannerUrl, promoLink, storeName, tagline, contactEmail, whatsapp, instagram, freeShippingThreshold, currency, socialLinks, contactInfo, seoDefaults |
| **Settings** | storeName, tagline, contactEmail, whatsapp, instagram, announcementBar, freeShippingThreshold, currency |

### Authentication
- Admin login returns a JWT signed with `JWT_SECRET` (8-hour expiry)
- `verifyAdmin` middleware checks `Authorization: Bearer <token>` on all admin routes

### Public API Routes

| Method | Route | Description |
|---|---|---|
| GET | `/api/products` | List all products |
| POST | `/api/coupons/validate` | Validate a coupon code against an order amount |
| GET | `/api/site-config` | Fetch current site configuration |
| POST | `/api/create-order` | Create a Razorpay order |
| POST | `/api/orders` | Save a completed order to the database |

### Admin API Routes (require JWT)

| Method | Route | Description |
|---|---|---|
| POST | `/api/admin/login` | Authenticate and receive JWT |
| GET | `/api/admin/products` | List products (same as public) |
| POST | `/api/admin/products` | Create a new product |
| PUT | `/api/admin/products/:id` | Update a product |
| DELETE | `/api/admin/products/:id` | Delete a product |
| POST | `/api/admin/products/upload-image` | Upload image to Cloudinary |
| GET | `/api/admin/orders` | List all orders |
| PATCH | `/api/admin/orders/:id` | Update order status |
| GET | `/api/admin/coupons` | List all coupons |
| POST | `/api/admin/coupons` | Create a coupon |
| PATCH | `/api/admin/coupons/:id` | Toggle coupon active status |
| DELETE | `/api/admin/coupons/:id` | Delete a coupon |
| GET | `/api/admin/reviews` | List all reviews |
| PUT | `/api/admin/reviews/:id` | Approve or update a review |
| DELETE | `/api/admin/reviews/:id` | Delete a review |
| GET | `/api/admin/settings` | Fetch store settings |
| PUT | `/api/admin/settings` | Update store settings |
| GET | `/api/admin/site-config` | Fetch site config |
| PUT | `/api/admin/site-config` | Update site config |

---

## Key Features — Summary

| Feature | How It Works |
|---|---|
| **Persistent Cart** | LocalStorage (`melini-cart`), synced on every state change |
| **Persistent Wishlist** | LocalStorage (`melini_wishlist`), toggle per product |
| **Offline Product Cache** | LocalStorage (`melini_products`), updated on API success |
| **Announcement Bar** | Pulls text from `ConfigContext`; dismissible via `sessionStorage` |
| **Cart Drawer** | Slide-in Sheet from the right, triggered by cart icon in header |
| **Product Image Zoom** | Mouse-position tracking on hover in product detail |
| **Per-Color Images** | Selecting a color swaps the entire image gallery |
| **Per-Size Pricing** | Price and discount recalculate when user selects a size |
| **Low Stock Badge** | Shows when `stockCount > 0` and `stockCount < 5` |
| **Recently Viewed** | Saved to localStorage (max 6), displayed on product detail page |
| **Size Guide Modal** | Dialog with chest/waist/hip/inseam table (XS–XXL) |
| **Scroll Progress Bar** | Fixed 2px bar at top, tied to scroll position via Framer Motion |
| **Back to Top Button** | Appears after scrolling down; smooth scroll back to top |
| **Coupon System** | Real-time API validation at checkout; deducts from order total |
| **Razorpay Integration** | SDK loaded from CDN; order created server-side, payment in modal |
| **WhatsApp Order Alert** | After successful payment, a `wa.me` link is generated with full order details and the user is redirected to it |
| **Admin JWT Auth** | Token stored in localStorage; auto-cleared and redirected on 401 |
| **Cloudinary Images** | Admin can upload images; URLs stored in product documents |
| **Live Config Reload** | After admin saves content/settings, `refreshConfig()` updates all UI without a page reload |
| **Cross-Tab Product Sync** | product list updated in all open tabs via `window.storage` event |
| **SEO per Product** | metaTitle, metaDescription, tags fields on each product |

---

## Product Data Model

```typescript
interface Product {
  id: string;
  name: string;
  slug: string;
  articleNo?: string;           // SKU
  price: number;                // base / default price
  originalPrice?: number;       // for showing crossed-out price
  sizePricing?: {               // overrides price per size
    size: string;
    price: number;
    originalPrice?: number;
  }[];
  description: string;
  shortDescription: string;
  category: 'summer' | 'semi-winter' | 'winter';
  images: string[];
  sizes: string[];              // e.g. ['XS','S','M','L','XL','XXL']
  colors: {
    name: string;
    value: string;              // hex color
    images?: string[];          // per-color image set
  }[];
  inStock: boolean;
  isNewProduct?: boolean;
  isBestSeller?: boolean;
  material: string;
  careInstructions: string[];
  features: string[];
  metaTitle?: string;
  metaDescription?: string;
  tags?: string[];
  stockCount?: number;
}
```

---

## Environment Variables

### Frontend (`.env`)
| Variable | Purpose |
|---|---|
| `VITE_API_URL` | Base URL for the backend API |
| `VITE_RAZORPAY_KEY` | Razorpay public key ID |

### Backend (`.env`)
| Variable | Purpose |
|---|---|
| `PORT` | Express server port (default 5000) |
| `MONGODB_URI` | MongoDB connection string |
| `JWT_SECRET` | Secret for signing admin JWT tokens |
| `ADMIN_USERNAME` | Admin login username |
| `ADMIN_PASSWORD` | Admin login password |
| `RAZORPAY_KEY_ID` | Razorpay key ID |
| `RAZORPAY_KEY_SECRET` | Razorpay key secret |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |

---

## Development Commands

```bash
# Install dependencies
bun install

# Start frontend dev server
bun run dev

# Build for production
bun run build

# Run tests
bun run test

# Lint
bun run lint
```

---

## Deployment

- Hosted on **Vercel**
- `vercel.json` rewrites all non-`/api/` routes to `index.html` (SPA mode)
- The `api/` directory contains Vercel serverless function handlers
- The `backend/` Express server can be deployed separately (e.g. Railway, Render) as a standalone Node.js service
