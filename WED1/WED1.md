---
typora-copy-images-to: ./Bilder
---

# Markup-Sprachen

![6C37E024-D241-4725-A785-8C6D9CAFA1B9](Bilder/6C37E024-D241-4725-A785-8C6D9CAFA1B9.png)

Achtung: das Leerzeichen nach dem Book gehört nicht zum Markup, es ist ein Trennzeichen (genau so wie das Leerzeichen zwischen der Menge und dem Preis nicht zum Markup gehört).

Markup gibt dem Text Semantik. Sie sind deklarativ, und beschreiben eine Bedeutung und nicht (direkt) eine Darstellung. Wenn man semantisches Markup verwendet, können auch Blinde die Semantik der Elemente verstehen.

## Encoding

Wenn möglich: UTF-8. TODO: nochmal genau nachschlagen, das kam irgendwie nicht so ganz raus wies funktioniert...

# HTML Dokument

Standard HTML-Dokumentstart

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>First document</title>
    <meta charset="utf-8" />
  </head>
  <body>
    
  </body>
</html>
```

Das DOM davon (mit H1 und P im Body):

![60A7BA1E-D73A-4DEC-B5B6-AFFAE96978CC](Bilder/60A7BA1E-D73A-4DEC-B5B6-AFFAE96978CC.png)

## Elemente

Die meisten Elemente benötigen einen closing-Tag. void-Elemente dürfen nicht geschlossen werden, sondern müssen offen gelassen werden, oder mit XML End Mark geschlossen werden (``/>``)

## Attribute

* lowercase (Konvention)
* Manche Attribute sind obligatorisch

```html
<img src="./hsr-logo.png" alt ="HSR Logo">
<a href="http://www.hsr.ch">HSR</a>
```

* Manche Attribute erlauben nur vordefinierte Werte

```html
<input type="text" /> <!-- "number", "email", "date", "password", ... -->
```

* Manche Attribute brauchen keine Werte

```html
<video src="video.mp4" autoplay controls></video>
```

* ``data-*``-Attribute sind custom

```html
<img data-size="mobile" src="images/logo.png" />
```

## Semantik

HTML sollte die Semantik und nicht die visuelle Erscheinung beschreiben

```html
<!-- gut -->
<p class="warning">Warning</p>
<!-- schlecht -->
<p class="red">Warning</p>
```