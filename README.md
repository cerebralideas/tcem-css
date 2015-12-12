# CSS Architecture and Authoring

From an architectural standpoint, CSS authoring needs more rules and conventions in addition to what's defined in most common CSS styleguides. This document will define these rules. If there's a conflict between the general guidelines and here, these rules take precedence.

**Table of contents:**

1. [Section 1: General rules to authoring CSS/Less](#section-1-general-rules-to-authoring-cssless)
2. [Section 2: Class construction with TCEM](#section-2-class-construction-with-tcem)
3. [Section 3: The UI pyramid and scoping](#section-3-the-ui-pyramid-and-scoping)

## Section 1: General rules to authoring CSS/Less
### Rule 1

**Stop thinking in pages and elements.** Think in components, elements and modifiers. Applications are not a series of documents, so don't code according to that old way of thinking. You may have a file named after your "page", what will be called a sub-application from here on out, but all the CSS should not reflect this sub-application.

```css
/* Bad */
.subApp .header {}

/* Good */
.myComponent-header {}
```

### Rule 2

**No more nesting selectors!** Nesting couples our Less/CSS with our markup, a sign of improper UI engineering. Our CSS selectors should be completely independent of how the DOM is marked-up. If you *have* to nest, it can *only* be one level deep and needs to incorporate the direct descendent selector `>`. This could be for state changes, like `.active > .element`, which is a legitimate need.

```css
/* Bad */
.myComponent .header h2 {}

/* Bad */
.myComponent {
	.header {
		h2 {}
	}
}

/* Good */
.myComponent-header_secondary {}
```

Strange class syntax? See [section 2 about class construction](#section-2-class-construction-with-tcem) below for more information.

### Rule 3

**No more mixins.** Mixins create a lot of duplicate CSS properties in outputted CSS, so it’s breaking the DRY principle. Mixins give the illusion of clean code, but the code that actually matters, the outputted CSS, is dirty and smelly.

Rather than mixins, use composition. The class that you would want to use as the mixin, you need to add that to the element, then add an additional class that is responsible for the deltas. This is multi-class composition.

```html
<!-- Bad -->
<div class="myCriticalAlert">Something is wrong.</div>
```
```css
.myCriticalAlert {
	.coreAlert();
	border-color: red;
	background-color: lightred;
}
```
```html
<!-- Good -->
<div class="coreAlert coreAlert_critical">Something is wrong.</div>
```
```css
.coreAlert {
	/* Stuff */
}
.coreAlert_critical {
	border-color: red;
	background-color: lightred;
}
```

### Rule 4

**Don't override framework/library classes!** Don't redeclare a framework or library provided class and modify it directly. This couples component level code with framework level code, breaking our "separation of responsibilities".

Rather, use composition again. Use the framework or library level class on your element, then add a modifying class:

```html
<!-- Bad -->
<div class="col-md-6">I'm special.</div>
```
```css
.col-md-6 {
	padding: 0;
}
```
```html
<!-- Good -->
<div class="col-md-6 column_tightSpacing">I'm special.</div>
```
```css
.column_tightSpacing {
	padding: 0;
}
```

### Rule 5

**The framework and library directories, and everything in them, are to be treated as immutable.** These frameworks live outside of the project and are maintained separately, so any modifications will be overwritten at next update.

### Rule 6

**Follow the principle of least privilege.** In other words, if you need a framework component or library widget, don't throw it in the app's global file, place it in the smallest scope possible. So, if you need the `someWidget` from the Shoestrap framework, place it in *your* component's CSS file.

As this component becomes more and more used, you then increase the level of this widget's scope. Only pulling it to the global CSS file if it's used everywhere.

### Rule 7

**Only use classes to select elements from the DOM for CSS!** IDs are okay for JavaScript selectors. Tag-name selectors are too non-specific and over-reaching for both CSS and JavaScript.

```css
/* Bad */
label {
	padding: 0;
}

/* Good */
.myComponent-label {
	padding: 0;
}
```

## Section 2: Class construction with TCEM

We follow a BEM-like class construction syntax. [Read more about BEM here.](http://bem.info/method/). We have our own flavor of BEM that is called TCEM (pronounced teesim): Type Component Element Modifier.

The intention is to leverage a naming convention that address the above problems: maintainability, scoping, specificity and communication.

```
.{type}_{component}-{element}_{modifier} .is{state}
```

### `{type}` (optional): 

Classes can be used for many things, JS selector, style selector, utility, functional testing … This describes that type. 

Choose one of these options:

- Absence of type is reserved for styling (CSS) selectors
- `js` is reserved for JS selectors
- `test` is reserved for functional test selectors

### `{component}` (required): 

This is also known as “block” in the BEM convention. A component represents a portable collection of elements that make up the intended "thing" you're building. A component may also be made of other smaller components. If this collection of elements or components can reasonably exist outside of the current context, it's a component.

This partial is required as it “namespaces” the class and is the root of this idea. 

Examples of valid names:

- `primaryNav`
- `profilePhoto`
- `contactForm`
- `activityTable`

Avoid generic single word names. Don't use `nav` or `filter` as they are not specific enough. Also, notice how multi-word names are camelCased rather than the common dashed-cased. The dashes and underscores are reserved for partial separators.

### `{element}` (optional): 

This will be very commonly used, but technically isn’t required. It describes the element within the component that you intend to style. Examples:

- `header`
- `container`
- `list`
- `link`
- `footer`

**Avoid the actual tag name of the element**; although there are some exceptions. Don't write `-h4` or `-p`, but `-header`, `-footer` or `-section` should be fine.

### `{modifier}` (optional): 

This should be a lesser used option but is important none-the-less. This describes any modifications to the original element. If in most circumstances you have a particular style, but in this one instance you need it to be different, this is where you apply the modifier. This usually reflects the context of the location. Examples:

- `sidebar`
- `siteFooter`
- `modal`
- `secondary`

We should avoid using page names as modifiers as it's too broad and irrespective of actual usage.

### `{state}` (optional): 

This is used for applying stateful styles to elements. These are always prefixed Examples:

- `isAnimating`
- `isActive`
- `isDisabled`

Notice how in the syntax guide at the start of this section, it's prefixed with `is*`.


## Section 3: The UI pyramid and scoping
We need to start separating our code into layers (visualize a pyramid):

#### 1. Base layer
Think Bootstrap. This is responsible for layout and structure only.

#### 2. Brand layer
This is responsible for the basic "skin" of your brand's aesthetic personality.

#### 3. Application layer
Global code/components specific to this application.

#### 4. Sub-application
If your application is composed of other applications.

#### 5. Component layer
This would be the tip of the pyramid. This is code that is to support the specific components within your section of the app.

### Why layers?
You can view these layers as isolations of intent/responsibilities. Neither layer reaches outside of itself, and that makes them all interchangeable.

Most of our work should reside in layer 4. If global changes are needed, they happen in layer 3. If changes are needed in layer 2 or 1, they happen in their respective repos.

