# Building a Bento AMP Component

AMP is extendable, allowing the creation of custom elements and functionality in form of AMP components. For example, AMP provides `amp-accordion`, `amp-base-carousel` and `amp-sidebar` as components, extending AMP.

Bento AMP is a project that allows you to take AMP components and use them in otherwise non-AMP pages. The only requirement of a Bento component is that is be fully functional on non-AMP pages.

Read this document to learn how to create a new Bento AMP component.

<!--
  (Do not remove or edit this comment.)

  This table-of-contents is automatically generated. To generate it, run:
    amp markdown-toc --fix
-->

<!-- {"maxdepth": 2} -->

-   [Getting started](#getting-started)
    -   [Naming](#naming)
    -   [Directory structure](#directory-structure)
    -   [Versioning](#versioning)
    -   [Updating build configs](#updating-build-configs)
-   [Extend AMP.PreactBaseElement](#extend-amppreactbaseelement)
    -   [AMP/Preact Bridge](#amppreact-bridge)
    -   [Actions and events](#actions-and-events)
    -   [PreactBaseElement callbacks](#preactbaseelement-callbacks)
    -   [Element and Component classes](#element-and-component-classes)
    -   [Loading state](#loading-state)
    -   [Element styling](#element-styling)
    -   [Layouts supported in your element](#layouts-supported-in-your-element)
    -   [Register element with AMP](#register-element-with-amp)
-   [Validation, Testing and Experiment](#validation-testing-and-experiment)
    -   [Allowing proper validation](#allowing-proper-validation)
    -   [Performance considerations](#performance-considerations)
    -   [Unit tests](#unit-tests)
    -   [Type checking](#type-checking)
    -   [Experiments](#experiments)
-   [Documentation and Examples](#documentation-and-examples)
    -   [Documenting your extended Bento component](#documenting-your-extended-bento-component)
    -   [Build an example](#build-an-example)
-   [Checklist for PR](#checklist-for-pr)
    -   [Linting and formatting](#linting-and-formatting)
    -   [Update YAML Metadata](#update-yaml-metadata)
    -   [Code Ownership](#code-ownership)
-   [Example PRs](#example-prs)

## Getting started

The first step to creating a new Bento AMP component is familiarizing yourself with [AMP's contributor guidelines](https://go.amp.dev/contribute/code). Adding new components follow the [process for making a significant change](https://go.amp.dev/contribute/code#process-for-significant-changes). This process includes filing an ["Intent to Implement" issue](https://go.amp.dev/i2i) and working with an assigned AMP developer guide before starting significant work.

If the contribution of a Bento component involves porting an existing component to a new version, and is expected to span multiple PRs, it is recommended to track progress with a [Bento tracking issue](https://github.com/ampproject/amphtml/issues/new?assignees=&labels=WG%3A+bento&template=bento-component-tracker.yml&title=%F0%9F%8D%B1+%5Bamp-component-name%3Aversion%5D+Bento+tracking+issue).

### Naming

All Bento AMP component extensions have their tag names prefixed with `amp-`.
Make sure to choose an accurate and clear name for your extension.

Extensions that embed a third-party service must follow the [guidelines for naming a third-party component](../docs/spec/amp-3p-naming.md).

To bootstrap the creation of a new component (or the Bento version of an existing component), the following command will create the directory structure and boilerplate code for you:

```shell
$ amp make-extension --bento --name=amp-my-element
```

This generates CSS used by the AMP element `<amp-my-element>`, and JSS used by the Preact component `<MyElement>`. To exclude either, you may additionally specify `--nocss` and/or `--nojss`.

### Directory structure

Create your extension's files inside the `extensions/` directory.
The directory structure is below:

```sh
/extensions/amp-my-element/
├── 1.0/
|   ├── storybook/                       # Element's manual sample playground (req'd)
|   |   ├── Basic.js                     # Preact usage examples
|   |   ├── Basic.amp.js                 # AMP usage examples
|   ├── test/                            # Element's unit test suite (req'd)
|   |   ├── test-amp-my-element.js
|   |   └── More test JS files (optional)
|   ├── base-element.js                  # Preact base element (req'd)
|   ├── component.js                     # Preact implementation (req'd)
|   ├── component.type.js                # Preact type definitions (req'd)
|   ├── amp-my-element.js                # Element's implementation (req'd)
|   ├── amp-my-element.css               # Custom CSS (optional)
|   ├── component.jss.js                 # Preact JSS (optional)
|   └── More JS files (optional)
├── validator-amp-my-element.protoascii  # Validator rules (req'd)
├── amp-my-element.md                    # Element's main documentation (req'd)
└── More documentation in .md files (optional)
└── OWNERS                               # Owners file. Primary contact(s) for the extension. (req'd)
```

In most cases you'll only create the required (req'd) files. If your element does not need custom CSS or JSS, you don't need to create those files.

### Versioning

AMP runtime is currently in v0 major version. Extensions versions are maintained separately. If your changes to your non-experimental
extension makes breaking changes that are not backward compatible you should release a new version of your extension. This would usually be by creating a [0.2 or 1.0](./spec/amp-versioning-policy.md#amp-extensions) directory next to your 0.1.

If your extension is still in experiments breaking changes usually are fine so you can just update the same version.

Note that Bento upgrades to existing AMP components should go by one major version, meaning it should create a 1.0 directory next to an existing 0.1 or 0.2.

### Updating build configs

You must make changes to [`build-system/compile/bundles.config.extensions.json`](../build-system/compile/bundles.config.extensions.json) for your Bento component to successfully build. You will need to add an entry in the top-level array.

```javascript
exports.extensionBundles = [
...
  {
    "name": "amp-carousel",
    "version": "0.1",
    "options": {
      "hasCss": true
    }
  },
  {
    "name": "amp-kaltura-player",
    "version": "0.1",
  },
...
];
```

The entry for your component must have `options.wrapper = "bento"`. It may optionally include `options.npm = true` to publish the Preact component portion on npm automatically, following AMP's [release schedule](./release-schedule.md).

```diff
exports.extensionBundles = [
...
  {
    "name": "amp-my-element",
    "version": "1.0",
    "options": {
+     "npm": true,
+     "wrapper": "bento"
    }
  },
...
];
```

Note, if you are providing a version upgrade (pre-existing 0.1 to Bento 1.0, for example), it is important to note that the latest version is the 0.1 until the 1.0 version is no longer experimental and fully launched. The following is an example of two version entries for one component:

```diff
exports.extensionBundles = [
...
  {
    "name": "amp-my-element",
    "version": "0.1",
    "options": {
      "hasCss": true
    }
  },
  {
    "name": "amp-my-element",
    "version": "1.0",
    "options": {
      "npm": true,
      "wrapper": "bento"
    }
  },
...
];
```

## Extend AMP.PreactBaseElement

All Preact-based Bento AMP extensions extend `AMP.PreactBaseElement`. Bento AMP extensions differ from classic AMP extensions because they are self-managing and independent, and therefore usable in a wider range of contexts beyond AMP pages, while still being fully integrated with the AMP environment when used in a fully AMP document.

Important: Bento-enabled AMP components built upon a Preact component must also be usable in insolation in React development contexts.

The configurations which bridge the Preact implementation of the component and its custom element counterpart in an HTML or AMP document are explained in the [AMP/Preact Bridge](#amppreact-bridge) section, and the callbacks which handle AMP- and DOM- specific mutability traits are explained in the [PreactBaseElement Callbacks](#preactbaseelement-callbacks) section. All of these are also explained inline in the [PreactBaseElement](https://github.com/ampproject/amphtml/blob/main/src/preact/base-element.js) class.

### AMP/Preact Bridge

#### PreactBaseElement['Component']

-   **Default**: Required.
-   **Override**: Always.
-   **Usage**: Define the corresponding Preact functional component.
-   **Example Usage**: `amp-fit-text`, `amp-timeago`

#### PreactBaseElement['props']

-   **Default**: Optional.
-   **Override**: Almost always.
-   **Usage**: Define the mapping of Preact prop to AmpElement DOM attributes and children. These will update and re-render the component on DOM mutation.
-   **Example Usage**: `amp-base-carousel`, `amp-lightbox`

#### PreactBaseElement['staticProps']

-   **Default**: Optional.
-   **Override**: Almost always.
-   **Usage**: Define the mapping of Preact prop to AmpElement DOM attributes. These will not update or re-render the component on DOM mutation. Can be used in lieu of init().
-   **Example Usage**: n/a

#### PreactBaseElement['layoutSizeDefined']

-   **Default**: Optional.
-   **Override**: Sometimes.
-   **Usage**: Specify that the component requires `layoutSizeDefined`, meaning it can have a layout of any of the following: `fixed`, `fixed-height`, `responsive`, `fill`, `flex-item`, `fluid`, or `intrinsic`. Learn more about how to use this in [Layouts supported in your element](#layouts-supported-in-your-element) or generally with [layouts in AMP](https://amp.dev/documentation/guides-and-tutorials/learn/amp-html-layout).
-   **Example Usage**: n/a

#### PreactBaseElement['loadable']

-   **Default**: Optional.
-   **Override**: Required for any component that makes network requests (directly or via other DOM elements). Optional otherwise.
-   **Usage**: Indicates a resource intensive component to allow granular control of when it should load and unload.
-   **Example Usage**: `amp-instagram`, `amp-video`

#### PreactBaseElement['shadowCss']

-   **Default**: Optional.
-   **Override**: Required CSS when using shadow DOM.
-   **Usage**: Define the CSS for shadow stylesheets.
-   **Example Usage**: `amp-lightbox`, `amp-sidebar`

#### PreactBaseElement['usesShadowDom']

-   **Default**: Optional.
-   **Override**: Rarely.
-   **Usage**: Notify when the element uses the Shadow DOM.
-   **Example Usage**: `amp-social-share`, `amp-youtube`

#### PreactBaseElement['usesTemplate']

-   **Default**: Optional.
-   **Override**: Rarely.
-   **Usage**: Notify when the element uses the template system.
-   **Example Usage**: `amp-date-countdown`, `amp-date-display`

#### Ways to process children (mutually exclusive)

##### PreactBaseElement['lightDomTag']

-   **Default**: Optional.
-   **Override**: Sometimes.
-   **Usage**: The tag name used when rendering into the light DOM. Used when children contents are overwritten.
-   **Example Usage**: `amp-date-countdown`, `amp-date-display`

##### PreactBaseElement['detached']

-   **Default**: Optional.
-   **Override**: Sometimes.
-   **Usage**: Define if user-supplied children must be referenced and coordinated with the internal state of the Preact component, but the component itself should not be visible (in favor of viewing the light DOM children).
-   **Example Usage**: `amp-accordion`, `amp-selector`

### Actions and events

AMP provides a framework for [elements to fire their own events](https://github.com/ampproject/amphtml/blob/main/docs/spec/amp-actions-and-events.md) to allow users of that element to listen and react to the events. For example, the `amp-base-carousel` extension fires a `slideChange` event. This allow publishers to listen to that event and react to it, for example, by updating an `amp-selector` state to match the current slide shown.

The other part of the event-system in AMP is actions. When listening to an event on an element usually you'd like to trigger an action (possibly on other elements). For example, in the example above, the publisher is executing the `toggle` action on `amp-selector`.

The syntax for using this on elements is as follow:

```html
<amp-base-carousel
  on="slideChange:my-selector.toggle(index=e.index, value=true)"
></amp-base-carousel>
```

To fire events on your element use AMP's action service and the
`.trigger` method.

```javascript
actionServiceForDoc(doc.documentElement).trigger(
  this.element_,
  'slideChange',
  createCustomEvent(
    win,
    `amp-base-carousel.${name}`,
    {'index': index}
  ),
  ActionTrust.DEFAULT
);
```

And to expose actions use `registerApiAction` method that your element inherits from `PreactBaseElement`. Note, this should correspond directly with the API exposed in the corresponding Preact compnent via [`useImperativeHandle`](https://reactjs.org/docs/hooks-reference.html#useimperativehandle), and the component should be defined using a [`forwardRef`](https://preactjs.com/guide/v10/switching-to-preact/#forwardref) accordingly.

```javascript
this.registerApiAction('close', (api) => api.close());
```

Your element can choose to override the default `activate` method inherited from BaseElement. For example `amp-lightbox` overrides `activate` to define the `open` default case.

You must document your element's actions and events in its own reference documentation and in [`AMP Actions and Events`](https://github.com/ampproject/amphtml/blob/main/docs/spec/amp-actions-and-events.md).

### PreactBaseElement callbacks

#### checkPropsPostMutations

-   **Default**: Optional.
-   **Override**: Rarely.
-   **Usage**: The callback called immediately after a component's defined props mutate. This can verify if any additional properties need mutations via `mutateProps()` API.
-   **Example Usage**: `amp-date-display`

#### isReady

-   **Default**: Optional.
-   **Override**: Rarely.
-   **Usage**: States whether the element is ready for rendering. Used if there are additional processing requirements needed before rendering the component. This includes providing rendering templates or defining critical asynchronous props.
-   **Example Usage**: `amp-date-countdown`, `amp-date-display`

#### init

-   **Default**: Optional.
-   **Override**: Sometimes, if additional processing is needed in the DOM.
-   **Usage**: If your extension 'props' definition is not sufficient, or when registering AMP actions and events.
-   **Example Usage**: `amp-base-carousel`, `amp-lightbox`

#### handleOnLoading

-   **Default**: Optional.
-   **Override**: Sometimes
-   **Usage**: Method to override to customize the display of the Placeholder/Fallback/Loader
-   **Example Usage**: `amp-render`

#### handleOnLoad

-   **Default**: Optional.
-   **Override**: Sometimes
-   **Usage**: Method to override to customize the display of the Placeholder/Fallback/Loader
-   **Example Usage**: `amp-render`, `amp-iframe`

#### handleOnError

-   **Default**: Optional.
-   **Override**: Sometimes
-   **Usage**: Method to override to customize the display of the Placeholder/Fallback/Loader
-   **Example Usage**: `amp-render`

#### mutationObserverCallback

-   **Default**: Optional.
-   **Override**: Sometimes, if additional processing is needed to make the DOM mutable.
-   **Usage**: A callback called immediately after mutations have been observed on a component. This differs from `checkPropsPostMutations` in that it is called in all cases of mutation, not just the subset of attributes configured in the extension 'props' definition.
-   **Example Usage**: `amp-base-carousel`, `amp-sidebar`

#### updatePropsForRendering

-   **Default**: Optional.
-   **Override**: Sometimes, if additional processing is needed in the DOM.
-   **Usage**: A callback called to compute props before rendering. The properties computed here are ephemeral and thus should not persist via a `mutateProps()` method.
-   **Example Usage**: `amp-timeago`

### Element and Component classes

The following shows the overall structure of your element implementation file (`extensions/amp-my-element/1.0/amp-my-element.js`). See [Experiments](#experiments) to make sure your component is experimentally gated if necessary.

```js
import {AmpPreactBaseElement, setSuperClass} from '#preact/amp-base-element';

import {func1, func2} from '../../../src/module';
import {BaseElement} from './base-element'; // Preact base element.
import {CSS} from '../../../build/amp-my-element-1.0.css';
// more ES2015-style import statements.

/** @const */
const EXPERIMENT = 'amp-my-element';

/** @const */
const TAG = 'amp-my-element';

class AmpMyElement extends setSuperClass(BaseElement, AmpPreactBaseElement) {
  /** @override */
  init() {
    // Perform any additional processing of prop values that are not
    // straightforward attribute passthroughs, as well as AMP-specific
    // work such as registering actions and events.
    this.registerApiAction('close', (api) => api.close());

    const processedProp = parseInt(element.getAttribute('data-binary'), 2);
    return {
      'processedProp': processedProp,
      'onClose': (event) => fireAmpEvent(event)
    };
  }

  /** @override */
  isLayoutSupported(layout) {
    return layout == LAYOUT.FIXED;
  }
}

AMP.extension('amp-my-element', '1.0', (AMP) => {
  AMP.registerElement('amp-my-element', AmpMyElement, CSS);
});
```

The following shows the corresponding overall structure of your Preact base element implementation file (`extensions/amp-my-element/1.0/base-element.js`).

```js
import {MyElement} from './component'; // Preact component.
import {PreactBaseElement} from '#preact/base-element'; // Custom element bridge to Preact.

export class BaseElement extends PreactBaseElement {}

BaseElement['Component'] = MyElement;            // Component definition.

BaseElement['props'] = {                         // Map DOM attributes and children to Preact Component props.
  'button': {selector: 'button', clone: true}    // All <button> children.
  'children': {passthrough: true}                // All remaining children excluding <button>s, which have already been distributed.
  'propName1': {attr: 'attr-name-1'},
  'propName2': {attr: 'attr-name-2', type: 'number'},
};
```

Note: AMP Layout attributes like ‘layout’, ‘height’, ‘heights’, ‘width’, ‘media’, etc are not needed to be mapped as they will be intrinsically passed to the Preact component.

The following shows the corresponding overall structure of your Preact functional component implementation file (`extensions/amp-my-element/1.0/component.js`).

```js
import * as Preact from '#preact';
import {func1, func2} from '../../../src/module';
// more ES2015-style import statements.

export function MyElement({propName1, propName2, ...rest}) {
  // Assign state variables or other behaviors using Preact
  // lifecycle hooks such as useRef, useState, useEffect, etc.
  const [foo, useFoo] = useState(null);

  // Render component via JSX syntax.
  return (
    <div>Hello world</div>
  );
}
```

### Loading state

For components that may load asynchronously, be sure to support `onLoading`, `onLoad`, and `onError` callbacks as props on your Preact component to allow components to fail gracefully.

By default, `PreactBaseElement` provides callbacks which do the following from the AMP layer:

-   `onLoading` toggles on a loader
-   `onLoad` toggles off any loaders, placeholders, or fallbacks
-   `onError` toggles on the fallpack (or, if unavailable, the placeholder)

Your Preact component **must** invoke the `onLoad()` callback once your component has loaded. This is especially important for components that display content dynamically, for example through an iframe or after fetching remote data, because they may take time to load which would affect perceived performance.

Your Preact component _should_ invoke the `onLoading()` and `onError()` callbacks so document authors in any environment can implement graceful behavior when appropriate.

Your AMP extension may customize this behavior by overriding the `handleOnLoading()` / `handleOnLoad()` / `handleOnError()` methods in your AMP extension's class.

### Element styling

You can write a stylesheet to style your element to provide a minimal visual appeal. Your element structure should account for whether you want users (publishers and developers using your element) to customize the default styling you're providing and allow for easy CSS classes and/or well-structure DOM elements.

Element styles load with the element script inside an AMP document. You tell AMP which CSS or JSS belongs to this element when [registering the element](#register-element-with-AMP).

#### Pre-upgrade CSS

Bento components must specify certain layout properties in order to prevent [Cumulative Layout Shift (CLS)](https://web.dev/cls) when in use on non-AMP pages. The following are a standard set of pre-upgrade styles to be used when no AMP runtime or boilerplate--which typically provides this measure of stability for the document author--are not in use:

```css
/* amp-my-element.css */

/*
 * Pre-upgrade:
 * - display:block element
 * - size-defined element
 */
amp-my-element {
  display: block;
  overflow: hidden;
  position: relative;
}

/* Pre-upgrade: size-defining element - hide text. */
amp-my-element:not(.i-amphtml-built) {
  color: transparent !important;
}

/* Pre-upgrade: size-defining element - hide children. */
amp-my-element:not(.i-amphtml-built)
  > :not([placeholder]):not([slot='i-amphtml-svc']) {
  display: none;
  content-visibility: hidden;
}
```

### Layouts supported in your element

AMP defines different layouts that elements can choose whether or not to support. Your element needs to announce which layouts it supports through overriding the `isLayoutSupported(layout)` callback and returning true if the element supports that layout. [Read more about AMP Layout System](https://github.com/ampproject/amphtml/blob/main/docs/spec/amp-html-layout.md) and [Layout Types](https://github.com/ampproject/amphtml/blob/main/docs/spec/amp-html-layout.md#layout).

#### What layout should your element support?

After understanding each layout type, if it makes sense, support all of them. Otherwise choose what makes sense to your element. A popular support choice is to support size-defined layouts (Fixed, Fixed Height, Responsive and Fill) through using the utility `isLayoutSizeDefined` in `layout.js`.

For example, `amp-inline-gallery-thumbnails` only supports fixed-height layout.

```javascript
class AmpPixel extends BaseElement {
  /** @override */
  isLayoutSupported(layout) {
    return layout == Layout.FIXED_HEIGHT;
  }
}
```

While `amp-base-carousel` supports all size-defined layouts.

```javascript
class AmpBaseCarousel extends AMP.BaseElement {
  /** @override */
  isLayoutSupported(layout) {
    return isLayoutSizeDefined(layout);
  }
}
```

Please note that for components that support all size-defined layouts, they also need to override the `layoutSizeDefined` property as follows:

```javascript
/** @override */
AmpBaseCarousel['layoutSizeDefined'] = true;
```

### Register element with AMP

Once you have implemented your AMP element, you need to register it with AMP; all AMP component extensions include the `amp-` prefix.

One way to specify the appropriate styles is to tell AMP which class to use for this tag name and which CSS to load.

```javascript
import {CSS} from '../../../build/amp-my-element-1.0.css';

AMP.extension('amp-my-element', '1.0', (AMP) => {
  AMP.registerElement('amp-my-element', AmpMyElement, CSS);
});
```

Alternatively, you may have defined styles via JSS that compile into CSS and attached via the shadow stylesheet. To tell AMP which class to use, override the `shadowCss` property:

```javascript
import {CSS} from './my-element.jss';

/** @override */
AmpMyElement['shadowCss'] = CSS;
```

## Validation, Testing and Experiment

### Allowing proper validation

One of AMP's features is static validation on AMP document markup. When you implement your element, you must update the [AMP Validator](https://github.com/ampproject/amphtml/blob/main/validator/README.md) to include rules for your element. Otherwise documents using your extended component become invalid. Create your own rules by following the directions at [Contributing Component Validator Rules](https://github.com/ampproject/amphtml/blob/main/docs/component-validator-rules.md).

When converting an existing component to Bento with a version bump, it suffices to copy existing validator tests if applicable for APIs that stay stable between versions. When copying test files, be sure to change the version of the extension being imported.

In simple terms:

1. Modify the spec in the `.protoascii` file, using satisfies and excludes to differentiate restrictiveness between `0.1` and `1.0` versions
2. Copy the existing `validator-amp-my-element.html` and `validator-amp-my-element.out` files from the `0.1/test` directory to the `1.0/test` directory
3. Manually update the script imported from the copied test files to `1.0`

In some cases, the validator rules for a `0.1` counterpart may allow for less strict usage of the component, such as with `requires_usage: EXEMPTED` or `deprecated_allow_duplicates: true`. The `1.0` version should reintroduce these restrictions by differentiating the versions via `satisfies` and `excludes`. [The validation PR for `amp-social-share`](https://github.com/ampproject/amphtml/pull/33478) serves as a useful reference and example for how to execute this change.

For example, changes to the Bento `amp-soundcloud` .protoascii should look as follows:

```diff
+ tags: {  # amp-soundcloud 1.0
+   html_format: AMP
+   tag_name: "SCRIPT"
+   satisfies: "amp-soundcloud 1.0"
+   excludes: "amp-soundcloud 0.1"
+   extension_spec: {
+     name: "amp-soundcloud"
+     version_name: "v1.0"
+     version: "1.0"
+   }
+   attr_lists: "common-extension-attrs"
+ }
tags: {  # amp-soundcloud 0.1 and latest
  html_format: AMP
  tag_name: "SCRIPT"
+   satisfies: "amp-soundcloud 0.1"
+   excludes: "amp-soundcloud 1.0"
  extension_spec: {
    name: "amp-soundcloud"
+     version_name: "v0.1"
    version: "0.1"
    version: "latest"
    requires_usage: EXEMPTED
    deprecated_allow_duplicates: true
  }
  attr_lists: "common-extension-attrs"
}
```

### Performance considerations

#### Loading external resources

You may need to add third party integration for extended components that need to load external resources, such as an SDK. Loading resources is only allowed inside a third party iframe. AMP serves this on a different domain for security and performance reasons. Take a look at adding [`amp-facebook`](https://github.com/ampproject/amphtml/pull/1479/) extension PR for examples of third party integration, and pair it with the [Bento contribution for `amp-facebook`](https://github.com/ampproject/amphtml/pull/34585/) to see how their impementations may differ.

Read about [Inclusion of third party software, embeds and services into AMP](https://github.com/ampproject/amphtml/blob/main/3p/README.md).

For contrast, take a look at `amp-instagram` which does NOT require an SDK to embed a post. Instead, it provides an iframe-based embedding that allows `amp-instagram` to use a normal iframe with no third party integrations. `amp-youtube` and others work similarly.

### Unit tests

Make sure you write good coverage for your code. We require unit tests for all checked in code. We use the following frameworks for testing:

-   [Mocha](https://mochajs.org/), our test framework
-   [Karma](https://karma-runner.github.io/), our tests runner
-   [Sinon](http://sinonjs.org/), spies, stubs and mocks.
-   [Enzyme](https://enzymejs.github.io/enzyme/), our Preact test utility.

For faster testing during development, consider using --files argument to only run your extensions' tests.

```shell
$ amp unit --files=extensions/amp-my-element/1.0/test/test-amp-my-element.js --watch
```

Please also reference [Testing in AMP HTML](https://github.com/ampproject/amphtml/blob/main/docs/testing.md) for the full range of testing commands available.

### Type checking

We use Closure Compiler to perform type checking. Please see [Annotating JavaScript for the Closure Compiler](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler) and existing AMP code for examples of how to add type annotations to your code.

The following shows the overall structure of your type definition file (extensions/amp-my-element/1.0/my-element.type.js). This will allow support of inline prop destructuring in Preact components.

```javascript
/** @externs */

/**
 * @typedef {{
 *   propName1: (string|undefined),
 *   propName2: (number|undefined),
 *   children: (?PreactDef.Renderable|undefined),
 * }}
 */
var MyElementProps;
```

Run the following command to ensure no type violations are introduced by your extension.

```shell
$ amp check-types
```

### Experiments

Most newly created elements are initially launched as [experiments](https://github.com/ampproject/amphtml/blob/main/tools/experiments/README.md). This allows people to experiment with using the new element and provide the author(s) with feedback. It also provides the AMP Team with the opportunity to monitor for any potential errors. This is especially required if the validator isn't updated with your extended component rules. Without your rules in the validator, documents using the component in production are invalid.

Add your extension as an experiment in the `amphtml/tools/experiments` file by adding a record for your extension in EXPERIMENTS variable.

```javascript
/** @const {!Array<!ExperimentDef>} */
const EXPERIMENTS = [
  // ...
  {
    id: 'bento-my-element',
    name: 'AMP My Element',
    spec:
      'https://github.com/ampproject/amphtml/blob/main/extensions/' +
      'amp-my-element/amp-my-element.md',
    cleanupIssue: 'https://github.com/ampproject/amphtml/issues/XXXYYY',
  },
  // ...
];
```

Then protect your code with a check for the component-specific flag `isExperimentOn(win, 'bento-my-element')`, or the global flag which enables all Bento components `isExperimentOn(win, 'bento')`, and only execute your code when it is on.

```javascript
import {CSS} from '../../../build/amp-my-element-1.0.css';
import {isExperimentOn} from '#experiments';
import {userAssert} from '../../../src/log';

/** @const */
const TAG = 'amp-my-element';

class AmpMyElement extends AMP.PreactBaseElement {
  /** @override */
  isLayoutSupported(layout) {
    userAssert(
      isExperimentOn(this.win, 'bento') ||
        isExperimentOn(this.win, 'bento-my-element'),
      'expected global "bento" or specific "bento-my-element" experiment to be enabled'
    );
    return layout == LAYOUT.FIXED;
  }
}

AMP.extension('amp-my-element', '1.0', AMP => {
  AMP.registerElement('amp-my-element', AmpMyElement, CSS);
});
```

#### Enabling and removing your experiment

Users wanting to experiment with your element can then go to the [experiments page](https://cdn.ampproject.org/experiments.html) and enable your experiment.

If you are testing on your localhost, use the command `AMP.toggleExperiment(id, true/false)` to enable the experiment. One option is to use `AMP.toggleExperiment('bento', true)` to enable all Bento components at once.

File a GitHub issue to cleanup your experiment. Assign it to yourself as a reminder to remove your experiment and code checks. Removal of your experiment happens after the extension has been thoroughly tested and all issues have been addressed.

## Documentation and Examples

### Documenting your extended Bento component

Create a `.md` file that serves as the main documentation for your element. This document should include:

-   Summary table
-   Overview
-   How to use it including code snippets and images
-   Examples
-   Attributes to specify (optional and required)
-   Validation

For samples of element documentation, see: [`amp-accordion`](https://github.com/ampproject/amphtml/blob/main/extensions/amp-accordion/amp-accordion.md), [`amp-instagram`](https://github.com/ampproject/amphtml/blob/main/extensions/amp-instagram/amp-instagram.md), [`amp-stream-gallery`](https://github.com/ampproject/amphtml/blob/main/extensions/amp-stream-gallery/amp-stream-gallery.md)

You must add a migration notes section for upgrades of existing AMP extended component. It should detail any differences between the new Bento and prior versions. For an example, reference the issue filed for [`amp-fit-text`](https://github.com/ampproject/amphtml/issues/28281).

### Build an example

Providing an example of your extended Bento component helps users understand how it works. It provides an easy starting point for developers to begin experimenting. This is where you actually build AMP HTML and non-AMP HTML documents and demonstrate element use. Create the file in the local `storybook/` directory, usually with the `Basic.amp.js` filename. There should also be a file which uses the component in a Preact environment with the Basic.js file name if applicable. Browse `storybook/` directories in other extended component directories to see examples. See how to view these samples on a local server in [Using Storybook](#using-storybook).

Also consider contributing an example to
[amp.dev](https://amp.dev/) on
[GitHub](https://github.com/ampproject/amp.dev).

#### Using Storybook

To speed up development and testing of components, we recommend using the Storybook dashboard and develop "stories" (testing scenarios) for components. [Storybook](https://storybook.js.org/) has many features that assist with the isolated development and manual testing of components including easy manual interaction with component parameters, integrated accessibility auditing, and responsiveness testing.

Two storybook enviroments are available in your component directory `amp-my-element/1.0/storybook` :

|            | AMP Storybook             | Preact Storybook      |
| ---------- | ------------------------- | --------------------- |
| File       | `\storybook\Basic.amp.js` | `\storybook\Basic.js` |
| Accessible | `localhost:9001`          | `localhost:9002`      |

To interact with the storybook component, [Knobs](https://github.com/storybookjs/addon-knobs) are available. [Color](https://github.com/storybookjs/addon-knobs#color), [text](https://github.com/storybookjs/addon-knobs#text), [number](https://github.com/storybookjs/addon-knobs#number), [select](https://github.com/storybookjs/addon-knobs#select), [boolean](https://github.com/storybookjs/addon-knobs#boolean) are commonly used knobs.

To run these environments and explore existing components, run:

```
amp storybook
```

## Checklist for PR

### Linting and formatting

It is recommended to check for errors with lint and prettify code. Run following commands to see and resolve any errors reported by them otherwise CircleCI check will not be passed.

Run the following command to validates JS files against the ESLint linter.

```shell
$ amp lint --local_changes
```

Run the following command to validate non-JS files using Prettier.

```shell
$ amp prettify --local_changes
```

Use `--fix` with above commands to automatially fix common warning and error.

### Update YAML Metadata

Currently, Bento is in the experiment stage. So it is advised to include the following YAML metadata in the markdown:

```
bento: true
experimental: true
```

For example, Bento `amp-youtube` markdown should have YAML metadata as:

```diff
---
$category@: media
+ bento: true
+ experimental: true
formats:
  - websites
teaser:
  text: Displays a YouTube video.
---
```

### Code Ownership

Developers are requested to provide code ownership for newly created components. Please read [Code Ownership and AMP](https://github.com/ampproject/amphtml/blob/main/docs/code-ownership.md) for detailed information. Refer discussion related to code ownership on PR [#34948-discussion_r656779590](https://github.com/ampproject/amphtml/pull/34948#discussion_r656779590).

## Example PRs

-   Adding general UI components
    -   [amp-accordion](https://github.com/ampproject/amphtml/pull/30446)
    -   [amp-sidebar](https://github.com/ampproject/amphtml/pull/31593)
    -   [amp-timeago](https://github.com/ampproject/amphtml/pull/26507)
    -   [amp-base-carousel](https://github.com/ampproject/amphtml/pull/29303)
    -   [amp-stream-gallery](https://github.com/ampproject/amphtml/pull/30597)
-   Adding video components which use `VideoWrapper`. [You may also follow the guide to Building a Bento Video Component.](./building-a-bento-video-player.md)
    -   [amp-video](https://github.com/ampproject/amphtml/pull/30280)
    -   [amp-vimeo](https://github.com/ampproject/amphtml/pull/33971)
    -   [amp-youtube](https://github.com/ampproject/amphtml/pull/30444)
-   Adding iframe based embeds [You may also follow the guide to Building a Bento Video Component.](./building-a-bento-iframe-component.md)
    -   [amp-instagram](https://github.com/ampproject/amphtml/pull/30230)
-   Adding third party iframe based embeds
    -   [amp-facebook](https://github.com/ampproject/amphtml/pull/34585)
    -   [amp-twitter](https://github.com/ampproject/amphtml/pull/33335)
