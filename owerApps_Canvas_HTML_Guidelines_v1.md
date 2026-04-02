# Power Apps Canvas HTML Development Guidelines


## User Profile & Preferences


### Developer Background
- **Role**: Microsoft Power Platform Developer
- **Specialization**: Power Apps, Power Automate
- **Learning Focus**: AI integration
- **Location**: Vaughan, Ontario, CA


### Standard Variable Naming Conventions
The user consistently uses the following variable naming patterns:
- `nfPrimaryColorText` - Primary brand color
- `nfAccentColorText` - Accent/secondary color
- `nfAppName` - Application name
- `nfAppVersion` - Application version number
- `vSelected[Entity]` - Selected item variables (e.g., `vSelectedOrder`, `vSelectedFormOrder`)
- `col[EntityName]` - Collection variables (e.g., `colNotes`, `colOrders`)
- `v[VariableName]` - General variables (e.g., `vShowSidePaneLeft`)


---


## HTML in Canvas Apps: Limitations & Best Practices


### Key Limitations
1. **No JavaScript execution** - `<script>` tags are completely ignored. Only static HTML and inline CSS supported.
2. **No `<style>` blocks** - The entire `<style>` block is stripped by the PA renderer. All CSS must be inline `style=''` on each element.
3. **No full document tags** - `<html>`, `<head>`, `<body>`, `<meta>`, `<title>` are all stripped. PA only renders the HTML fragment.
4. **No external resources** - Cannot load external CSS files, fonts, or images via URL.
5. **Limited interactivity** - Cannot handle click events or dynamic behaviors. Use native Power Apps controls for buttons and inputs.
6. **Text encoding** - Use `EncodeHTML()` for user-generated content to prevent HTML injection.
7. **Performance** - Complex HTML can impact app performance; keep markup lean.
8. **Quote rules** - Use single quotes `' '` for all HTML attributes outside `{ }`. Inside `{ }` Power Fx expressions, use normal double quotes `" "` â€” no escaping needed. Never use `\"` (backslash escape) or `""` (doubled quotes) inside `{ }` â€” neither pattern works in Power Apps.


### Unsupported CSS Properties
The following CSS properties are silently stripped or broken in the PA HtmlText renderer:
- `position: fixed` and `position: absolute` â€” layout collapses or disappears
- `position: sticky` â€” not supported
- `backdrop-filter: blur()` â€” silently ignored
- `clip-path` â€” not rendered
- `inset: 0` shorthand â€” not supported; use `top/right/bottom/left` explicitly
- `@keyframes` / CSS animations / `animation` / `transition` â€” no animation support
- `min-height: 100vh` â€” expands to full browser viewport, causes scrollbar (see Height Rules below)
- `z-index` â€” ignored since positioning is unsupported
- `transform` â€” not supported
- `filter` â€” not supported
- `grid-template-columns` / CSS Grid â€” limited support; use `<table>` instead
- `gap` on flex containers â€” inconsistent; use `<table>` column spacing instead


### Supported CSS Properties âœ…
The following are confirmed to work reliably:
- All inline `style=''` attributes
- `background-color`, `background` with simple `linear-gradient()` (2â€“3 stops)
- `border`, `border-radius`, `border-left/right/top/bottom`
- `box-shadow`
- `font-family`, `font-size`, `font-weight`, `color`, `letter-spacing`, `text-transform`, `line-height`
- `padding`, `margin`, `width`, `height` (fixed `px` or `%`)
- `display: flex`, `flex-direction: row/column`, `align-items`, `justify-content`, `flex: 1`, `flex-shrink`
- `display: inline-block`
- `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap`
- `box-sizing: border-box`
- `opacity`
- `rgba()` colors
- `vertical-align`, `text-align`
- HTML entities (`&#8594;`, `&middot;`, `&nbsp;`, `&#8226;`)
- Emojis as icon substitutes


### When to Use HTML
- Custom table layouts with precise styling
- Rich text formatting not achievable with native controls
- Complex visual layouts (cards, banners, headers, landing pages)
- Dynamic content rendering with collections


### When NOT to Use HTML
- Interactive buttons or forms (use native Power Apps controls)
- Data entry fields
- Charts or visualizations (use native charting)
- Simple text displays


---


## Height & Scroll Rules


> **Issue: HtmlText control causes vertical scrollbar**


The Power Apps HtmlText control **auto-sizes its height to match the full rendered height of the HTML content**. There is no overflow clipping â€” if the content is taller than the control, a scrollbar appears.


**Causes:**
- `min-height: 100vh` â€” expands to the full browser viewport height, far taller than the PA control
- `padding: 48px` on outer wrapper â€” adds ~96px of extra vertical space
- `<br/>` line breaks inside headings â€” each adds a full line height
- Large `margin-bottom` values stacked across sections (e.g. 32px + 36px + 32px = 100px in gaps alone)
- `height: 100vh` without a matching control height


**Fix:**
- Remove all `min-height`
- Keep outer padding to `16pxâ€“24px` max
- Keep section `margin-bottom` to `8pxâ€“16px` max
- To fill the full control height in Power Fx, use: `height: {Parent.Height}px`
- Use `overflow: hidden` on the outer wrapper to prevent content bleed


---


## Table Structure Standards


### Table Header Format


```powerfx
$"<table style='table-layout: fixed; width: 100%; border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; font-family: Segoe UI, -apple-system, sans-serif; border: 0; box-shadow: 0 2px 8px rgba(0,0,0,0.08);'>
  <colgroup>
    <!-- C1: Column Name --><col style='width: X%'>
    <!-- C2: Column Name --><col style='width: Y%'>
  </colgroup>
  <tr style='border: 0;'>
    <!-- H1 --><th style='background-color: {nfPrimaryColorText}; color: #ffffff; font-weight: 600; border: 0; border-right: 1px solid rgba(255,255,255,0.1); padding: 16px 12px; text-align: center; vertical-align: middle; font-size: 11px; letter-spacing: 0.3px; text-transform: uppercase; word-break: break-word; white-space: nowrap;'>HEADER TEXT</th>
    <!-- H2 --><th style='...'></th>
  </tr>
</table>"
```


**Key Points:**
- Use `<colgroup>` with comments: `<!-- C1: Column Name -->`
- Header comments: `<!-- H1 -->`, `<!-- H2 -->`, etc.
- Background uses `{nfPrimaryColorText}` variable
- Fixed table layout with percentage widths
- Border separators with `rgba(255,255,255,0.1)` between headers


### Table Row Format


```powerfx
$"<table style='...same as header...'>
  <colgroup>
    <!-- Same as header -->
  </colgroup>
  <tr style='cursor: pointer;'>
    <!-- R1 -->
    <td style='padding: 10px 6px; vertical-align: middle; text-align: center; font-size: 11px; color: #64748b; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; position: relative;' title='{ThisItem.FieldName}'>
      {ThisItem.FieldName}
      <div style='position: absolute; right: 0; top: 10%; height: 80%; width: 1px; background: #f3f4f6;'></div>
    </td>
    <!-- R2 -->
    <td style='...'></td>
  </tr>
</table>"
```


**Key Points:**
- Row comments: `<!-- R1 -->`, `<!-- R2 -->`, etc. (sequential numbering)
- Use `title` attribute for tooltip on overflow content
- Vertical divider lines between cells using absolute positioned div
- Use `{ThisItem.FieldName}` for gallery template context
- Only use `EncodeHTML()` for fields that may contain special characters (like Next Step or Notes)


### Selected Row Highlighting


```powerfx
$"<table style='...
  background: {If(ThisItem.IsSelected, "#e0f2fe", "#ffffff")};
  border-left: {If(ThisItem.IsSelected, "4px solid " & nfPrimaryColorText, "4px solid transparent")};
  box-shadow: {If(ThisItem.IsSelected, "0 2px 8px rgba(0,0,0,0.1)", "none")};'>
"
```


**Features:**
- Light blue background (`#e0f2fe`) when selected
- Colored left border using brand color
- Subtle shadow for depth


### Column Width Calculation
1. Sum all column widths before target column = Start %
2. Add half the target column width = Center %
3. Example: For column at position 44%â€“56%, center X = 50%


---


## Common Component Patterns


### 1. Page Header / Nav Bar


```powerfx
$"<div style='font-family: Segoe UI, -apple-system, system-ui, sans-serif; background: {nfPrimaryColorText}; padding: 0; margin: 0; height: 80px; width: 100%; box-shadow: 0 2px 8px rgba(0,0,0,0.08); box-sizing: border-box;'>
  <div style='padding: 24px 48px; display: flex; align-items: center; justify-content: space-between; height: 100%; box-sizing: border-box;'>
    <div style='display: flex; align-items: center; gap: 16px;'>
      <div style='width: 40px; height: 40px;'></div>
      <div>
        <div style='font-size: 22px; font-weight: 700; color: #ffffff; letter-spacing: -0.5px;'>{nfAppName}</div>
        <div style='font-size: 11px; color: rgba(255,255,255,0.65); font-weight: 600; background: rgba(255,255,255,0.1); padding: 2px 8px; border-radius: 4px; display: inline-block; margin-top: 4px;'>{nfAppVersion}</div>
      </div>
    </div>
    <div style='display: flex; align-items: center; gap: 12px;'>
      <div style='width: 36px; height: 36px; background: rgba(255,255,255,0.15); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 600; color: #ffffff; font-size: 14px;'>{Upper(Left(User().FullName, 1))}</div>
    </div>
  </div>
  <div style='height: 3px; background: linear-gradient(90deg, {nfPrimaryColorText} 0%, #0078D4 50%, {nfAccentColorText} 100%); width: 100%;'></div>
</div>"
```


### 2. Empty State Message


```powerfx
$"<div style='display:flex;flex-direction:column;align-items:center;justify-content:center;padding:60px 20px;text-align:center;'>
  <div style='font-size:48px;margin-bottom:16px;opacity:0.3;'>ðŸ“</div>
  <div style='font-size:16px;font-weight:600;color:#666;margin-bottom:8px;'>No Items Found</div>
  <div style='font-size:13px;color:#999;'>Descriptive message about the empty state.</div>
</div>"
```


### 3. Notes / Comments Section with Collection


```powerfx
$"<div style='padding:0;margin:0;'>
  {If(
    IsEmpty(colNotes),
    "<div style='display:flex;flex-direction:column;align-items:center;justify-content:center;padding:60px 20px;text-align:center;'>
      <div style='font-size:48px;margin-bottom:16px;opacity:0.3;'>ðŸ“</div>
      <div style='font-size:16px;font-weight:600;color:#666;margin-bottom:8px;'>No Notes Found</div>
      <div style='font-size:13px;color:#999;'>Be the first to add a note to this item.</div>
    </div>",
    Concat(
      colNotes,
      $"<div style='background-color:#ffffff;border:1px solid #e8e8e8;border-radius:8px;padding:16px;box-shadow:0 1px 3px rgba(0,0,0,0.06);margin:0 0 6px 0;'>
        <div style='display:flex;align-items:center;margin-bottom:10px;'>
          <div style='width:32px;height:32px;border-radius:50%;background-color:{nfPrimaryColorText};display:flex;align-items:center;justify-content:center;color:#ffffff;font-weight:600;font-size:13px;margin-right:10px;'>
            {Upper(Left(userName, 1))}
          </div>
          <div style='flex:1;'>
            <div style='font-size:13px;font-weight:600;color:#333;'>{userName}</div>
            <div style='font-size:11px;color:#999;'>{Text(ts, "mmm dd, yyyy hh:mm tt")}</div>
          </div>
        </div>
        <div style='font-size:13px;color:#555;line-height:1.6;padding-left:42px;'>
          {EncodeHTML(desc)}
        </div>
      </div>"
    )
  )}
</div>"
```


**Key Pattern:**
- Use `If(IsEmpty(collection), emptyStateHTML, Concat(collection, itemHTML))`
- Each note has an avatar circle with user initial
- User info and timestamp in header row
- Content indented to align with avatar


### 4. Side Navigation Toggle


```powerfx
$"<div style='background: transparent; color: {nfPrimaryColorText}; padding: 32px 12px; writing-mode: vertical-rl; text-orientation: mixed; font-size: 14px; font-weight: 700; border-radius: 0 8px 8px 0; letter-spacing: 2px; border-right: 3px solid {nfAccentColorText};'>
  {If(vShowSidePaneLeft, 'âœ• CLOSE VIEW PICKER', 'â˜° MORE VIEWS')}
</div>"
```


### 5. Detail Card Layout


```powerfx
$"<table style='width: 100%; border-collapse: collapse; table-layout: fixed; font-family: Segoe UI, -apple-system, sans-serif; box-shadow: 0 1px 3px rgba(0,0,0,0.08); border-radius: 6px; overflow: hidden;'>
  <colgroup>
    <col style='width:25%;'><col style='width:25%;'><col style='width:25%;'><col style='width:25%;'>
  </colgroup>
  <tbody>
    <tr style='background: #ffffff;'>
      <td style='padding: 8px 12px; font-size: 12px; font-weight: 600; color: #374151; border-bottom: 1px solid #e5e7eb;'>Label</td>
      <td style='padding: 8px 12px; font-size: 12px; color: #6b7280; border-bottom: 1px solid #e5e7eb; border-left: 1px solid #e5e7eb;'>{Value1}</td>
      <td style='padding: 8px 12px; font-size: 12px; font-weight: 600; color: #374151; border-bottom: 1px solid #e5e7eb; border-left: 1px solid #e5e7eb;'>Label</td>
      <td style='padding: 8px 12px; font-size: 12px; color: #6b7280; border-bottom: 1px solid #e5e7eb; border-left: 1px solid #e5e7eb;'>{Value2}</td>
    </tr>
    <tr style='background: rgba(59,130,246,0.08);'>
      <td colspan='4' style='padding: 8px 12px; font-size: 11px; font-weight: 600; color: #1e40af; text-align: center; border-bottom: 1px solid rgba(59,130,246,0.1); text-transform: uppercase; letter-spacing: 0.5px;'>Section Header</td>
    </tr>
  </tbody>
</table>"
```


### 6. Product Pill Badge


```powerfx
$"<div style='display: inline-block; padding: 4px 10px; background-color: rgba(116,39,116,0.10); border: 1px solid rgba(116,39,116,0.40); border-radius: 20px; font-size: 11px; font-weight: 600; color: {nfPrimaryColorText}; letter-spacing: 0.4px; text-transform: uppercase; white-space: nowrap;'>
  <span style='display: inline-block; width: 6px; height: 6px; background-color: {nfPrimaryColorText}; border-radius: 50%; margin-right: 5px; vertical-align: middle;'></span>Label
</div>"
```


### 7. Section Divider (Brand Colour Bar)


```powerfx
"<div style='width: 40px; height: 3px; background-color: {nfPrimaryColorText}; border-radius: 2px; margin-bottom: 16px;'></div>"
```


### 8. Quick Access Card with Left Border


```powerfx
$"<div style='background: #f9fafb; border: 1px solid #e2e8f0; border-radius: 8px; padding: 20px 24px; margin-bottom: 12px; border-left: 4px solid {nfPrimaryColorText};'>
  <div style='font-size: 14px; font-weight: 600; color: #0f172a; margin-bottom: 4px;'>Card Title</div>
  <div style='font-size: 12px; color: #64748b;'>Card description text goes here.</div>
</div>"
```


---


## Color Standards


### Primary Colors
- **Primary**: `{nfPrimaryColorText}` â€” headers, buttons, brand elements
- **Accent**: `{nfAccentColorText}` â€” accents, borders, highlights, TD green


### Power Platform Product Colors
- **Power Apps**: `#742774` (purple)
- **Power Automate**: `#0078D4` (blue)
- **Copilot / AI**: `#54B948` (TD green)
- **Power BI**: `#F2B31B` (gold)


### Neutral Gray Scale
- **Dark Gray**: `#0f172a`, `#1e293b`, `#334155`, `#374151` â€” headings, primary text
- **Medium Gray**: `#475569`, `#64748b`, `#6b7280` â€” secondary text, labels
- **Light Gray**: `#94a3b8`, `#cbd5e1`, `#e2e8f0`, `#e5e7eb` â€” borders, dividers
- **Background Gray**: `#f8fafc`, `#f9fafb`, `#f3f4f6` â€” subtle backgrounds


### Status Colors
- **Red**: `#dc2626` (text), `rgba(239,68,68,0.1)` (background)
- **Amber**: `#d97706` (text), `rgba(245,158,11,0.1)` (background)
- **Green**: `#059669` (text), `rgba(16,185,129,0.1)` (background)
- **Gray/Closed**: `#6b7280` (text), `rgba(107,114,128,0.1)` (background)


### Selection / Highlight
- **Selected Background**: `#e0f2fe` (light blue)
- **Selected Border**: `{nfPrimaryColorText}` with 4px width


---


## Typography Standards


### Font Families
- **Primary**: `Segoe UI, -apple-system, system-ui, sans-serif`
- **Alternative**: `Inter, -apple-system, system-ui, sans-serif` (modern layouts)


### Font Sizes
- **Hero**: 44px
- **Page Titles**: 22â€“28px
- **Section Headers**: 20px
- **Body Text**: 14â€“17px
- **Labels**: 13â€“14px
- **Metadata / Captions**: 11â€“12px
- **Footnotes**: 10px


### Font Weights
- **Bold**: 700 (headings)
- **Semi-bold**: 600 (labels, subheadings)
- **Medium**: 500 (body text)
- **Regular**: 400 (default)


---


## Spacing & Layout


### Padding Standards
- **Containers**: 48px (desktop), 24px (compact), 16px (tight)
- **Cards**: 16px (default), 12px (compact)
- **Table Cells**: 8â€“10px vertical, 6â€“12px horizontal
- **Buttons**: 10â€“16px vertical, 24â€“32px horizontal


### Margins Between Sections
- **Large**: 24â€“32px (between major sections)
- **Medium**: 12â€“16px (between related elements)
- **Small**: 4â€“8px (between inline elements)


### Border Radius
- **Large**: 8px (cards, containers)
- **Medium**: 6px (buttons, inputs)
- **Small**: 4px (badges, tags)
- **Circle**: 50% (avatars)


---


## Common Patterns & Best Practices


### 1. Always Comment Columns and Rows
```powerfx
<!-- C1: Column Name -->   <!-- H1 -->   <!-- R1 -->
```


### 2. Use Consistent Variable Names
Follow `nf` prefix for named formulas, `v` for variables, `col` for collections.


### 3. EncodeHTML Only When Needed
- **Use for**: User-generated content, notes, descriptions, next steps
- **Don't use for**: Static data, numbers, dates, controlled dropdown values


### 4. Responsive Width Strategy
- Use percentage widths in `<table>` columns
- Use `display: flex; flex-direction: column;` for vertical stacks
- Always add `box-sizing: border-box` to include padding in width calculations


### 5. Empty State Pattern
Always provide empty states with:
- Large emoji icon (48px, 30% opacity)
- Bold heading (16px, 600 weight)
- Descriptive text (13px, lighter colour)


### 6. Collection-Based Dynamic Content
```powerfx
{Concat(collection, itemTemplate)}
```


### 7. Conditional Styling
```powerfx
background-color: {If(condition, "#color1", "#color2")}
```


### 8. Multi-Column Layouts
Use `<table>` with `<colgroup>` for all multi-column layouts instead of CSS Grid. Flexbox rows work but avoid `gap` â€” use empty `<td>` spacer cells instead.


### 9. Logo / Image Space
Reserve space for logos using an empty `<div style='width: Xpx; height: Ypx;'>` â€” do not use `<img>` with external URLs, as external resources are blocked.


---


## Testing & Debugging Tips


1. **Test with Empty Data** â€” always test with empty collections and null values
2. **Check Encoding** â€” ensure special characters display correctly
3. **Verify Column Widths** â€” sum of all `<col>` percentages must equal 100%
4. **Check Height** â€” if a scrollbar appears, reduce padding/margins and remove `min-height`
5. **Mobile View** â€” test on different screen sizes
6. **Performance** â€” limit HTML complexity in galleries with many items


---


## Common Issues & Solutions


### Issue: Only plain text visible in Power Apps, no styling
**Cause**: Used `<style>` block or double quotes inside HTML string
**Solution**: Move all CSS to inline `style=''` attributes; use single quotes only; wrap in `$" "`


### Issue: HTML too tall, vertical scrollbar appears
**Cause**: `min-height: 100vh`, large padding/margins, or stacked `<br/>` breaks
**Solution**: Remove `min-height`; reduce outer padding to 16â€“24px; reduce section margins to 8â€“16px; use `height: {Parent.Height}px` in Power Fx


### Issue: HTML too wide, horizontal scroll
**Solution**: Set `width: 100%; box-sizing: border-box;` on outer container


### Issue: Layout breaks / elements disappear
**Cause**: Used `position: fixed`, `position: absolute`, `clip-path`, or `transform`
**Solution**: Replace with table-based layout or normal document flow


### Issue: Animations not working
**Cause**: `@keyframes`, `animation`, `transition` are not supported
**Solution**: Use static styles only; animations are not possible in HtmlText


### Issue: Text overflowing cells
**Solution**: Add `overflow: hidden; text-overflow: ellipsis; white-space: nowrap;`


### Issue: Selected row not obvious
**Solution**: Combine background colour, left border, and box-shadow


### Issue: Special characters breaking layout
**Solution**: Use `EncodeHTML(Text(value))` for user-generated content


### Issue: Inconsistent spacing
**Solution**: Explicitly set all spacing; don't rely on browser defaults


---


## Quick Reference: Most Used Patterns


1. **Table Header** â€” See Table Header Format section
2. **Table Row** â€” See Table Row Format section
3. **Nav Bar / Page Banner** â€” See Page Header pattern
4. **Empty State** â€” See Empty State Message pattern
5. **Notes Section** â€” See Notes/Comments pattern
6. **Selection Highlight** â€” `background: {If(ThisItem.IsSelected, "#e0f2fe", "#ffffff")}`
7. **User Avatar** â€” Circle with initial, 32px, brand colour background
8. **Section Divider** â€” 3px height, brand colour bar, 40px wide
9. **Status Badge** â€” Coloured background with matching text colour, border-radius: 20px
10. **Product Pill** â€” See Product Pill Badge pattern
11. **Quick Access Card** â€” See Quick Access Card pattern


---


## Quote Rules in $" " Interpolated Strings

> **We always wrap HTML in `$" "` â€” this simplifies the quote rules to just 2 scenarios.**

### The Core Rule
Inside `$" "`, anything inside `{ }` is treated as **pure Power Fx**. Never escape quotes inside `{ }`.

### Scenario 1 â€” HTML Attributes (outside `{ }`)
Always use **single quotes** `' '` for all HTML `style`, `class`, `title`, and other attributes:
```powerfx
<div style='font-size:13px;color:#333;'>
<td title='some text'>
```
Single quotes are required because the outer wrapper is already `$" "` double quotes.

### Scenario 2 â€” Power Fx Expressions (inside `{ }`)
Always use **normal double quotes** `" "` inside `{ }` for all string values:
```powerfx
{Text(timestamp, "mmm dd, yyyy hh:mm ss")}
{If(condition, "value1", "value2")}
{EncodeHTML(note)}
```
`{ }` breaks out of the HTML string into Power Fx context â€” standard double-quote syntax applies. No escaping needed.

### âŒ Common Mistake â€” Do NOT Escape Quotes Inside `{ }`

Never use `\"` (backslash escape) or `""` (doubled quotes) inside `{ }` blocks.
Power Apps does not support either escape pattern inside `{ }` interpolated expressions.

```powerfx
// âŒ Wrong â€” backslash escape
{If(condition, \"value1\", \"value2\")}

// âŒ Wrong â€” doubled quotes
{If(condition, ""value1"", ""value2"")}

// âœ… Correct â€” plain single double quotes inside { }
{If(condition, "value1", "value2")}
```

### âš ï¸ Common Trap: Text() Format Strings Inside `{ }`

> **This is the #1 most frequent mistake when generating HTML for Power Apps.**

The `Text()` function format string sits inside `{ }` â€” so it follows the exact same rule: plain single double quotes only. This is the most common place where doubled quotes accidentally appear.

| âŒ Wrong | âœ… Correct |
|---|---|
| `{Text(Now(), ""mmm dd, yyyy"")}` | `{Text(Now(), "mmm dd, yyyy")}` |
| `{Text(Today(), ""yyyy"")}` | `{Text(Today(), "yyyy")}` |
| `{Text(timestamp, ""hh:mm AM/PM"")}` | `{Text(timestamp, "hh:mm AM/PM")}` |
| `{Text(dueDate, ""mmm dd, yyyy"")}` | `{Text(dueDate, "mmm dd, yyyy")}` |

> ðŸš¨ **If you ever see `""` inside `{ }` â€” it is always wrong. Fix it to `"` immediately.**

> **One-line rule**: **HTML attributes** â†’ `' '` single quotes | **Power Fx inside `{ }`** â†’ `" "` double quotes â€” no escaping, no doubling, no backslashes. This applies to ALL Power Fx functions including `Text()`, `If()`, `Concatenate()`, and `Switch()`.

### Quick Reference

| Location | Quote Style | Example |
|---|---|---|
| HTML attributes (outside `{}`) | Single `' '` | `style='font-size:13px;'` |
| Power Fx strings (inside `{}`) | Double `" "` | `{If(done, "Yes", "No")}` |
| Text() format strings (inside `{}`) | Double `" "` | `{Text(Now(), "mmm dd, yyyy")}` |


---


## End of Guidelines


**Last Updated**: March 14, 2026
**Version**: 2.3
