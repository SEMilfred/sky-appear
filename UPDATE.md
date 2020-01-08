# `lmbo-appear`
Renderless wrapper Vue component as well as Vue mixin (utilized by the wrapper) for exposing style classes and variables to elements based on their position in relation to the viewport.

## Installation
```bash
npm install lmbo-appear
```
or
```bash
yarn add lmbo-appear
```

This package depends on the [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver) API which needs a polyfill for old browsers, including all of IE.  Consider using the [W3C IntersectionObserver polyfill](https://github.com/w3c/IntersectionObserver/tree/master/polyfill).

## Using the wrapper component
Import and install the LmboAppear wrapper component and/or mixin:
```js
import Vue from ‘vue’;
import LmboAppear from ‘lmbo-appear’; // Wrapper component
import { LmboAppearMixin } from ‘lmbo-appear’; // Mixin

export default {
	name: ‘ExampleComponent’,
	mixins: [LmboAppearMixin],
	components: { LmboAppear },
	mounted() {
		console.log(‘Example component loaded’);
	},
};
```

### Basic wrapper example
The wrapper component adds the class `lmbo-appear` to the wrapped element when the dom is loaded. Supplementary classes are then added or removed depending on the element’s relation to the viewport.

* When the element enters the viewport the class `lbmo-appear--in-viewport` is added.
* When the element is above the viewport the class `lbmo-appear--before-viewport` is added.
* When the element is below the viewport the class `lbmo-appear--after-viewport` is added.
```html
<!-- As written in Vue -->
<LmboAppear>
	<div class=“test-element”>
		Show only when entering the viewport
	</div>
</LmboAppear>

<!-- As it may appear in the dom -->
<div class=“test-element lmbo-appear lmbo-appear--after-viewport”>
	Show only when entering the viewport
</div>
```

### Advanced wrapper example
The advanced example below showcases all the functionalities of the wrapper component.
```html
<!-- As written in Vue -->
<div ref=“newRoot”>
	<LmboAppear
		:active=“isActive”
		:default=“’outside'"
		:direction=“‘inline’”
		:enter-once=“false”
		:lazy-load=“true”
		:target=“$refs.newTarget”
		:root=“$refs.newRoot”
		:root-margin=“10% 10%”
		:thresholds=“[0, .25, .5, .75, 1]”
		@enter=“enterHandling”
		@leave=“leaveHandling”
		@update=“updateHandling”
	>
		<div class=“test-element”>
			<p ref=“newTarget”>
				Show only when entering the viewport
			</p>
		</div>
	</LmboAppear>
</div>
```

#### Props overview
| Prop        | Description                                                                                                                                                                                                                                                                                                                | Default value       | Data type       |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|-----------------|
| active      | Whether or not the wrapper is active and detecting.                                                                                                                                                                                                                                                                        | true                | Boolean         |
| init-state  | The initiary state before any detection is done. A value of 'inside' will have the element starting with the `--in-viewport` class modifier, and 'outside' will have it start without it.                                                                                                                                  | 'inside'            | String          |
| direction   | Whether to detect viewport intersection vertically ('block') or horizontally ('inline'). This means when in block-mode the `--before-viewport` class modifier is set when the element is above the viewport, and the `--after-viewport` modifier is when below. When in inline-mode ‘before’ is left and ‘after’ is right. | ‘block’             | String          |
| enter-once  | If set to true, the `--in-viewport` class modifier will be set only once to never be removed again. Internally tracking of intersections will stop when this happens.                                                                                                                                                      | true                | Boolean         |
| lazy-load   | If set to true, the wrapper will replace the content with an empty div having the special class `lmbo-appear--stand-in` until it’s inside the viewport after which it will be replaced by the actual content.                                                                                                              | false               | Boolean         |
| target      | Overrides the default target to instead observe another for changes, for example an element deeper within the structure.                                                                                                                                                                                                   | the slotted element | Object          |
| root        | The root of the intersection observer. [Read more.](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/root)                                                                                                                                                                                            | null / viewport     | Object          |
| root-margin | The root margin of the intersection observer. [Read more.](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/rootMargin)                                                                                                                                                                               | ‘0px 0px 0px 0px’   | String          |
| thresholds  | The thresholds of the intersection observer. [Read more.](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver/thresholds)                                                                                                                                                                                | [0]                 | Array or Number |

@Simon: Considered “delay” prop as well, though a delay can easily be set from the outside using the “active” prop.
“init-state” prop needs some consideration in terms of sites without js enabled.
“lazy-load” cannot be fully done from this wrapper, as proper importing is needed where it is used - though it is a start and helps with nested stuff.

#### Events overview
| Event  | Description                                                                                                                  |
|--------|------------------------------------------------------------------------------------------------------------------------------|
| enter  | Event triggering when the wrapper content enters into the viewport. The element needs only be partially within the viewport. |
| leave  | Event triggering when the wrapper content leaves the viewport. The element needs to leave the viewport fully.                |
| update | Event triggering whenever the viewport changes while the wrapper content is within view.                                     |

Each of the event types pass an event object when called upon:
```js
event = {
	target, // The targeted/observed element (typically the wrapped element)
	insideness, // Value from 0 (fully out of view) to 1 (fully in view)
	outsideBefore, // Boolean value telling whether partially or fully outside of view in the before-direction (left or upwards depending on set direction)
	outsideAfter, // Boolean value telling whether partially or fully outside of view in the after-direction (right or downwards depending on set direction)
}
```

## Using the mixin
The mixin allows for even more control and direct access to the data behind the wrapper. It supplies a series of variables and methods to be used within a component.

@Simon: Unsure if it makes sense to have the mixin as well. Maybe we should have a talk about it - possibly better to just open up for other needed data through the custom events.

```js
export default {
	data() {
		return {
			$lmboAppear: {
				active: true,
				direction: 'block',
				target: this.$el,
				state: 'inside',
				insideness: 1,
				outsideBefore: false,
				outsideAfter: false,
				// Intersection observer options
				root: null,
				rootMargin: '0px 0px 0px 0px',
				thresholds: [0],
			},
		};
	},
	methods: {
	},
};
```

## Sources of inspiration and help
[GitHub - BKWLD/vue-in-viewport-mixin: Vue 2 mixin to determine when a DOM element is visible in the client window](https://github.com/BKWLD/vue-in-viewport-mixin)
[Building renderless components](https://css-tricks.com/building-renderless-vue-components/)
[Renderless Components in Vue.js](https://adamwathan.me/renderless-components-in-vuejs/)
[Skitse for renderless wrapper](https://webcomponents.dev/edit/bnNrwGVY0kLGdTlKwEaz)