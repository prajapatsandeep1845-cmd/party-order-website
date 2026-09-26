# PartyOrder

A responsive, local-first party jewelry order website. It provides:

- Login and first-time shop setup (shop name and mobile number).
- Four categories: Earings, Tops, Kalanki, and Groom Necklace.
- 100 generated products in each category (400 catalog items total).
- Cart quantity, color, and color-setting fields for every product.
- Automatic 3% GST calculation.
- Browser print invoice layout and saved invoice/customer history.

## Run it

No build tools are required. Open `index.html` in a browser, or serve the folder with any static server:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

## Important production note

This version stores login, shop information, and invoices in the browser's `localStorage`, so it is a complete working prototype but not suitable for shared production use. For production, replace the demo login and local storage with a backend API, password hashing, authorization, and a database. Product data is generated in `app.js`; replace the generator with your real product list and image URLs.
