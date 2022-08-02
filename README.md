# Container Query Polyfill

A small (9 kB compressed) polyfill for CSS Container Queries using [`ResizeObserver`](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) and [`MutationObserver`](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) supporting the full [`@container`](https://drafts.csswg.org/css-contain-3/) query syntax:

- Discrete queries (`width: 300` and `min-width: 300px`)
- Range queries (`200px < width < 400px` and `width < 400px`)
- Container relative length units (`cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, and `cqmax`) in properties and keyframes

## Getting Started

To use the polyfill, add this script tag to the head of your document: :

```js
<script type="module">
  if (!("container" in document.documentElement.style)) {
    import("https://unpkg.com/container-query-polyfill/cqfill.modern.js");
  }
</script>
```

You may also wish to use a service to conditionally deliver the polyfill based on `User-Agent`, or self-host it on your own origin.

> **Note**
> All browsers have support for container queries released or on their roadmap, so it's recommended that you avoid bundling the polyfill with your other code.

For the best user experience, it's recommended that you initially only use the polyfill for content below-the-fold and use `@supports` queries to temporarily replace it with a loading indicator until the polyfill is ready to display it:

```css
@supports not (container-type: inline-size) {
  .container,
  footer {
    display: none;
  }

  .loader {
    display: flex;
  }
}
```

You can view a more complete demo [here](https://codesandbox.io/s/smoosh-glitter-m2ub4w?file=/index.html). On sufficiently fast networks and devices, or devices that natively support Container Queries, this loading indicator will never be displayed.

> **Note**
> Keep in mind that this technique effectively trades off LCP for less jank during initial load, so you may see regressions in the former as a result, particularly on low end devices.

## Limitations

- **CSS first**: The polyfill currently only supports `<style>` and `<link>` elements. Inline styles via the `style` attribute or CSSOM methods are not polyfilled. Likewise, JavaScript APIs like `CSSContainerRule` are not polyfilled, and APIs like `CSS.supports()` are not monkey-patched.
- **Best effort**: Style changes that do not lead to observable DOM or layout mutations (e.g. `font-size` in a container without content) may not be detected, or may be detected a frame late on some browsers.
- Currently, there is no support for Shadow DOM, or functions like `calc(...)` in container conditions. Your contribution would be welcome!

## Supporting browsers without `:where()`

The polyfill uses the CSS [`:where()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:where) pseudo-class to avoid changing the specificity of your rules. This pseudo-class is relatively new, however. If you need to support browsers without it, you will need to append the dummy `:not(container-query-polyfill)` pseudo-class to the originating element of every selector under a `@container` block:

<table>
<tr>
<td> Before </td> <td> After </td>
</tr>
<tr>
<td>

```css
@container (min-width: 200px) {
  #foo {
    /* ... */
  }

  .bar {
    /* ... */
  }

  #foo,
  .bar {
    /* ... */
  }

  ul > li {
    /* ... */
  }

  ::before {
    /* ... */
  }
}
```

</td>
<td>

```css
@container (min-width: 200px) {
  #foo:not(.container-query-polyfill) {
    /* ... */
  }

  .bar:not(.container-query-polyfill) {
    /* ... */
  }

  #foo:not(.container-query-polyfill),
  .bar:not(.container-query-polyfill) {
    /* ... */
  }

  ul > li:not(.container-query-polyfill) {
    /* ... */
  }

  :not(.container-query-polyfill)::before {
    /* ... */
  }
}
```

</td>
</tr>
</table>

This is to ensure the specificity of your rules never changes (e.g. while the polyfill is loading, or on browsers with native support for container queries). On browsers without `:where()` supports, rules without the dummy will be ignored.

## ResizeObserver Loop Errors

When using the polyfill, you may observe reports of errors like `ResizeObserver loop completed with undelivered notifications` or `ResizeObserver loop limit exceeded`. These are expected, and may safely be ignored.
