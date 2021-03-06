# Fluid Typography

This Fluid Typography starter kit quickly sets you up to create dynamically-resizing, responsive websites (using the vw, or viewport width, CSS unit).

## Getting Started

* Add the _fluid-type.scss SASS file to your project.
* Prior to using _fluid-type.scss mixins and function, set layout widths in your _variables.scss if different from the defaults.

````
// IMPORTANT!
// Define these from largest to smallest. If you change the order, some of the mix-ins will break.
$layouts: (
  'desktop-large': 1660px,
  'desktop': 1440px,
  'desktop-small': 1024px,
  'tablet': 800px,
  'small-tablet': 640px,
  'mobile': 375px
);

// Account for difference between 100vw and 100% when scroll bars are included on page.
// Center and trims the offset caused in most browsers by scroll bars when true
$body-width-fix: true;

// set to true to limit maximum scale up for larger screens (default false)
$use-maximum-scaling-width: true;
$maximum-scaling-width: 1600px;

// set the base font size in pixels that will be used for non-screen media (default 10px)
// in general, make this the same value in px that you intend to use as your root font size for rem based sizing.

i.e. body { font-size: 1.6rem; } // 16px for print style
$print-base-font-size: 10px;
````

## Usage

### Quick Setup

#### Proportionally Perfect

For each layout, to make rem values that scale perfectly to match your layout.

```
// in your scss
// rem values scale 1:1 with screen size to be proportional to layout
html {
  @include vw-base;
}
```

#### Limited Up-scale

For each layout, make rem values proportionally perfect at the ideal width, and then
scale gradually until the next layout.

```
// in your scss
// rem values are proportional to layout at minimum (at the layout width)
// and scale up slowly to 120% maximum before next layout
html {
  @include base-font-locks;
}
```

### Pass Layouts to Javascript Environment

Use the bp-to-json mixin to package the layouts map into your body:before psuedo element as parsable JSON.

In your scss:

```
body {
  @include bp-to-json;
}
```

In your javascript:

```
// Get a list of layouts that came from css
const layouts = JSON.parse(JSON.parse(window.getComputedStyle(document.body, ':before').getPropertyValue('content')));

// Now you can run js media queries on this
let isMobile = window.matchMedia(`(max-width: ${layouts.tablet - 1}px)`).matches;
```

## TLDR;

There are a couple different ways you can use VWs in your project.

### Option 1 (proportionally perfect):

 Apply the vw-base mixin to the root <html> element. Then, in your CSS, set your style measurements (font-size, padding, margin, width, height, position, etc.) for all other elements using the px2rem() function. Simply measure your dimensions and values in pixels in Photoshop/Sketch, and enter the pixel value into the px2rem() function. Your value will be converted to REMs, which are similar to EMs except that they are relative to the root element and will not be affected by the parent element's font-size. Viewed at any viewport width, your page will always look perfectly proportional to the creative design.

The only down-side to this approach is that text will get comically large when growing to the next layout change if you have few layouts. Also, if your design
doesn't account for smaller screens, the text will be unreadably small on displays smaller than your ideal layout width. It's best to plan for several layout that grow from the smallest ideal viewport width (which should match your design layout) up to the next larger layout design. On the other hand, this might be extremely good behavior if your want the text to be readable on a large screen from a distance.

Also keep in mind that some full-bleed elements (e.g. a full-bleed hero region where the design has text placed centered and near the bottom of the hero) may grow vertically beyond the bottom of wider (but shorter) screens than the design planned for. You may need to use max-height properties on some elements or aspect ratio media queries in some of these cases to adjust these.

````
html {
  // root font-sizes, in vw's
  @include vw-base;
}

body {
  // the true default font-size, in rem's (should be 16px when viewed at exact layout width, will be larger than 16 for
  // screens larger than the layout width, but proportional to the width)
  font-size: px2rem(16);
}

h1 {
  font-size: px2rem(48); // 48 pixels when viewed at ideal layout width
  margin: px2rem(30) 0;
}
````

### Option 2: ("font locks")

Apply the base-font-locks mix-in to your html element, and use the px2rem() function for sizes as you would for option 1. This approach has the benefits of Option 1 when viewing your design at the ideal layout width, but will allow limited up-scale as the viewport becomes larger. This will limit up-scale to prevent type from becoming excessively large and begin to look "horsey" (a term I hear creatives use sometimes). By default, it will allow up to 120% growth (say 12px type on the design will grow to as much as 14.4px for large enough screens.) Use the base-font-locks mix-in to set this growth factor to your particular needs.

The down-side to this approach is that the layout won't be proportionally "perfect" to the design at all intermediate viewport widths, but everything will look nice, and you still won't need to layer in a million media queries to fix breaking text everywhere if you plan to let your design grow into additional screen real-estate. With both option 1 and option 2, it is best to plan for growth, not shrink. If you try this method and have your layout scale down/shink (NOT recommended), you'll lose some of the efficient CSS benefits of the fluid type approach.

````
html {
  // root font-sizes, in calculated scale
  // scale root font size from 10px proportional to layout up to 12px
  // proportional to layout
  // use this *instead* of vw-base
  @include base-font-locks(10, 12);
}

body {
  // the true default font-size, in rem's (should be 16px when viewed at exact layout width, will be larger than 16 for
  // screens larger than the layout width, no more than 120% size)
  font-size: px2rem(16); // from 16px physical up to 19.2px before breaking to next layout
}

h1 {
  font-size: px2rem(48); // 48 pixels when viewed at ideal layout width
  margin: px2rem(30) 0;
}
````

### Option 3: (individual component control):

To affect individual components only, apply the vw-base mixin to the component container element. Then, use ems and the px2em() function to resize all component elements. This is a little more complicated because EMs are relative to the font-size of the context in which they are used, so you will have to track the current sizing context each time you set the font-size.

````
header {
	@include vw-base;

	h1 {
		font-size: px2em(48);
		margin: px2em(30, 48) 0;
	}

	h2 {
		font-size: px2em(36);
	}
}
````

### Setting minimum and maximum font sizes

To avoid text that gets too small or expands too large, you can set min and max font sizes using the font-size-breakpoint() function.

Example:

````
h2 {
	font-size: px2em(16);

	// when the text scales down to 12px, set a fixed font-size of 12px on desktop
	@media screen and (max-width: font-size-breakpoint(12px, $desktop-layout, 16px)) and (min-width: $desktop-min) {
		font-size: 12px;
	}

	// when the text scales up to 20px, set a fixed font-size of 20px on tablet
	@media screen and (max-width: $tablet-max) and (min-width: font-size-breakpoint(20px, $tablet-layout, 16px)) {
		font-size: 20px;
	}
}
````

### VW Quirks & Fixes

#### Body Width Bug Fix

In many browsers, 100vw is not equivalent to 100% on the <html> element--the scroll bars are taken into account for % but not for vws. The relatively easy fix below forces <html> to be 100vw wide and is included, by default, in the vw-base mixin. If 100vw is larger than 100% it applies a negative margin to center the <html> element. The only caveat is that a little bit of your page will be trimmed on the sides.

````
html {
	@media screen {
		width: 100vw;
		margin-left: calc((100% - 100vw) / 2);
		overflow-x: hidden;
	}
}
````

In your scss variables file, you an make this adjustment automatically by setting $body-width-fix to true. Set this to false if you are ok with the layouts being slightly truncated by the right-side scroll-bar (i.e. if this is already accounted for in the design with left/right margins for instance). Warning: If your design is edge to edge and there is visible content touching the edge, using this approach may cut off your content slightly.

To adjust for scroll bars.
```
$body-width-fix: true;
```

#### Print Stylesheets

VWs are interpreted as 0px when you print which renders elements invisible. To avoid undesirable styling, keep all vw styles within @media screen blocks.

Style for desktop first, then use @media **screen** to override your styles for tablet and mobile.

### Browser Compatibility
IE9+, Firefox, Chrome, Safari, iOS, Android 4.4+
