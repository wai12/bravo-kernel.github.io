title: How to create responsive vertically aligned multiline radio button labels
tags:
  - css
  - foundation
categories:
  - css
date: 2017-01-28 10:22:56
---

Step-by-step instructions for creating responsive vertical radio buttons with nicely
aligned  multiline labels (using either plain CSS or Foundation 6).

<strong>[Fiddle example](https://jsfiddle.net/bravo_kernel/wuxysegc/)</strong>

{% asset_img preview-vertical-radio-buttons.png 'Preview of vertically aligned radio button labels' %}

## The HTML

```html
  <!-- optionally include the Foundation 6 framework -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/foundation/6.3.0/css/foundation.min.css" />

  <p>What is your favorite Pokemon?</p>
  
  <div class="vertical-radio-buttons">
    <fieldset>
      <form>
        <div>
          <span>
            <input type="radio" name="pokemon" value="Red" id="pokemonRed" required><label for="pokemonRed">Pokemon Red</label>
          </span>
        </div>

        <div>
          <span>
            <input type="radio" name="pokemon" value="Blue" id="pokemonBlue" required><label for="pokemonBlue">Multiline Pokemon Blue. Multiline Pokemon Blue. Multiline Pokemon Blue. Multiline Pokemon Blue. Multiline Pokemon Blue. </label>
          </span>
        </div>

        <div>
          <span>
            <input type="radio" name="pokemon" value="Yellow" id="pokemonYellow" required><label for="pokemonYellow">Pokemon Yellow</label>
          </span>
        </div>

      </form>
    </fieldset>
  </div>
```

## The CSS

```css
.vertical-radio-buttons div {
  display: block;
  padding: 0 0 5px 5px;
  clear: both;
}

.vertical-radio-buttons span {
  display: block;
  padding-left: 20px;
  cursor: inherit;
}

.vertical-radio-buttons label {
  font-size: rem-calc(16);
}

.vertical-radio-buttons input {
  float: left;
  width: 20px;
  margin-left: -20px;
  margin-top: 6px; // vertically align radio button with text
  padding: 0;
  -webkit-appearance: radio;
}
```



