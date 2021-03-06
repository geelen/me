---
title: Metaquery and the end of media queries
---

### Ye olde csse

Around about the 50th time you copy paste something like:

	@media only screen and (min-width: 481px) and (max-width: 768px) {
      /* TODO: literally CSS until you die */
    };
    
you might start to think there should be a better way to manage breakpoints. 

<small>In case it's not obvious why copy pasting snippets like the above is a bad idea, you're propagating magic numbers, the chance of introducing errors is high (like two media queries that overlap or, worse, have a gap between them), and if you ever want to change breakpoints or introduce another, you're literally going to die of CSS (not a pretty way to go).</small>

### Enter SASS

You might have been pretty excited to learn of SASS 3.2's [support](http://thesassway.com/intermediate/responsive-web-design-in-sass-using-media-queries-in-sass-32/) for `@content` blocks in mixins, and started doing the following:

```scss
@mixin respond-to($media) {
  @if $media == small {
    @media only screen and (max-width: 480px) { @content; }
  }
  @else if $media == medium {
    @media only screen and (min-width: 481px) and (max-width: 768px) { @content; }
  }
  @else if $media == large {
    @media only screen and (min-width: 769px) { @content; }
  }
}    
```

```scss
.wow-such-responsive {
  @include respond-to(small) {
    /*  still  */
  }
  @include respond-to(medium) {
    /*   not   */
  }
  @include respond-to(large) {
    /*  great  */
  }
}
```
    
That's a big improvement. You're now able to use meaningful words `small` `medium` and `large` rather than magic pixel breakpoints, which massively reduces the chance of errors creeping through.

The output CSS isn't ideal, though, with the lengthy `@media` queries copied verbatim throughout the output. And, given the importance of order of style declarations in CSS, it's [not necessarily possible](https://github.com/css/csso/issues/162) to merge them after generation.

### Javascript wut

You might have tidied up your SASS using a mixin, but you've not made any impact on JS-land. You're back to using strings again:

```js
if (window.matchMedia("only screen and (min-width: 481px) and (max-width: 768px)").matches) {
  /*  yeah  */
} else {
  /*  nah  */
}
```

Media queries in JS aren't _super_ common, I'll grant you, but this still sucks.

---

## Metaquery

[Ben Schwarz](http://germanforblack.com), who I share an office with and who is rad at the web, wrote a tiny library to treat media queries a little differently. The basic premise is to have a series of `meta` tags define your breakpoints that get turned into `breakpoint-X` class on your `html` attribute automatically.

Define these in your `head`:

```html
<meta name="breakpoint" content="small"  
      media="only screen and (max-width: 480px)">
<meta name="breakpoint" content="medium" 
      media="only screen and (min-width: 481px) and (max-width: 768px)">
<meta name="breakpoint" content="large"  
      media="only screen and (min-width: 769px)">
```

And Metaquery will attach classes depending on which media query match:

```html
<html class='breakpoint-small'>
```

Your CSS can then be based directly off the presence or absence of that class, rather than ever worrying about `@media`:

```css
.breakpoint-small  .wow-such-responsive { /*  much  */ }
.breakpoint-medium .wow-such-responsive { /* simple */ }
.breakpoint-large  .wow-such-responsive { /*  wowe  */ }
```

Even better with SASS:

```scss
.wow-such-responsive {
  .breakpoint-small & {
    /* such clear */
  }
  .breakpoint-medium & {
    /*  so reuse  */
  }
  .breakpoint-large {
    /*    wow     */
  }
}
```

And your JS can use it too:

```js
if (document.documentElement.classList.contains('breakpoint-small')) {
  /*  hashtag  */
} else {
  /*  winning  */
}
```

---

## The pro way to use it

#### Default breakpoint for old IE
Metaquery relies on [matchMedia](https://developer.mozilla.org/en-US/docs/Web/API/Window.matchMedia) which IE9 and below [don't have](http://caniuse.com/#search=matchmedia). So add a default desktop breakpoint on the HTML and wrap Metaquery in [conditional IE comments](http://css-tricks.com/how-to-create-an-ie-only-stylesheet/).

There is a [polyfill](https://github.com/paulirish/matchMedia.js/), but I don't recommend it. Older browsers will do just fine with a static experience.

#### Load Metaquery before CSS

Something to watch out for when you have a default breakpoint or no breakpoint set: when the CSS is loaded, the browser will start laying out the screen incorrectly until Metaquery adds the right classes. So you'll probably get an unsightly FOUC (Flash of unstyled content, or in this case incorrectly-styled content).

Additionally, if you're using different background images depending on breakpoint, the browser may download the wrong images first, then have to fetch the right ones. That's bad. So you want to ensure Metaquery is loaded first. How?

#### Inline it!

Metaquery is so small, it's faster to inline it into your HTML than serve it separately. That way it'll be one of the first things to execute, and should prevent any FOUC:

```html
<!doctype html>
<html lang="en" class="breakpoint-medium">
  <head>
    <!-- Normal HEAD stuff here -->

	<!-- Breakpoints -->
    <meta name="breakpoint" content="small" media="(max-width: 480px)">
    <meta name="breakpoint" content="medium" media="(min-width: 481px) and (768px)">
    <meta name="breakpoint" content="large" media="(min-width: 769px)">
    
    <!--[if gt IE 9]><!--><script type="text/javascript" charset="utf-8">
      <!-- INLINE METAQUERY HERE -->  
    </script><!--<![endif]-->
    
    <!-- Then link your stylesheets -->
    <link rel="stylesheet" href="styles/main.css">
  </head>
```

#### Get cracking!
I've got a full demo [Gist](https://gist.github.com/geelen/8858962) showing a real copy-pastable boilerplate for getting going. Start your next HTML file with this and never write a `@media` query again!
