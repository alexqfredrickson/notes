# HTML

## Errata

### Initial scaling for iPad devices
This HTML will fix some rendering issues on iPad by initially zooming out the page a little bit.

```html
<meta name="viewport" content="width=device-width, initial-scale=0.8">
```

### Removing links from phone numbers on Apple devices
Apple devices automatically style and underline phone numbers it locates within HTML documents.  This code will prevent this from happening without having to use any hacks.

```html
<meta name="format-detection" content="telephone=no">
```