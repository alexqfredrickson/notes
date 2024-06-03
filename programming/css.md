# CSS

## Errata

### Media queries for detecting Apple devices
via http://stackoverflow.com/a/12848217/3474146

```css
/* iPhone < 5: */
@media screen and (device-aspect-ratio: 2/3) {
}
/* iPhone 5: */
@media screen and (device-aspect-ratio: 40/71) {
}
/* iPhone 6: */
@media screen and (device-aspect-ratio: 375/667) {
}
/* iPhone 6 Plus: */
@media screen and (device-aspect-ratio: 16/9) {
}
/* iPad: */
@media screen and (device-aspect-ratio: 3/4) {
}
```

