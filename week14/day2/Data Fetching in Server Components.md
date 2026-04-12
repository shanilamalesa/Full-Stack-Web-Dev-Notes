# Week 14, Day 2: Data Fetching in Server Components

By the end of today, your Next.js shop will read a `products` table from the Week 12 Postgres database directly inside a Server Component, without a single `/api` route, without `useEffect`, without a loading spinner. You will also understand how Next.js caches `fetch` calls, how to opt out of caching, and how `loading.js` files give you streaming UI for free.

**Prior-week concepts you will use today:**
- Server vs Client Components (Week 14, Day 1)
- Postgres + the `pg` driver + connection pools (Week 12, Day 2)
- SQL, prepared statements, indexes (Week 12, Day 1)
- async/await (Week 4)

**Estimated time:** 3-4 hours

---

## The Big Shift

In the Week 11-12 React dashboard you did this on every page that needed data:

```jsx
function LeadsPage() {
  const [leads, setLeads] = useState(null);
  useEffect(() => {
    fetch("/api/leads").then(r => r.json()).then(data => setLeads(data.leads));
  }, []);

  if (!leads) return <p>Loading...</p>;
  return <LeadsTable leads={leads} />;
}
```

Three lines of state, one loading guard, one API endpoint, one network round-trip, one hydration cost. For every data-bound page. Across the dashboard that was dozens of repetitions.

In Next.js 14 with the App Router, the same thing is:

```jsx
async function LeadsPage() {
  const { rows: leads } = await db.query("SELECT * FROM leads");
  return <LeadsTable leads={leads} />;
}
```

No `useState`. No `useEffect`. No loading guard. No API endpoint in between. The component *is* `async`, runs on the server, awaits the query, returns JSX. The browser gets HTML with the leads already inside it.

This is not a trick. This is the design. A Server Component can be `async`, and Next.js will await it before rendering. Everything that used to require a backend+frontend round-trip collapses.

---

## Setting Up The Database Connection

The shop lives in a separate project from the Week 12 CRM, but it talks to the **same Postgres database**. Both apps read and write to the same `crm` database on `localhost:5432`. This is the "one backend, many frontends" shape from yesterday.

From inside the `shop/` folder:

```bash
npm install pg
```

Create `lib/db.js`:

```javascript
// lib/db.js
import { Pool } from "pg";

const globalForPool = globalThis;

export const pool =
  globalForPool._pool ||
  new Pool({
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT, 10),
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    max: 10,
  });

if (process.env.NODE_ENV !== "production") globalForPool._pool = pool;

export async function query(text, params) {
  return pool.query(text, params);
}
```

Two things are new compared to the Week 12 CRM `db.js`:

**`globalThis._pool` trick.** In dev mode, Next.js hot-reloads your code every time you save. Without the global cache, every reload would create a *new* `Pool`, and within an hour you would have hundreds of lingering Postgres connections and a crashed dev server. Stashing the pool on `globalThis` means the same pool is reused across reloads. In production this does nothing because there are no reloads.

**Named exports, ES modules.** Next.js uses ESM by default. `import/export` syntax everywhere, not `require/module.exports`. If you see a tutorial with `require`, it is either old or using the Pages Router. Ignore.

Create `.env.local` (Next.js reads `.env.local` by convention):

```env
DB_HOST=localhost
DB_PORT=5432
DB_USER=crm_user
DB_PASSWORD=crm_dev_password
DB_NAME=crm
```

Note the file is `.env.local`, not `.env`. Next.js git-ignores `.env.local` automatically.

---

## Adding A `products` Table

The shop needs products. Open `psql` against the `crm` database and create:

```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  description TEXT,
  price_cents INTEGER NOT NULL CHECK (price_cents >= 0),
  image_url TEXT,
  stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
  category TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_slug ON products(slug);

INSERT INTO products (slug, name, description, price_cents, image_url, stock, category) VALUES
  ('nokia-105', 'Nokia 105', 'Dual SIM feature phone with FM radio. Lasts days on one charge.', 250000, '/products/nokia-105.jpg', 47, 'phones'),
  ('tecno-spark-10', 'Tecno Spark 10', 'Budget Android with 6.6 inch screen and 50MP camera.', 1599900, '/products/tecno-spark.jpg', 12, 'phones'),
  ('samsung-a05', 'Samsung A05', 'Entry-level Samsung with solid build and good battery.', 1899900, '/products/samsung-a05.jpg', 8, 'phones'),
  ('infinix-hot-40', 'Infinix Hot 40', 'Gamer-friendly with 120Hz display and 90W fast charging.', 2499900, '/products/infinix-hot-40.jpg', 5, 'phones'),
  ('oraimo-powerbank', 'Oraimo 20000mAh Power Bank', 'Charges your phone five times. Fast charging in and out.', 250000, '/products/oraimo-pb.jpg', 30, 'accessories');
```

Two decisions worth pointing at.

**`price_cents INTEGER`**, not `DECIMAL`. Money is always stored as the smallest unit (cents for KSh, since KSh has no sub-unit use 100ths). Floating-point money is a generational bug. KSh 2,499.99 becomes `249999`. Every display layer divides by 100 at render time. Every payment provider works this way.

**`slug TEXT UNIQUE`** -- a URL-friendly identifier. You will route by slug (`/products/nokia-105`) instead of UUID because `/products/c5f3a1b2-...-...` is bad for SEO, bad for users, and bad for every list of URLs you paste into a WhatsApp message.

Save the table creation to `shop/db/products.sql` so future-you can recreate it.

---

## Your First Server Component With Real Data

Create `app/products/page.js`:

```jsx
// app/products/page.js
import Link from "next/link";
import { query } from "@/lib/db";

export const metadata = {
  title: "All Products | Mctaba",
};

export default async function ProductsPage() {
  const { rows: products } = await query(
    "SELECT id, slug, name, price_cents, image_url, stock FROM products ORDER BY created_at DESC"
  );

  return (
    <div className="max-w-5xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">All Products</h1>
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        {products.map((p) => (
          <Link key={p.id} href={`/products/${p.slug}`} className="border rounded p-4 hover:shadow">
            <div className="aspect-square bg-gray-100 mb-3" />
            <div className="font-medium">{p.name}</div>
            <div className="text-gray-600 text-sm mt-1">
              KSh {(p.price_cents / 100).toLocaleString()}
            </div>
            {p.stock === 0 && <div className="text-red-600 text-sm mt-1">Out of stock</div>}
          </Link>
        ))}
      </div>
    </div>
  );
}
```

Run `npm run dev`, visit `/products`. You should see five cards with the product names and prices. The component function is `async`, awaits the query, renders JSX. That is *it*.

Notice three small things:

**`import { query } from "@/lib/db"`.** The `@` alias points at the project root -- set by Next.js by default. Use it for everything; relative imports get ugly fast.

**`price_cents / 100`** with `.toLocaleString()`. Display formatting at render time. Never store formatted money.

**No `loading...` fallback.** The page does not show until the query finishes. For a fast local query that is fine -- the user sees the full product list on first paint. For slow queries we will use `loading.js` below.

---

## The `loading.js` Convention

Next.js has a special file name: `loading.js` in any route folder. When Next renders that route, if the Server Component is `async` and taking time, Next immediately shows `loading.js` as a placeholder and streams the real content in when the data arrives.

Create `app/products/loading.js`:

```jsx
// app/products/loading.js
export default function Loading() {
  return (
    <div className="max-w-5xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">All Products</h1>
      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        {Array.from({ length: 6 }).map((_, i) => (
          <div key={i} className="border rounded p-4">
            <div className="aspect-square bg-gray-200 animate-pulse mb-3" />
            <div className="h-4 bg-gray-200 animate-pulse rounded w-3/4" />
            <div className="h-3 bg-gray-200 animate-pulse rounded w-1/2 mt-2" />
          </div>
        ))}
      </div>
    </div>
  );
}
```

The local query is too fast to notice it. Simulate slowness by adding a `setTimeout` inside the data fetch:

```jsx
const { rows: products } = await query("SELECT ...");
await new Promise(r => setTimeout(r, 2000));
```

Reload. You now see the skeleton grid for two seconds, then the real content. Remove the delay after you see it working.

This is called **streaming UI**. Next.js sends the HTML shell of the page immediately, including the `loading.js` placeholder, and replaces it with the real content when the server finishes. The user sees something in ~100ms instead of staring at a blank browser tab.

### Skeleton screens vs spinners

Those gray pulsing boxes are a **skeleton screen**. They are strictly better than spinners because they tell the user *what is loading* (the shape of the content) and reduce the perceived wait. Pick skeleton screens over spinners every time in production apps. The Week 11 dashboard should eventually get them too.

---

## The `fetch` Cache

When you use the built-in `fetch()` in a Server Component, Next.js silently caches the response by default. Try this:

```jsx
// somewhere in a page
const res = await fetch("https://api.example.com/rates");
const data = await res.json();
```

Next.js fetches once, caches the result, and every subsequent request for the same URL across every visitor gets the cached copy. In dev mode the cache is per-request; in production it is shared. This is great for external data that does not change often (exchange rates, weather, a blog post list).

### Opting out

For our product queries, we are using `pg` directly -- not `fetch` -- so the `fetch` cache does not apply. Database queries run on every request. If you want to cache a database query, you use `unstable_cache`:

```jsx
import { unstable_cache } from "next/cache";
import { query } from "@/lib/db";

const getProducts = unstable_cache(
  async () => {
    const { rows } = await query("SELECT * FROM products ORDER BY created_at DESC");
    return rows;
  },
  ["products"],          // cache key
  { revalidate: 60 }     // seconds
);

export default async function ProductsPage() {
  const products = await getProducts();
  // ...
}
```

The call is cached for 60 seconds across all visitors. The next request after 60 seconds triggers a fresh query.

For an e-commerce site with slow-changing products, 60 seconds is a reasonable default. For stock counts (which change on every sale), you want 0 -- always fresh. For content pages (about, contact), you want hours.

### Manual invalidation

When an admin updates a product, you want the cache to clear immediately -- not wait 60 seconds. That is what `revalidatePath` is for:

```javascript
import { revalidatePath } from "next/cache";

// after updating a product in an admin form:
revalidatePath("/products");
```

We will use this heavily in Week 15 when orders are placed and stock changes.

For today, skip caching entirely -- run queries fresh on every request. Caching is an optimisation and you optimise last.

---

## A Few Anti-Patterns To Avoid

**Do not create an `/api/products` route just to call it from a Server Component.** The Server Component can hit the database directly. An API route only makes sense when a *client* (a browser, another service) needs to call it.

**Do not put `"use client"` on a page just because one child needs state.** Keep the page a Server Component, isolate the interactive piece into a small Client Component, and pass props down. The cost of marking a whole page client-only is bundle size and loss of server-side data access.

**Do not `await` in a `.map()` inside render.** If you find yourself writing `{products.map(async (p) => await somethingAsync(p))}`, stop. Fetch all the data you need at the top of the component, then map over it synchronously.

**Do not `fetch` your own API routes from Server Components.** Same database, same process, same deploy. Call the function directly. This is one of the top confusions for people coming from CRA+Express.

---

## A Tiny Detour: `<Image>`

Next.js ships a custom `<Image>` component that does a lot for you: automatic size generation, lazy loading, modern formats (WebP/AVIF), and a blur placeholder. You should use it for every product photo.

Drop some placeholder images in `public/products/` -- download any five phone images from the internet and rename them to match your slugs, or use https://placehold.co/400x400 URLs.

In the product card:

```jsx
import Image from "next/image";

<Image
  src={p.image_url}
  alt={p.name}
  width={400}
  height={400}
  className="aspect-square object-cover"
/>
```

Next.js will serve a size-appropriate version to each visitor (a mobile user gets a smaller file). For remote image URLs you have to whitelist the domain in `next.config.js`:

```javascript
module.exports = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "placehold.co" },
    ],
  },
};
```

Without that whitelist, `<Image>` refuses to load remote URLs -- a deliberate defence against hosting someone else's images on your CDN.

---

## Checkpoint

1. `/products` shows five cards pulled from the database.
2. Stopping Postgres and reloading `/products` results in a clear 500 error from Next.js -- not a silent blank page. (If it blanks, add a top-level `try/catch` with a fallback UI.)
3. Adding a `setTimeout(2000)` delay in the query makes the skeleton grid appear for two seconds, then the real data.
4. The HTML source of `/products` contains the product names and prices *in the initial response* -- not fetched after page load. View source to confirm.
5. Changing a product name in `psql` and reloading shows the new name immediately (because we are not caching).
6. The `pg` pool is reused across dev reloads -- `SELECT count(*) FROM pg_stat_activity WHERE datname='crm';` stays stable after many saves instead of climbing.

Commit:

```bash
git add .
git commit -m "feat: product listing page backed by postgres server components"
```

---

## What You Learned

- Server Components can be `async` and fetch data directly.
- The database lives on one side of the wall, the shop on the other, no REST layer in between.
- `loading.js` gives you streaming skeletons with zero code.
- `unstable_cache` and `revalidatePath` control caching without a Redis layer for now.
- The `pg` pool needs a global-cache hack in dev mode.
- `<Image>` and a remote-patterns whitelist handle product photos.

Tomorrow we add dynamic routes -- `/products/[slug]` -- so each product has its own detail page with a real URL you can share on WhatsApp.
