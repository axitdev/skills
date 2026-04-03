# PHP Regex Cheatsheet (PCRE2)

Read this before building, explaining, or debugging any regex pattern.

## Table of Contents

1. [Character Classes](#character-classes)
2. [Anchors](#anchors)
3. [Quantifiers](#quantifiers)
4. [Groups and Backreferences](#groups-and-backreferences)
5. [Lookaround Assertions](#lookaround-assertions)
6. [Modifiers (Flags)](#modifiers-flags)
7. [PHP Functions Quick Reference](#php-functions-quick-reference)
8. [PHP String Escaping Rules](#php-string-escaping-rules)
9. [Common Patterns](#common-patterns) *(see separate file)*
10. [Pitfalls and Gotchas](#pitfalls-and-gotchas)
11. [Performance Tips](#performance-tips)
12. [PCRE2 Advanced Features](#pcre2-advanced-features)

---

## Character Classes

| Pattern | Matches |
|---|---|
| `.` | Any character except newline (unless `s` modifier) |
| `\d` | Digit `[0-9]` |
| `\D` | Non-digit `[^0-9]` |
| `\w` | Word char `[a-zA-Z0-9_]` |
| `\W` | Non-word char `[^a-zA-Z0-9_]` |
| `\s` | Whitespace `[ \t\n\r\f\v]` |
| `\S` | Non-whitespace |
| `\h` | Horizontal whitespace (space, tab) |
| `\v` | Vertical whitespace (newline, carriage return) |
| `\R` | Any Unicode line break sequence |
| `\X` | Any Unicode extended grapheme cluster |
| `\p{L}` | Any Unicode letter (with `u` modifier) |
| `\p{N}` | Any Unicode number (with `u` modifier) |
| `\p{Lu}` | Uppercase Unicode letter |
| `\p{Ll}` | Lowercase Unicode letter |
| `\p{Z}` | Any Unicode separator/space |
| `\p{Cyrillic}` | Cyrillic script characters |
| `\p{Latin}` | Latin script characters |
| `[abc]` | Any of a, b, c |
| `[^abc]` | Not a, b, or c |
| `[a-z]` | Range: a through z |
| `[a-zA-Z]` | Any letter (ASCII only) |
| `[\d\s]` | Digit or whitespace |

**Inside character classes:** `-` must be first, last, or escaped. `]` must be first or
escaped. `^` is negation only when first. `\` always needs escaping.

## Anchors

| Pattern | Matches |
|---|---|
| `^` | Start of string (or line with `m` modifier) |
| `$` | End of string (or line with `m` modifier) |
| `\A` | Start of string (absolute, ignores `m`) |
| `\z` | End of string (absolute, ignores `m`) |
| `\Z` | End of string or before final newline |
| `\b` | Word boundary |
| `\B` | Non-word boundary |
| `\G` | Start of match (after previous match in `preg_match_all`) |

## Quantifiers

| Greedy | Lazy | Possessive | Meaning |
|---|---|---|---|
| `*` | `*?` | `*+` | 0 or more |
| `+` | `+?` | `++` | 1 or more |
| `?` | `??` | `?+` | 0 or 1 |
| `{n}` | `{n}?` | `{n}+` | Exactly n |
| `{n,}` | `{n,}?` | `{n,}+` | n or more |
| `{n,m}` | `{n,m}?` | `{n,m}+` | Between n and m |

**Greedy:** matches as much as possible, backtracks if needed.
**Lazy:** matches as little as possible, expands if needed.
**Possessive:** matches as much as possible, never backtracks (prevents catastrophic backtracking).

## Groups and Backreferences

| Syntax | Purpose |
|---|---|
| `(...)` | Capturing group |
| `(?<name>...)` or `(?P<name>...)` | Named capturing group |
| `(?:...)` | Non-capturing group |
| `(?>...)` | Atomic group (no backtracking inside) |
| `(?|...\|...)` | Branch reset group (groups share numbers) |
| `\1`, `\2` | Backreference by number |
| `\k<name>` or `(?P=name)` | Backreference by name |
| `(?=...)` | Positive lookahead |
| `(?!...)` | Negative lookahead |
| `(?<=...)` | Positive lookbehind (fixed length in PCRE) |
| `(?<!...)` | Negative lookbehind (fixed length in PCRE) |
| `(?(condition)yes\|no)` | Conditional pattern |

**Named groups are strongly preferred** when there are 2+ capture groups. They make PHP code
much more readable: `$matches['year']` vs `$matches[3]`.

## Lookaround Assertions

Lookarounds match a position without consuming characters.

```
Positive lookahead:  foo(?=bar)    — "foo" only if followed by "bar"
Negative lookahead:  foo(?!bar)    — "foo" only if NOT followed by "bar"
Positive lookbehind: (?<=foo)bar   — "bar" only if preceded by "foo"
Negative lookbehind: (?<!foo)bar   — "bar" only if NOT preceded by "foo"
```

**PCRE limitation:** lookbehind must be fixed length. `(?<=a+)` is invalid.
PCRE2 relaxes this slightly — variable-length alternatives are allowed: `(?<=cat|horse)`.

## Modifiers (Flags)

| Modifier | Name | Effect |
|---|---|---|
| `i` | Case-insensitive | `A` matches `a` |
| `m` | Multiline | `^`/`$` match start/end of each line |
| `s` | Single-line (dotall) | `.` matches newline too |
| `x` | Free-spacing | Whitespace ignored, `#` starts comments |
| `u` | UTF-8 mode | Treat pattern and subject as UTF-8 |
| `U` | Ungreedy | Swap greedy/lazy defaults (rarely useful) |
| `J` | Allow duplicate names | Multiple groups can share a name |
| `A` | Anchored | Pattern is anchored at start (like `\A`) |
| `D` | Dollar end only | `$` matches only end of string (not before `\n`) |
| `S` | Study | Extra analysis for optimization |
| `X` | Extra | Error on unrecognized escape sequences |

**Always-recommend:** `u` for any text that might contain non-ASCII characters.

**Free-spacing (`x`) example:**
```
/
  ^                    # start of string
  (?P<year>\d{4})      # year: 4 digits
  -                    # separator
  (?P<month>0[1-9]|1[0-2])  # month: 01-12
  -                    # separator
  (?P<day>[0-2]\d|3[01])    # day: 01-31
  $                    # end of string
/xu
```

## PHP Functions Quick Reference

### preg_match — first match
```php
// Returns 1 if match found, 0 if not, false on error
$result = preg_match('/pattern/u', $subject, $matches);
// $matches[0] = full match, $matches['name'] = named group
```

### preg_match_all — all matches
```php
// Returns count of matches
$count = preg_match_all('/pattern/u', $subject, $matches);
// $matches[0] = array of all full matches
// $matches['name'] = array of all named group matches
```

### preg_replace — replace
```php
// Simple replacement
$result = preg_replace('/pattern/u', 'replacement', $subject);
// With backreferences: $1 or ${1} or $name
$result = preg_replace('/(?P<year>\d{4})-(\d{2})/', '$1/$2', $date);
```

### preg_replace_callback — replace with logic
```php
$result = preg_replace_callback('/pattern/u', function (array $matches): string {
    return strtoupper($matches[0]);
}, $subject);
```

### preg_split — split string
```php
// Split by pattern
$parts = preg_split('/\s*,\s*/u', $csv);
// Keep delimiters: PREG_SPLIT_DELIM_CAPTURE (requires capturing group)
$parts = preg_split('/(\s*,\s*)/u', $csv, -1, PREG_SPLIT_DELIM_CAPTURE);
// Remove empty: PREG_SPLIT_NO_EMPTY
$parts = preg_split('/\s+/u', $text, -1, PREG_SPLIT_NO_EMPTY);
```

### preg_quote — escape for use in pattern
```php
// Escape special characters in a user-provided string
$escaped = preg_quote($userInput, '/');
$pattern = "/{$escaped}/u";
```

### Error handling
```php
$result = preg_match($pattern, $subject);
if ($result === false) {
    $error = preg_last_error();
    $errorMsg = preg_last_error_msg(); // PHP 8.0+
    throw new RuntimeException("Regex error: {$errorMsg}");
}
```

**Error codes:** `PREG_NO_ERROR`, `PREG_INTERNAL_ERROR`, `PREG_BACKTRACK_LIMIT_ERROR`,
`PREG_RECURSION_LIMIT_ERROR`, `PREG_BAD_UTF8_ERROR`, `PREG_BAD_UTF8_OFFSET_ERROR`,
`PREG_JIT_STACKLIMIT_ERROR`.

## PHP String Escaping Rules

This is the #1 source of regex bugs in PHP.

**Single-quoted strings (RECOMMENDED for patterns):**
- Only `\\` and `\'` are escape sequences
- `\d` stays as `\d` — perfect for regex
- `$` stays as `$` — no interpolation
- Rule: **use single quotes for patterns**

**Double-quoted strings (AVOID for patterns):**
- `\n`, `\t`, `\r`, `\$`, `\\`, etc. are interpreted by PHP first
- `$var` is interpolated
- `\d` happens to work (undefined escape = literal), but it's fragile
- `\1` becomes a byte with value 1, NOT a backreference

```php
// GOOD — single quotes
$pattern = '/\d{3}-\d{4}/u';

// BAD — double quotes: \d works by accident, but risky
$pattern = "/\d{3}-\d{4}/u";

// VERY BAD — double quotes with backreference
$pattern = "/(\w+)\1/u";    // \1 becomes chr(1), not a backreference!
$pattern = '/(\w+)\1/u';    // correct
```

## Common Patterns

See `references/common-patterns.md` for a full library of ready-to-use patterns (email, URL,
phone, dates, Polish/Ukrainian IDs, financial formats, crypto addresses, etc.).

## Pitfalls and Gotchas

### 1. Catastrophic backtracking
```
BAD:  /(a+)+b/        — exponential time on "aaaaaac"
GOOD: /(a++)b/         — possessive quantifier, fails fast
GOOD: /(?>a+)b/        — atomic group, same effect
```

### 2. Greedy `.` eating too much
```
BAD:  /<(.+)>/        — on "<a>text<b>" matches "a>text<b"
GOOD: /<(.+?)>/       — lazy: matches "a" then "b" separately
BEST: /<([^>]+)>/     — negated class: faster than lazy
```

### 3. The `$` anchor surprise
```
/^foo$/  matches "foo\n" because $ matches before final newline by default
/^foo\z/ matches only "foo" (use \z for absolute end)
Or use D modifier: /^foo$/D
```

### 4. Unicode without `u` modifier
```
Without u: strlen('café') = 5, \w matches bytes not characters
With u:    mb_strlen('café') = 4, \w matches Unicode word characters
ALWAYS use u for text input.
```

### 5. Delimiter in pattern
```
BAD:  '/path/to/file/'        — delimiter / conflicts
GOOD: '/path\/to\/file/'      — escaped
GOOD: '#path/to/file#'        — different delimiter
GOOD: '~path/to/file~'        — another option
```
Choose a delimiter that doesn't appear in your pattern.

### 6. preg_match returns false on error
```php
// BAD — treats error as "no match"
if (preg_match($pattern, $input, $m)) { ... }

// GOOD — distinguishes error from no-match
$result = preg_match($pattern, $input, $m);
if ($result === false) { /* error */ }
if ($result === 1) { /* match */ }
```

### 7. Named groups in preg_match_all
```php
preg_match_all('/(?P<word>\w+)/u', 'hello world', $m);
// $m['word'] = ['hello', 'world']  — array of all captures
// $m[1] = ['hello', 'world']       — same data, numeric index
```

### 8. preg_replace with $n in replacement
```php
// In replacement strings, $0 is full match, $1 is group 1
// Use ${1} when followed by a digit: ${1}0 means group 1 + literal "0"
// Use single quotes for replacement too to avoid PHP interpolation
```

## Performance Tips

1. **Anchor when possible** — `^` and `$` prevent the engine from trying at every position
2. **Use possessive quantifiers** — `*+`, `++` prevent backtracking into them
3. **Prefer character classes over alternation** — `[abc]` is faster than `a|b|c`
4. **Put most likely alternative first** — `cat|dog` tries `cat` first
5. **Use atomic groups** — `(?>...)` prevents backtracking inside the group
6. **Avoid `.*` at the start** — `.*foo` checks the entire string at each position
7. **Use `\K` instead of lookbehind** — `foo\Kbar` matches `bar` preceded by `foo`, faster
8. **Use `S` modifier** for patterns used many times — tells PCRE to study the pattern
9. **Compile once, use many** — store pattern in a constant or static property
10. **Use `preg_match` with offset** instead of `preg_match_all` + loop when processing matches
    one at a time

## PCRE2 Advanced Features

PHP 7.3+ uses PCRE2. These features are available:

**`\K` — reset match start:**
```
/foo\Kbar/ matches "bar" in "foobar" but only returns "bar" as the match
```

**`(*SKIP)(*FAIL)` — skip matched text:**
```
/"[^"]*"(*SKIP)(*FAIL)|,/  — match commas NOT inside quotes
```

**`(*UTF)` — inline UTF-8 mode:**
```
/(*UTF)pattern/  — same as /pattern/u
```

**`(*BSR_UNICODE)` — Unicode line break for `\R`:**
```
/(*BSR_UNICODE)line\Rbreak/
```

**Conditional patterns:**
```
/(?(DEFINE)(?P<digit>\d))(?P>digit){3}-(?P>digit){4}/
— define reusable subpatterns (like regex functions)
```

**Recursive patterns:**
```
/\((?:[^()]*|(?R))*\)/  — match balanced parentheses
```

**`(*MARK:name)` — debugging aid:**
```
/cat(*MARK:animal)|car(*MARK:vehicle)/
— after match, MARK tells which branch matched
```