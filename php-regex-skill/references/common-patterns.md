# Common Regex Patterns for PHP

Starting points for frequently needed patterns. **Always adapt to your specific requirements** —
these are generic patterns that cover the most common cases but may not match every edge case
in your domain.

Read this file when the user asks for a pattern that matches a common format (email, URL, date,
phone, etc.) — don't reinvent the wheel, start from here and customize.

---

## Validation Patterns

### Email (simple validation, not RFC 5322)
```
/^[\w.+-]+@[\w-]+\.[\w.]+$/u
```
Covers most real-world emails. For RFC-compliant validation, use PHP's `filter_var($email, FILTER_VALIDATE_EMAIL)` instead of regex.

### URL
```
/^https?:\/\/[\w.-]+(?:\.[\w]{2,})(?:[\/\w.-]*)*\/?(?:\?[\w.&=%-]*)?\s*$/u
```
Matches `http://` and `https://` URLs. For more complex URL validation, use `filter_var($url, FILTER_VALIDATE_URL)` or `parse_url()`.

### IPv4 Address
```
/^(?:(?:25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)\.){3}(?:25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)$/
```
Validates each octet is 0-255. For IPv6, use `filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_IPV6)`.

### IPv6 Address (simplified)
```
/^(?:[0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$/
```
Basic full-form only. For compressed forms (`::`), use `filter_var()` instead.

### UUID v4
```
/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
```

### Hex Color
```
/^#(?:[0-9a-fA-F]{3}){1,2}$/
```
Matches `#fff` and `#ffffff`.

### Semantic Version
```
/^(?P<major>0|[1-9]\d*)\.(?P<minor>0|[1-9]\d*)\.(?P<patch>0|[1-9]\d*)(?:-(?P<pre>[\da-zA-Z-]+(?:\.[\da-zA-Z-]+)*))?(?:\+(?P<build>[\da-zA-Z-]+(?:\.[\da-zA-Z-]+)*))?$/
```

---

## Date and Time Patterns

### Date YYYY-MM-DD
```
/^(?P<year>\d{4})-(?P<month>0[1-9]|1[0-2])-(?P<day>0[1-9]|[12]\d|3[01])$/
```
Validates month 01-12 and day 01-31 but does NOT validate month/day combinations (e.g. Feb 30 passes). For strict validation, parse with `DateTimeImmutable::createFromFormat()`.

### Date DD.MM.YYYY (European format)
```
/^(?P<day>0[1-9]|[12]\d|3[01])\.(?P<month>0[1-9]|1[0-2])\.(?P<year>\d{4})$/
```

### Time HH:MM:SS (24-hour)
```
/^(?P<hour>[01]\d|2[0-3]):(?P<min>[0-5]\d):(?P<sec>[0-5]\d)$/
```

### ISO 8601 DateTime
```
/^(?P<year>\d{4})-(?P<month>0[1-9]|1[0-2])-(?P<day>0[1-9]|[12]\d|3[01])T(?P<hour>[01]\d|2[0-3]):(?P<min>[0-5]\d):(?P<sec>[0-5]\d)(?:\.(?P<frac>\d+))?(?:Z|(?P<tz>[+-](?:[01]\d|2[0-3]):[0-5]\d))$/
```

---

## Phone and ID Patterns

### Phone Number (international, loose)
```
/^\+?[\d\s\-().]{7,20}$/
```
Intentionally loose — phone formats vary wildly by country. Tighten for specific markets.

### Polish Phone Number
```
/^(?:\+48\s?)?(?:\d{2}\s?\d{3}\s?\d{2}\s?\d{2}|\d{3}\s?\d{3}\s?\d{3})$/
```
Matches `+48 12 345 67 89`, `123 456 789`, `+48123456789`, etc.

### Ukrainian Phone Number
```
/^(?:\+380\s?)?(?:\d{2}\s?\d{3}\s?\d{2}\s?\d{2})$/
```
Matches `+380 67 123 45 67`, `+380671234567`, etc.

### Polish PESEL
```
/^\d{11}$/
```
Format check only. For full validation (checksum, date extraction), use dedicated logic.

### Polish NIP
```
/^\d{3}-?\d{3}-?\d{2}-?\d{2}$/
```
Matches with or without dashes: `123-456-78-90` and `1234567890`.

### Polish REGON
```
/^(?:\d{9}|\d{14})$/
```

### Polish Postal Code
```
/^\d{2}-\d{3}$/
```
Matches `30-001`, `00-950`, etc.

### Ukrainian IPN (ІПН / tax ID)
```
/^\d{10}$/
```

### IBAN (generic)
```
/^[A-Z]{2}\d{2}[A-Z0-9]{4}\d{7}(?:[A-Z0-9]?){0,16}$/
```
Format check only. Validate checksum separately. For Polish IBAN specifically: 26 characters starting with `PL`.

---

## Text Parsing Patterns

### CSV Field (respecting quotes)
```
/(?:^|,)(?:"([^"]*(?:""[^"]*)*)"|([^,]*))/
```
Handles quoted fields with escaped double-quotes inside. For serious CSV work, use `str_getcsv()` or League\Csv.

### Log Line (common format)
```
/^\[(?P<date>[^\]]+)\]\s+(?P<level>\w+):\s+(?P<message>.*?)(?:\s+(?P<context>\{.*\}|\[.*\]))?\s*$/u
```
Parses lines like `[2024-01-15 10:30:00] ERROR: Something failed {"key":"val"}`.

### PHP Namespace / Class Name
```
/^(?:[A-Z][a-zA-Z0-9]*\\)*[A-Z][a-zA-Z0-9]*$/
```
Matches `App\Http\Controllers\UserController`.

### PHP Use Statement Extraction
```
/^use\s+(?P<fqcn>(?:[A-Z][a-zA-Z0-9]*\\)*[A-Z][a-zA-Z0-9]*)(?:\s+as\s+(?P<alias>[A-Z][a-zA-Z0-9]*))?;/m
```

### Markdown Link
```
/\[(?P<text>[^\]]+)\]\((?P<url>[^)]+)\)/u
```
Matches `[link text](url)`.

### HTML Tag (simple, non-nested)
```
/<(?P<tag>[a-zA-Z][\w-]*)(?:\s[^>]*)?\s*\/?>/
```
For real HTML parsing, **always use a DOM parser** like `DOMDocument` or Symfony DomCrawler.

---

## Financial Patterns

### Currency Amount (with optional decimals)
```
/^(?P<currency>[€$£¥₴₿])?(?P<amount>\d{1,3}(?:[,.\s]\d{3})*(?:[.,]\d{1,2})?)$/u
```
Handles `€1,234.56`, `$1 234,50`, `£100`. Ambiguous formats (is `1,234` one thousand or one-point-two-three-four?) require locale context.

### Credit Card Number (Luhn-eligible format)
```
/^\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}$/
```
Format check only. Validate Luhn checksum separately. Never store raw card numbers.

### Cryptocurrency Address — Bitcoin (P2PKH / P2SH / Bech32)
```
/^(?:(?P<legacy>[13][a-km-zA-HJ-NP-Z1-9]{25,34})|(?P<bech32>bc1[a-zA-HJ-NP-Z0-9]{25,90}))$/
```

### Cryptocurrency Address — Ethereum
```
/^0x[0-9a-fA-F]{40}$/
```

---

## Web and API Patterns

### Slug (URL-safe string)
```
/^[a-z0-9]+(?:-[a-z0-9]+)*$/
```

### JWT Token (format check)
```
/^[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+$/
```

### JSON Key Extraction (simple, non-nested)
```
/"(?P<key>[^"]+)"\s*:/
```
For actual JSON parsing, **always use `json_decode()`**.

### HTTP Header Value
```
/^(?P<name>[A-Za-z][\w-]*):\s*(?P<value>.+)$/m
```

### Query String Parameter
```
/[?&](?P<key>[^=&]+)=(?P<value>[^&]*)/
```
For proper parsing, use `parse_url()` + `parse_str()`.

---

## Tips for Using Common Patterns

1. **Start from a pattern above, then customize** — don't copy blindly. Check if the edge
   cases match your actual data.

2. **When in doubt, use PHP's built-in functions** — `filter_var()`, `parse_url()`,
   `json_decode()`, `DateTimeImmutable::createFromFormat()` are more robust than regex for
   their specific domains.

3. **Test against real data from your project** — search the codebase for existing examples
   of the data format you're matching and use those as test cases.

4. **Consider locale** — date formats, phone numbers, currency, and ID numbers vary by country.
   The patterns here cover common formats but may need adjustment for your market.