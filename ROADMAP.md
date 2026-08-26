# Roadmap / notes for this branch (`ppd-invoice-improvements`)

## Already supported, no core change needed

- **Extra total rows** (e.g. "Paid Total", "Total Due" alongside Subtotal/Tax/
  Grand Total): `addTotal($name, $value, $colored = false)` already accepts
  any label/value pair and is called once per row, so callers can already add
  as many rows as they want in whatever order they want. No package change
  was needed for this — it was purely an application-side (`fsuuaas/ppd`)
  usage fix.

## Shipped on this branch

- Fixed a crash (`Undefined array key`) when `setFrom()`/`setTo()` receive a
  `array_filter()`-produced array with gapped keys.
- Fixed a fatal `iconv(): Detected an illegal character in input string` for
  any UTF-8 codepoint with no Windows-1252/ISO-8859-1 equivalent (e.g. `৳`,
  or any non-Latin script) — now degrades to `?` instead of crashing the
  whole PDF.
- Added dedicated `draft` (gray) / `partial` (amber) badge colors instead of
  falling back to the invoice's own brand color for everything except
  `unpaid`/`cancelled`.

## Not done here — needs its own effort

- **True Unicode / non-Latin script rendering** (actually printing `৳`, or
  Bengali/Arabic/CJK text, instead of falling back to `?`): this package
  extends plain `setasign/fpdf`, whose core fonts (Helvetica/Times/Courier)
  are single-byte and Latin-only. Every string in `InvoicePrinter` is piped
  through `iconv()` to Windows-1252/ISO-8859-1 for exactly this reason.
  Real support requires:
  1. Switching to a Unicode-capable engine — either the `tFPDF` extension
     (adds `AddFont(..., true)` + UTF-8 output, minimal API change) or a
     different library entirely (`mPDF`, `dompdf` from HTML/CSS).
  2. Embedding a font that actually contains the needed glyphs (e.g. Noto
     Sans Bengali for `৳`), which increases the generated PDF's file size
     per embedded font subset.
  3. Removing every `iconv()`/`toCharset()` call site and the `mb_strtoupper`
     charset-conversion dance, since a Unicode font takes UTF-8 directly.
  This touches most of `InvoicePrinter.php`'s rendering code, not a single
  method, and needs real visual regression testing (logo/table/badge
  positioning can shift with a different font's metrics) — intentionally
  left as a separate, larger piece of work rather than guessed at here.
