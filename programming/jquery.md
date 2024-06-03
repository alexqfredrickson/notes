# jQuery

## Errata

### Enabling single-click navigation for anchor tags on Apple devices
On iPhone 5 (presumably some other Apple devices too), when a user clicks an anchor tag link, instead of redirecting the user to a new page, the iPhone will trigger a hover state, forcing the user to double-click. The following code enables single-click navigation on anchor tag clicks.

```javascript
$("a").on("click touchend", function () {
    var href = $(this).attr('href');
    window.location = href;
});
```