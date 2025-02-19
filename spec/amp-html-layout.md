<!---
Copyright 2015 The AMP HTML Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS-IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# AMP HTML Layout System

## Overview

The main goal of the layout system is to ensure that AMP elements can express their layout
so that the runtime is able to infer sizing of elements before any remote resources, such as
JavaScript and data calls, have been completed. This is important since this significantly
reduces rendering and scrolling jank.

With this in mind the AMP Layout System is designed to support few but flexible layout
that provide good performance guarantees. This system relies on a set of attributes such
as `layout`, `width` and `height` to express the element's layout and sizing needs.

## Layout Attributes

### `width` and `height`

Depending on the value of the `layout` attribute AMP component elements must have a `width` and
`height` attribute that contains an integer pixel value. Actual layout behavior is determined by the
`layout` attribute as described below.

In a few cases if `width` or `height` are not specified the AMP runtime can default these values
as following:
- `amp-pixel`: Both `width` and `height` are defaulted to 0.
- `amp-audio`: The default `width` and `height` are inferred from browser.

### `layout`

The optional layout attribute allows specifying how the component behaves in the document layout.
Valid values for the layout attribute are:

- Not present: The `layout` will be inferred as following:
  - if `width` equals to `auto` `fixed-height` layout is assumed;
  - if `width` or `height` attributes are present `fixed` layout is assumed;
  - if `width` and `height` are not present `container` layout is assumed
- `fixed`: The `width` and `height` attributes must be present. The only exceptions are `amp-pixel`
and `amp-audio` elements.
- `fixed-height`: The `height` attribute must be present. The `width` attribute must not be present
or must be equal to `auto`.
- `responsive`: The `width` and `height` attributes must be present and are used to determine the
aspect ratio of the component. The component is sized to the width of its container element while
maintaining the height based on the aspect ratio.
- `fill`: Element size will be determined by the parent element.
- `container`: The component is assumed to not have specific layout itself but only act as a
container. Its children are rendered immediately.
- `nodisplay`: The component takes up zero space on the screen as if its display style was `none`.
The `width` and `height` attributes are not required.

Each element documents which `layout` values it supported. If an element does not support the
specified value it would trigger a runtime error.

### `media`

All AMP custom elements support the `media` attribute. The value of media is a media query. If the query does not match, the element is not rendered at all and it's resources and potentially it's child resources will not be fetched. If the browser window changes size or orientation the media queries are re-evaluated and elements are hidden and shown based on the new results.

Example: Here we have 2 images with mutually exclusive media queries. Depending on the screen width one or the other will be fetched and rendered. Note that the media attribute is available on all custom elements, so it can be used with non-image elements such as ads.

```html
    <amp-img
        media="(min-width: 650px)"
        src="wide.jpg"
        width=466
        height=355 layout="responsive" ></amp-img>
    <amp-img
        media="(max-width: 649px)"
        src="narrow.jpg"
        width=527
        height=193 layout="responsive" ></amp-img>
```

### `placeholder`

The `placeholder` attribute can be set on any HTML element, not just AMP elements. It indicates that
the element marked with this attribute acts as a placeholder for the parent AMP element. If specified
a placeholder element must be a direct child of the AMP element. By default, the placeholder is
immediately shown for the AMP element, even if the AMP element's resources have not been downloaded
or initialized. Once ready the AMP element typically hides its placeholder and shows the content.
The exact behavior w.r.t. to placeholder is up to the element's implementation.

```html
    <amp-anim src="animated.gif" width=466 height=355 layout="responsive" >
      <amp-img placeholder src="preview.png" layout="fill"></amp-img>
    </amp-anim>
```

### `fallback`

The `fallback` attribute can be set on any HTML element, not just AMP elements. It's a convention that
allows the element to communicate to the reader that the browser does not support it. If specified
a fallback element must be a direct child of the AMP element. The exact behavior w.r.t. to fallback
is up to the element's implementation.

```html
    <amp-anim src="animated.gif" width=466 height=355 layout="responsive" >
      <div fallback>Cannot play animated images on this device.</div>
    </amp-anim>
```

### `noloading`

Whether the "loading indicator" should be turned off for this element. Many AMP elements
are whitelisted to show a "loading indicator", which is a basic animation that shows that
the element has not yet fully loaded. The elements can opt out of this behavior by adding
this attribute.


## Behavior

A non-container (`layout != container`) AMP element starts up in the unresolved/unbuilt mode in which
all of its children are hidden except for a placeholder (see `placeholder` attribute). The JavaScript
and data payload necessary to fully construct the element may still be downloading and initializing,
but the AMP runtime already knows how to size and layout the element only relying on CSS classes and
`layout`, `width`, `height` and `media` attributes. In most cases a `placeholder`, if specified, is
sized and positioned to take all of the element's space.

The `placeholder` is hidden as soon as the element is built and its first layout complete. At this
point the element is expected to have all of its children properly built and positioned and ready
to be displayed and accept a reader's input. This is the default behavior. Each element can override
to, e.g., hide `placeholder` faster or keep it around longer.

The element is sized and displayed based on the `layout`, `width`, `height` and `media` attributes
by the runtime. All of the layout rules are implemented via CSS internally. The element is said to
"define size" if its size is inferrable via CSS styles and does not change based on its children:
available immediately or inserted dynamically. This does not mean that this element's size cannot
change. The layout could be fully responsive as is the case with `responsive`, `fixed-height` and
`fill` layouts. It simply means that the size does not change without an explicit user action, e.g.
during rendering or scrolling or post download.

If the element has been configured incorrectly it will not be rendered at all in PROD and in DEV mode
the runtime will display the element in the error state. Possible errors include invalid or unsupported
values of `layout`, `width` and `height` attributes.

## (tl;dr) Appendix 1: Layout Table

What follows is the table of layouts, acceptable configuration parameters and CSS classes and styles
used by this layouts. Notice that:
1. Any CSS class marked prefixed with "-amp-" and elements prefixed with "i-amp-" are considered to be
internal to AMP and their use in user stylesheets is not allowed. They are shown here simply for
informational purposes.
2. The only layouts that currently do not "define size" are `container` and `nodisplay`.
3. Even though `width` and `height` are specified in the table as required the default rules may
apply as is the case with `amp-pixel` and `amp-audio`.

| Layout       | Width/Height Required? | Defines Size? | Additional Elements | CSS "display" |
|--------------|------------------------|---------------|---------------------|---------------|
| nodisplay    | no                     | no | no | `none` |
| fixed        | yes                    | yes, specified by `width` and `height` | no | `inline-block` |
| responsive   | yes                    | yes, based on parent container and aspect ratio of `width:height` | yes, `i-amp-sizer` | `block` |
| fixed-height | `height` only. `width` can be `auto` | yes, specified by the parent container and `height` | no | `block` |
| fill         | no                     | yes, parent's size | no | `block` |
| container    | no                     | no            | no | `block` |
