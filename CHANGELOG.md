# Changelog

## Unreleased

### Changed
- Halaman **Kategori** dirancang ulang: hero dengan grafik distribusi produk, 4 kartu ringkasan, kartu per kategori (jumlah produk, unit stok, nilai modal, porsi katalog, produk di dalamnya, peringatan stok menipis), pencarian + urutan, kartu "Kategori baru", dan peringatan untuk produk tanpa kategori. Datanya kini terhubung ke produk (`Categories` menerima `products` dan `onAdd`).
- Halaman **Inventory** dirancang ulang: hero dengan donat kesehatan stok, kartu ringkasan (total unit, nilai persediaan, menipis, habis), panel "Perlu perhatian" dengan tombol restock +10, filter status + filter kategori + urutan, baris produk dengan indikator level stok, stepper −/+ dan +10. Logika penyesuaian stok (compare-and-swap) tidak diubah. Kolom "Update: Baru saja" yang palsu diganti nilai modal per produk.
- `compactNumber()` dipindah ke `lib/format.ts` (dipakai halaman Pelanggan, Kategori, Inventory) dan dites.

### Removed
- ±840 baris CSS lama untuk halaman Inventory/Kategori (`inventory-*`, `category-row`, dll.); tampilan halaman lain diverifikasi identik per piksel (terang & gelap).

### Added
- Dark mode lengkap dan konsisten di semua halaman (Overview, Kasir, Riwayat, Analisis, Produk, Kategori, Pelanggan, Inventory, modal, pemilih pelanggan). Tombol matahari/bulan di bilah atas dan halaman **Pengaturan** (Terang / Gelap / Otomatis mengikuti perangkat). Pilihan disimpan di perangkat, diterapkan sebelum render pertama (tanpa kedip putih), dengan transisi halus.
- `scripts/generate-dark-theme.mjs` menghasilkan `app/dark-theme.generated.css` dari `globals.css` (jalan otomatis lewat `predev`/`prebuild`; manual: `npm run theme:build`). Tweak manual ada di `app/dark-overrides.css`. Tes memastikan berkas hasil selalu terbaru.

### Changed
- Tema kini dipasang di `<html>` (class `dark`), bukan di `.shell`, sehingga elemen portal (pemilih pelanggan) ikut tema. Aturan `.shell.dark` lama (termasuk yang memaksa Analytics tetap terang) dihapus.

### Added
- Menu di form pelanggan selalu mengikuti katalog: produk baru dari Produk/Kasir, perangkat lain, atau tab lain muncul otomatis (Realtime `products`/`categories`, refresh saat tab aktif dan saat form dibuka). Migrasi `supabase/migrations/004_realtime_catalog.sql` mengaktifkan Realtime untuk tabel katalog. Menu bertanda "Stok habis" tetap bisa dipilih untuk pesanan di muka.

### Removed
- Kode tidak terpakai: kit `components/ui/` (shadcn), `hooks/use-toast.ts`, `lib/utils.ts`, `components.json`; variabel/state mati di `pos.tsx`, `modal.tsx`, `analytics.tsx`, `dashboard.tsx`; ±1.750 baris CSS (258 aturan, 264 selector, 3 `@keyframes`) untuk class yang tidak dipakai komponen mana pun, termasuk seluruh style `customer-*` lama. Tampilan diverifikasi identik per piksel.

### Changed
- Redesign premium & modern untuk seluruh bagian pelanggan: halaman Pelanggan (hero, KPI, grafik, insight, direktori), pemilih pelanggan di Kasir, form tambah/ubah pelanggan, dan baris pelanggan pada pencarian global. Style baru memakai prefix `cx-*` di akhir `app/globals.css`; logika data tidak berubah.
- Form tambah/ubah pelanggan kini punya pemilih menu makanan & minuman dengan filter kategori, pencarian, dan stepper jumlah. Pilihan otomatis ditulis ke catatan sebagai `Nama menu jumlah` (format yang sudah dibaca pelacak progres pesanan), jadi tidak perlu perubahan database. Helper: `splitNotesIntoMenu` dan `composeNotesFromMenu` di `lib/customers.ts` (dengan tes).

## 0.2.0 — Professional clean-up

### Fixed

- **Stale data after deleting.** Deleting a category left its old name and tab on products until a reload; the same happened for
  customers on orders and products on order items. Local state now mirrors the database's `ON DELETE SET NULL`.
- **Lost stock updates.** A manual stock adjustment could overwrite a sale made on another device (the screen wrote an absolute number
  computed from a stale value). Adjustments are now compare-and-swap: on conflict nothing is written, the number is refreshed and the user is asked to repeat.
  The movement record is no longer able to leave the screen out of sync, and the blocking `alert()` was replaced by the in-app error banner.
- **`supabase/schema.sql` did not create the `notifications` table** that the app uses, so projects set up from the documented file silently lost
  notification persistence. The canonical schema now includes it (`migrations/002_notifications.sql` for existing projects).
- **Duplicate `create_sale` overloads** (4 and 5 arguments) could trigger PostgREST `PGRST203`. Only the 5-argument function remains (`migrations/001_drop_legacy_create_sale.sql`).
- **Writes are explicit.** Category and customer saves no longer spread the whole form state into the database call (previously `id` / `created_at` were sent on edit);
  names are trimmed and required. Product updates are scoped by `owner_id` like every other table.
- **Type errors that were hidden behind `ignoreBuildErrors`**: `runWithAuthRetry` rejected Supabase query builders (they are `PromiseLike`, not `Promise`),
  the notification list type, the `POS` `onDone` signature, a strict-null error on `Map.get`, six callbacks without parameter types,
  and iterating a `Map` with `target: es5` (target is now `ES2017`; Next.js compiles with SWC regardless).
- Dangling reference to a non-existent “PINDAHKAN DATA LAMA” section in the SQL header (replaced by a commented helper), and error hints that still mentioned QRIS.

### Changed

- `app/page.tsx` (one 141 KB file) is split into 20 modules under `components/` and `lib/`; the page itself is an 18-line auth gate.
  Declarations were moved verbatim by a script, not retyped (see Verification).
- `app/globals.css`: removed 613 rules whose selectors can never match (105 classes no component renders) and 2,102 declarations that a later rule for
  the same selector always overrides, and merged duplicate blocks.
  Delivered size: 400 KB → 290 KB minified (−27 %), 63 KB → 49 KB gzip (−22 %); `!important` 4,737 → 3,314.
- Fonts are loaded with `<link>` in `app/layout.tsx` instead of a CSS `@import` (parallel instead of chained requests).
- `next.config.js`: `STRICT_BUILD=1` turns type errors into build failures (CI sets it); security headers in production; `poweredByHeader` off.
  Linting is a separate gate (`npm run lint`).
- Receipt store profile comes from `NEXT_PUBLIC_STORE_*` variables (defaults unchanged).
- Package renamed to `mie-kelar-pos` (was the template name `nextjs`); page metadata, `noindex`, web manifest and app icons added.
- README and docs rewritten to describe what the code actually uses; new `docs/ARCHITECTURE.md`.

### Added

- `npm test` (12 unit tests for the customer-target and formatting logic, no extra packages) and `npm run verify`.
- GitHub Actions CI (Node 20 and 22): typecheck, tests, lint (non-blocking until clean), strict build.
- Accessibility: `prefers-reduced-motion` support, keyboard focus ring for controls without their own focus style, accessible names on row action buttons.
- `.editorconfig`, `.nvmrc`, `.prettierrc.json`, `supabase/README.md`.

### Removed

- Build artifacts and clutter: `tsconfig.tsbuildinfo`, `next.config.js.bak`, `.bolt/`.
- Duplicate SQL: `MIEFLOW-MULTI-USER-FIX.sql` (identical to the old `schema.sql`), `MIEFLOW-SUPABASE-SETUP-FINAL.sql` and `REPAIR-EXISTING.sql`
  (identical to each other and a strict superset of the old `schema.sql`, now merged into `schema.sql`). Older scripts moved to `supabase/legacy/`.

### Verification

How this release was checked (the sandbox it was prepared in had no internet access, so `npm install` and `next build` could not be run there):

- **Render equivalence of the module split:** 40 components and helpers rendered with identical props before and after the split produce identical HTML.
  The one deliberate difference: the chart tooltips (`<title>`) in Overview now use a single string child, because React 19 renders a `<title>` with array children as empty.
- **Stylesheet equivalence:** the computed style of every element (plus `::before` / `::after`) was compared between the old and new stylesheet in
  55 UI states (desktop, tablet, phone, modals, pickers, search, notification panel, dark-theme class): 17,438 element states identical,
  plus 443 hover and keyboard-focus states identical. The only intended differences are the keyboard focus ring and reduced motion.
  A negative control (deliberately damaged stylesheets) was detected, including a one-digit colour change inside a `:hover` rule.
- **Behavior:** 11 end-to-end scenarios against an in-memory backend (delete flows, stock conflict, error display, trimming, payload shape,
  duplicate name, checkout, product edit, login gate). 6 failed on v0.1.0 (the bugs above); all 11 pass on v0.2.0.
- **Types:** strict TypeScript check with stubbed third-party typings: 0 errors. Run `npm run verify` once with dependencies installed
  (CI does) to confirm against the real library typings.
- **Unit tests:** `npm test` passes (12 tests).

### Recommended next steps

Format the sources with Prettier, prune the unused template dependencies, split `dashboard.tsx` into hooks, and upgrade Next.js to a patched release
(details in `docs/ARCHITECTURE.md`).
