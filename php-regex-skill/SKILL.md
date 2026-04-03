---
name: php-regex
description: >
  Interactive PHP regex builder, debugger, and tester. Use this skill whenever the user needs help
  with regular expressions in PHP. Trigger on phrases like "regex", "regular expression", "pattern
  matching", "preg_match", "preg_replace", "preg_split", or any request involving string pattern
  matching, extraction, validation, or replacement in PHP. Also trigger when the user describes
  what they want to match/extract without using the word "regex" — e.g. "extract all emails from
  this string", "validate phone numbers", "parse this log format", "find all URLs in text",
  "replace all dates with a different format", "split this CSV line respecting quotes". This skill
  handles building patterns from scratch, debugging broken patterns, explaining existing patterns,
  optimizing slow patterns, and generating test cases. Even if the user just pastes a regex and
  asks "what does this do" — use this skill.
---

# PHP Regex Workshop

Interactive regex builder, debugger, and tester for PHP. Builds patterns from natural language
descriptions, explains existing patterns, generates test suites, and catches common pitfalls.

## References

- `references/php-regex-cheatsheet.md` — PHP regex syntax reference, PCRE features, and
  PHP-specific gotchas. **Read this before building or explaining any pattern.**
- `references/common-patterns.md` — Ready-to-use patterns for common formats (email, URL,
  phone, dates, Polish/Ukrainian IDs, financial, crypto, etc.). **Check this before building
  a pattern from scratch** — the pattern you need may already exist here as a starting point.

## Configuration

This skill reads an optional `.regex-workshop.yaml` from the project root. If it doesn't exist,
use defaults — don't ask the user to create one.

**Key config fields and their defaults:**

| Field | Default | Purpose |
|---|---|---|
| `output.patterns_dir` | `./docs/regex` | Where to save pattern documentation |
| `output.index_file` | `./docs/regex/index.md` | Pattern library index |
| `test.framework` | `auto` | Test framework: `auto`, `pest`, `phpunit`, `codeception` |
| `test.output_dir` | `./tests/Unit/Regex` | Where to save generated test files |
| `defaults.delimiter` | `/` | Default regex delimiter |
| `defaults.modifiers` | `u` | Default modifiers (u for UTF-8 is recommended) |
| `defaults.flavor` | `pcre2` | PCRE version hint: `pcre`, `pcre2` |

**Config loading rules:**

1. Read `.regex-workshop.yaml` / `.regex-workshop.yml` from project root
2. If not found, use all defaults
3. Any missing field uses its default
4. If `test.framework` is `auto`, detect from project: check for `pestphp/pest` in
   `composer.json` → Pest, otherwise PHPUnit

## Workflow

The skill operates in different modes depending on what the user needs. Detect the mode from
their request and follow the corresponding flow. At every stage, if you have questions — ask.

### Mode Detection

Read the user's request and determine the mode:

| User intent | Mode |
|---|---|
| "Build me a regex for...", "I need a pattern that matches..." | **Build** |
| "What does this regex do?", "Explain this pattern" | **Explain** |
| "My regex doesn't match...", "Why isn't this working?" | **Debug** |
| "This regex is slow", "Optimize this pattern" | **Optimize** |
| "Generate tests for this regex" | **Test** |
| "Save this pattern", "Add to pattern library" | **Save** |

If the mode is ambiguous, ask the user. Multiple modes can chain — e.g. Build → Test → Save.

---

### Mode: Build

Build a regex from the user's natural language description.

**Step 1: Check common patterns first**

Before building from scratch, read `references/common-patterns.md` — the pattern the user needs
may already exist as a starting point. If you find a match, show it and ask if it fits or needs
customization.

**Step 2: Search the codebase for real examples**

This is what makes regex patterns accurate. Before writing any pattern, search the project for
real data that the pattern will operate on:

- **Find existing regex patterns** — `grep -rn 'preg_match\|preg_replace\|preg_split' --include="*.php"` to see how regex is already used in the project. The user might be duplicating
  or refactoring an existing pattern.
- **Find real sample data** — if the user says "match our order IDs", search for order ID
  usage in code, database seeders, test fixtures, config files, or factory definitions. Real
  data beats hypothetical examples for testing.
- **Check validation rules** — look for existing validation (Laravel rules, Symfony constraints,
  or custom validators) that might already define the format the user wants to match.
- **Find where the pattern will be used** — search for the function/class where the user
  plans to use this regex. Understanding the context (input source, expected volume, error
  handling) helps build a better pattern.

Share what you found with the user before building:

> I searched the codebase and found:
> - 3 existing regex patterns related to order IDs in `app/Services/OrderService.php`
> - Sample order IDs in test fixtures: `ORD-2024-00001`, `ORD-2024-12345`
> - A Laravel validation rule `'order_id' => 'regex:/^ORD-\d{4}-\d{5}$/'` in FormRequest
>
> Should I build on the existing pattern, or do you need something different?

**Step 3: Clarify requirements**

Before writing anything, make sure you understand:
- What exactly should match (and what should NOT match)
- Whether they need full match or partial/search
- Whether they need capture groups (and which parts to capture)
- Input encoding (UTF-8? ASCII only? Mixed?)
- Performance context (running once on a short string, or millions of times in a loop?)

If the user gave clear examples or the codebase search provided enough context, skip questions
you can already answer. Don't over-ask.

**Step 4: Build the pattern incrementally**

Don't dump a complex regex all at once. Build it piece by piece:

1. Start with the simplest pattern that matches the basic case
2. Show it to the user with an explanation of each part
3. Add complexity to handle edge cases
4. Show the updated pattern with explanations of what changed

For each iteration, show:
- The pattern itself
- A breakdown of what each part does
- Which PHP function to use (`preg_match`, `preg_match_all`, `preg_replace`, `preg_split`)
- Example PHP code ready to use

**Step 5: Test against real data**

Use real examples found during codebase search (Step 2) as primary test cases. Supplement with
generated edge cases. Show a test table:

```
Pattern: /your-pattern-here/u

✅ SHOULD MATCH (from codebase):
  "ORD-2024-00001"  → matched (group order_id: "00001")
  "ORD-2024-12345"  → matched (group order_id: "12345")

✅ SHOULD MATCH (generated):
  "ORD-2025-99999"  → matched (boundary: max digits)

❌ SHOULD NOT MATCH:
  "ORD-24-001"      → no match ✓ (wrong format)
  ""                 → no match ✓ (empty string)

⚠️ EDGE CASES:
  "ord-2024-00001"  → no match (case sensitive — is this intended?)
```

If any result is unexpected, iterate on the pattern.

**Step 6: Provide final PHP code**

Give the user a complete, copy-paste-ready code snippet:

```php
/**
 * {Brief description of what this pattern does}
 *
 * Pattern breakdown:
 *   {part1} — {explanation}
 *   {part2} — {explanation}
 */
$pattern = '/your-pattern/u';

// Example usage
if (preg_match($pattern, $input, $matches)) {
    $fullMatch = $matches[0];
    $groupName = $matches['name'] ?? null;
}
```

Always use named capture groups when there are 2+ groups — they make code much more readable
than numeric indexes.

---

### Mode: Explain

Break down an existing regex into human-readable parts.

**Step 1: Parse the pattern**

Take the user's regex and break it into a structured explanation. Use a visual breakdown format:

```
Pattern: /^(?:https?:\/\/)?(?:www\.)?([a-zA-Z0-9-]+(?:\.[a-zA-Z]{2,})+)(?:\/.*)?$/i

 ^                              — start of string
 (?:https?:\/\/)?               — optional "http://" or "https://" (non-capturing)
   https?                       — "http" followed by optional "s"
   :\/\/                        — "://" (escaped slashes)
 (?:www\.)?                     — optional "www." prefix (non-capturing)
 ([a-zA-Z0-9-]+                — CAPTURE GROUP 1: domain name
   (?:\.[a-zA-Z]{2,})+         — one or more TLD segments like ".com", ".co.uk"
 )
 (?:\/.*)?                      — optional path after domain (non-capturing)
 $                              — end of string

Modifiers:
  i — case-insensitive matching

This pattern extracts a domain name from a URL. Group 1 captures the domain without
protocol or path.
```

**Step 2: Identify issues**

After explaining, proactively flag any problems you spot:
- Common pitfalls (see references/php-regex-cheatsheet.md § Pitfalls)
- Missing `u` modifier for text that might contain UTF-8
- Catastrophic backtracking risks
- Overly greedy quantifiers
- Unescaped characters that should be escaped
- Groups that should be non-capturing for performance

**Step 3: Suggest improvements**

If you found issues, offer a corrected version with explanation of what changed and why.

---

### Mode: Debug

Help the user fix a regex that isn't working as expected.

**Step 1: Gather context**

You need:
- The pattern (with delimiters and modifiers)
- The input string that fails
- What they expected to happen vs. what actually happened
- The PHP function they're using
- The PHP error message (if any)

If the user didn't provide all of these, ask for the missing pieces.

Also **search the codebase** to find where this pattern is used — check the surrounding code
for context. Look for:
- What data is being fed into the regex (variable source, user input, file content?)
- Other patterns in the same file that might interact
- Whether the same pattern is duplicated elsewhere (fixing one but not the other causes bugs)

**Step 2: Diagnose**

Common issues to check, in order:

1. **Delimiter conflicts** — is the delimiter present in the pattern without escaping?
2. **Missing modifiers** — `u` for UTF-8, `s` for dotall, `m` for multiline?
3. **Escaping issues** — double escaping in PHP strings (`\\d` vs `\d`)?
4. **Greedy vs lazy** — `.*` eating too much? Need `.*?`?
5. **Anchoring** — missing `^`/`$` or using them in multiline without `m`?
6. **Character class mistakes** — unescaped `-` or `]` inside `[]`?
7. **Backtracking catastrophe** — nested quantifiers like `(a+)+`?
8. **Lookahead/lookbehind issues** — variable-length lookbehind (not supported in PCRE)?
9. **Group numbering** — wrong capture group index?
10. **PHP string interpolation** — using double quotes and `$` being interpreted?

**Step 3: Fix and explain**

Show the original pattern vs. the fixed pattern side by side, with a clear explanation of
what was wrong and why the fix works.

Always provide a test to verify the fix works:
```php
$pattern = '/fixed-pattern/u';
$input = 'the input that was failing';
var_dump(preg_match($pattern, $input, $matches));
var_dump($matches);
```

---

### Mode: Optimize

Improve a regex for performance.

**Step 1: Analyze the pattern and its usage**

Search the codebase to understand how this pattern is used:
- How many times is it called? (once per request vs. in a loop over thousands of records)
- What's the typical input size? (short strings vs. large text files)
- Is the result cached anywhere?

This context determines whether optimization is worth the effort and which optimizations matter.

Identify performance issues:
- Catastrophic backtracking (nested quantifiers, alternation with overlap)
- Unnecessary capture groups (use `(?:...)` instead)
- Overly broad character classes
- Unanchored patterns that force the engine to try every position
- Alternation order (put most likely match first)
- Unnecessary lookahead/lookbehind

**Step 2: Optimize**

Apply optimizations and show before/after:
- Replace `(.*)` with `([^delimiter]*)` where possible
- Use possessive quantifiers (`++`, `*+`) or atomic groups to prevent backtracking
- Use `\K` to avoid lookbehind where applicable
- Anchor patterns when possible
- Use `(*SKIP)(*FAIL)` for complex exclusions
- Consider `preg_match` with offset parameter instead of `preg_match_all` in loops

**Step 3: Benchmark suggestion**

For performance-critical patterns, suggest a quick benchmark:
```php
$input = str_repeat($sampleString, 10000);
$start = hrtime(true);
for ($i = 0; $i < 1000; $i++) {
    preg_match_all($pattern, $input, $matches);
}
$elapsed = (hrtime(true) - $start) / 1e6;
echo "Elapsed: {$elapsed}ms\n";
```

---

### Mode: Test

Generate test cases for a regex pattern.

**Step 1: Analyze the pattern**

Understand what the pattern matches and identify:
- Valid match cases (happy path)
- Boundary cases (minimum/maximum length, edge characters)
- Invalid input cases (should not match)
- Unicode edge cases (if `u` modifier is present)
- Empty string behavior
- Special characters and escaping

**Step 2: Generate test file**

Detect test framework from config (or auto-detect). Generate a complete test file.

**Pest example:**
```php
describe('{PatternDescription}', function () {
    $pattern = '/your-pattern/u';

    it('matches valid {thing}', function () use ($pattern) {
        expect(preg_match($pattern, 'valid input', $m))->toBe(1)
            ->and($m['group_name'])->toBe('expected');
    });

    it('rejects invalid {thing}', function () use ($pattern) {
        expect(preg_match($pattern, 'invalid input'))->toBe(0);
    });
});
```

**PHPUnit example:**
```php
final class {PatternName}RegexTest extends TestCase
{
    private const PATTERN = '/your-pattern/u';

    #[DataProvider('validInputProvider')]
    public function testMatchesValid(string $input, string $expected): void
    {
        $this->assertSame(1, preg_match(self::PATTERN, $input, $m));
        $this->assertSame($expected, $m[0]);
    }

    public static function validInputProvider(): iterable
    {
        yield 'basic case' => ['input', 'expected'];
        yield 'edge case' => ['edge-input', 'expected'];
    }

    #[DataProvider('invalidInputProvider')]
    public function testRejectsInvalid(string $input): void
    {
        $this->assertSame(0, preg_match(self::PATTERN, $input));
    }

    public static function invalidInputProvider(): iterable
    {
        yield 'empty string' => [''];
        yield 'wrong format' => ['bad-input'];
    }
}
```

**Step 3: Save test file**

Save to test output directory from config (default: `./tests/Unit/Regex/`).
File name: `{PatternName}RegexTest.php`.

---

### Mode: Save

Save a pattern to the project's regex pattern library for reuse and documentation.

**Step 1: Gather pattern info**

You need:
- The final pattern (with delimiters and modifiers)
- A short name (e.g. "email-validator", "phone-number-us")
- A description of what it matches
- Example matches and non-matches
- Which PHP function to use it with

**Step 2: Save pattern file**

Save to `{patterns_dir}/{pattern-name}.md` (default: `./docs/regex/{pattern-name}.md`):

```markdown
# {Pattern Name}

> {One-line description}

## Pattern

` ``
/your-pattern-here/u
` ``

## Breakdown

{Visual breakdown of each part, same format as Explain mode}

## Usage

` ``php
{Complete PHP code example}
` ``

## Test Cases

| Input | Expected | Result |
|---|---|---|
| `valid-input` | Match: `"result"` | ✅ |
| `invalid-input` | No match | ✅ |

## Notes

- {Any caveats, limitations, or edge cases}
```

**Step 3: Update index**

Create or update the index file (default: `./docs/regex/index.md`). Build dynamically — only
list patterns that exist.

```markdown
# Regex Pattern Library

> Pattern index. Last updated: {YYYY-MM-DD}

| Pattern | Description | Modifiers |
|---|---|---|
| [{Name}](./{pattern-name}.md) | {short description} | `{modifiers}` |
```

**Rules:**
- Keep entries sorted alphabetically
- Preserve existing entries — only add or update
- Use relative paths from the index file's location

---

## Important Principles

1. **Always explain** — never hand over a regex without explaining what each part does. Regex
   is write-only enough as it is; the explanation is as valuable as the pattern itself.

2. **Test-first mindset** — before declaring a pattern "done", test it against edge cases.
   At minimum: empty string, very long string, unicode characters (if `u` modifier), and
   strings that almost-but-don't-quite match.

3. **Prefer readability** — use `x` modifier (free-spacing) for complex patterns, named capture
   groups for 2+ groups, and comments. A readable regex is a maintainable regex.

4. **PHP-specific awareness** — always account for PHP string escaping. A pattern in a
   single-quoted string needs different escaping than double-quoted. Recommend single quotes
   for patterns to avoid interpolation surprises.

5. **UTF-8 by default** — always include the `u` modifier unless the user explicitly says
   their input is ASCII-only. PHP without `u` treats strings as bytes, which breaks on
   multi-byte characters.

6. **Ask, don't assume** — if the requirements are ambiguous (e.g. "match a phone number"
   without specifying country format), ask before building. There's no universal phone regex.

7. **Know when regex isn't the answer** — if the user is trying to parse HTML, XML, JSON,
   CSV, or email addresses (RFC 5322), recommend a proper parser instead. Regex can validate
   simple formats, but parsing nested structures is a job for a parser. Be direct about this.