# Bootstrap 3

## Handling interstitial modals containing dynamic &quot;Leaving Site&quot; links with jQuery
Assuming you've got an interstitial modal dialog with a button which confirms whether or not the user would like to leave the site, the following code will open up the modal and dynamically infer which URL the modal should link to:

```("a.interstitial-modal-toggle").on("click", function (e) {
    e.preventDefault();
    e.stopPropagation();
    var href = $(this).attr("href"),
        modal = $(".interstitial-modal"),
        modalExternalLink = modal.find("a.external-link");
    leaveSiteButton.prop("href", href);
    modal.modal("show");
});
```

The following code will link the user offsite, and close the modal dialog:

```(".interstitial-modal a.external-link").on("click", function () {
    var modal = $(".interstitial-modal");
    modal.modal("hide");
});
```

## Removing all padding/margins from Bootstrap grid rows and columns

```.row {
    margin: 0;
}
                                
div[class*='col-'] {
    padding-left: 0;
    padding-right: 0;
}
```

## Creating CSS media queries for Bootstrap
The following code will create media queries which target Bootstrap's `.visible-xs` and `.hidden-xs` elements:

```@media (max-width:767px) {
    // mobile
}

@media (min-width:768px) {
    // desktop
}
```

