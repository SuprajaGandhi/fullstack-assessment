# StackShop - Bug Fixes & Improvements Documentation

## Overview

This document details all bugs identified and fixed in the StackShop eCommerce application.

---

## 1. Runtime Error: Cannot Read Properties of Undefined

### What was the issue?
The application crashed with a runtime error: "Cannot read properties of undefined (reading '0')" when trying to access `product.imageUrls[0]`. This occurred because some products in the dataset had `imageUrls` as `undefined` rather than an empty array.

### How it was fixed
Changed all image access from `product.imageUrls[0]` to `product.imageUrls?.[0]` using optional chaining. Added null checks before rendering image galleries: `product.imageUrls && product.imageUrls.length > 1`. Applied the same fix to both the product listing page and product detail page.

### Why this approach?

Optional chaining is the standard for safely accessing nested properties. It gracefully handles undefined values by returning undefined instead of throwing an error, preventing the entire application from crashing.
---

## 2. Security Vulnerability: URL Query Parameter Injection

### What was the issue?
The product detail page passed the entire product object through the URL query string using `JSON.stringify(product)`. This created extremely long, unreadable URLs and exposed several security and usability issues:
- Entire data structure visible in URL
- Users could manipulate product data (price, title, etc.) by editing the URL
- URL length limits could break functionality

### How it was fixed
Created a new API endpoint at `/api/products/[sku]/route.ts` to fetch products by SKU. 
Moved product page from `app/product/page.tsx` to `app/product/[sku]/page.tsx` for dynamic routing.

### Why this approach?
Using the SKU as a URL parameter is the standard REST API pattern. It provides clean URLs, prevents data tampering, and follows security best practices by keeping sensitive data on the server.

---

## 3. Missing Image Hostname Configuration

### What was the issue?
The application crashed with an error when trying to load images from `images-na.ssl-images-amazon.com` because this hostname wasn't configured in Next.js image optimization settings. Next.js requires all external image domains to be explicitly whitelisted for security.

### How it was fixed
Added the missing hostname to the `remotePatterns` array in `next.config.ts` alongside the existing `m.media-amazon.com` hostname.

### Why this approach?
Next.js requires explicit configuration of external image domains to prevent security vulnerabilities and ensure proper image optimization.

---

## 4. Improved Search Performance

### What was the issue?
The search functionality triggered an API call on every single keystroke, causing excessive network requests and poor performance. For example, typing "kindle" would make 6 separate API calls (k, ki, kin, kind, kindl, kindle).

### How it was fixed
Implemented debounce logic with a 500ms delay. Created a separate `debouncedSearch` state that updates only after the user stops typing. The API call now uses the debounced value instead of the raw search input.

### Why this approach?
Debouncing is the standard solution for search inputs. It significantly reduces server load and improves user experience by waiting for the user to finish typing before making the API call. 

---

## 5. Missing Pagination

### What was the issue?
The application only displayed the first 20 products with no way to view additional products. The "Showing X products" text was misleading as it showed the loaded count, not the total available.

### How it was fixed
Added pagination controls with Previous and Next buttons.
Show current page number and total pages.

### Why this approach?
Offset-based pagination is simple to implement with the existing API structure and provides a familiar user experience. 

---

## 6. Missing Price Display

### What was the issue?
Products displayed without any pricing information, which is essential for an eCommerce application. The `retailPrice` field existed in the data but wasn't being shown to users.

### How it was fixed
Added `retailPrice` to the Product TypeScript interface. Display price prominently on product cards with proper formatting. 

### Why this approach?
Price is critical information for eCommerce. Displaying it prominently on both listing and detail pages helps the end user.

---

## 7. Improved Accessibility (Lighthouse Report)

### What was the issue?
Interactive elements lacked proper accessibility attributes, making the application difficult to use with screen readers and keyboard navigation. Lighthouse accessibility score was low due to missing labels.

### How it was fixed
Added comprehensive aria-labels to all interactive elements for screen readers. 

### Why this approach?
Proper accessibility is both a legal requirement and improves usability for all users. ARIA labels provide context for screen readers without changing the visual design.

---

## 8. Image Performance Issues (Lighthouse Report)

### What was the issue?
All product images loaded with the same priority, causing slower initial page load and poor Largest Contentful Paint (LCP) scores. Images below the fold were loading unnecessarily early.

### How it was fixed
Added priority loading for the first 4 images (above the fold). Implemented lazy loading for remaining images. Configured modern image formats such as WebP in Next.js config. Optimized device sizes for responsive images. Added proper sizes attribute for responsive loading.

### Why this approach?
Prioritizing above-the-fold images improves perceived performance and LCP scores. Modern image formats provide better compression without quality loss.

### Additional improvements
Improved Lighthouse performance score and reduced initial page load time.

---

## 9. Missing Error Handling

### What was the issue?
All API fetch calls lacked error handling. Network failures or API errors would fail silently or crash the application without informing the user.

### How it was fixed
Added try-catch blocks to all API routes. Added .catch() handlers to all fetch calls in components. Return appropriate HTTP status codes (404, 500) from API routes.

### Why this approach?
Proper error handling is essential, Users need to know when something goes wrong. 

### Additional improvements
The application now gracefully handles network failures and API errors instead of crashing or failing silently.

