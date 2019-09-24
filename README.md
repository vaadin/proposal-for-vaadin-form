# Vaadin Form: Proposal
We are planning to build a small and fast framework-agnostic library for creating forms with Web Components. Strong typing using TypeScript. Lots of examples to use good UX patterns.

![code-sample](https://user-images.githubusercontent.com/22416150/65491510-d70f5b00-deb7-11e9-8231-aa8df2929a95.png)

## Vaadin Form goals
_Do you need a form in your app? Not sure how to do it best? Vaadin Form docs explain what makes a good form UX, and how to implement it using the `vaadin-form` web component so that the code is clean, bug-free and easy to maintain._

Vaadin Form _should_:
 - explain how to create great forms with Web Components
 - give ready-to-use examples for common use cases
 - make typical tasks like validation, submission handling, and form state management much easier to implement than with plain JavaScript
 - support TypeScript to allow static type-checking
 - work smoothly with any standard Web Components
 - work out-of-the-box with Vaadin components
 - be small and fast with little run-time overhead

## Status
We are validating the idea. There is no implementation yet.
Your feedback is welcome! Please [create issues](https://github.com/vaadin/proposal-for-vaadin-form/issues/new) in this repo.

## Overview and motivation
Forms are overwhelmingly common on the Web and even more so in line-of-business apps. Yet, it's surprisingly difficult to make forms with a good UX. And for complex forms, the DX often suffers as well.

Vaadin helps developers build web apps that users love. Excellent support for making forms has been a strong side of Vaadin, and we want that to remain as the front-end technology used in Vaadin evolves. When building UIs with Web Components on the client-side, developers would be looking to answer the same questions about making forms:
 - how to show backend data in a form
 - how to submit a form to the backend
 - how to validate a form (per field / as a whole)
 - how to show that the form does not pass validation
 - how to run validations asynchronously on the backend
 - how to render parts of a form conditionally
 - how to create multi-page forms (aka wizards)
 - how to reset form to its initial state

There are many good (and bad) examples out there for developers building apps with React, Angular and Vue. But there are not so many examples that are not tied to a framework. At Vaadin we see a lack of resources focusing on building forms with Web Components, e.g. with [Vaadin](https://vaadin.com/framework) or on the [`open-wc`](https://open-wc.org/) tech stack using libraries like [`lit-element`](https://www.npmjs.com/package/lit-element) or [`haunted`](https://www.npmjs.com/package/haunted). A form component is currently missing from the Vaadin components set, and there has been an [open request](https://github.com/vaadin/vaadin-core/issues/207) for it for a while.

The Vaadin Form library would become such a resource: helping developers create forms that users love, out of standard Web Components.

## Code samples
Some people say a code snippet is worth a thousand words.

The specific APIs used in the code snippets below are a very early sketch and may very well change in the final product. The snippets are here to illustrate the key concepts and use cases the library may support.

Listing features separately allows discussing them separately and eventually prioritizing between them. If you think that the feature would be useful to you, please +1 it (each feature has its own GitHub issue that allows you to add reacrions and comments). If the feature you want to see in the Vaadin Form library is missing from this list, please [open a new issue](https://github.com/vaadin/proposal-for-vaadin-form/issues/new) in this repo.

### API access: Custom Elements `<vaadin-form>` and `<vaadin-form-field>` ([#3](https://github.com/vaadin/proposal-for-vaadin-form/issues/3))
This is an HTML-first (template-driven) approach to the API. It is intended for use with a templating library like `lit-html` because such libraries simplify binding JavaScript handlers and values to DOM events and properties. However, there is no dependency on `lit-html` and it can as well be used with plain JavaScript or other frontend libraries.

 - In order to use the Vaadin Form APIs one would need to add a `<vaadin-form>` element into the DOM and place the form contents inside it.
 - Individual form fields would need to be wrapped into `<vaadin-form-field>` elements.
 - The library functionality is exposed via the custom elements' properties and DOM events.
 - The link between the form controls (DOM elements) and the Vaadin Form instance is derived from the DOM structure.

```js
import { html } from 'lit-html';

html`<vaadin-form @submit=${this.onSubmit}>
  <vaadin-form-field .validator=${this.validateUsername}>
    <label>User Name <input type="text" name="username"></label>
  </vaadin-form-field>

  <vaadin-form-field>
    <input type="submit" value="Submit">
  </vaadin-form-field>
</vaadin-form>`;
```

### API access: JavaScript classes `VaadinForm` and `VaadinFormField` ([#4](https://github.com/vaadin/proposal-for-vaadin-form/issues/4))
This is a code-first approach to the API. It is intended for use with a statically typed language like TypeScript because in order for type-checker to work with forms they need to be declared in TypeScript, not in HTML. Depending on the implementation choices this approach may be combined with a templating library like `lit-html`.

 - In order to use the Vaadin Form APIs one would need to create an instance of the `VaadinForm` class in their JavaScript / TypeScript code.
 - Individual form fields would be created using the `VaadinFormField` class.
 - The library functionality is exposed via JavaScript properties and methods.
 - The link between the form controls (DOM elements) and the Vaadin Form instance needs to be established separately. There are several approaches to it:
   - link a form instance to the existing DOM using `document.querySelector()`
      ```html
      <form id="myform">
        <label>User Name <input type="text" name="username"></label>
        <input type="submit" value="Submit">
      </form>
      ```

      ```js
      import { VaadinForm } from '@vaadin/vaadin-form';

      const form = new VaadinForm(document.getElementById('myform'));
      form.addSubmitHandler((e) => this.onSubmit(e));
      ```

   - link DOM elements to a form instance in a template when using a library like `lit-html`
      ```js
      import { VaadinForm } from '@vaadin/vaadin-form';
      import { html } from 'lit-html';

      const form = new VaadinForm();

      html`<form>
        ${form(html`
          <label>User Name <input type="text" name="username"></label>
          <input ?disabled=${!form.canSubmit} type="submit" value="Submit">
        `)}
      </form>`;
      ```

   - define a form in JavaScript first, and then let it create the DOM it needs
      ```js
      import { VaadinForm, NativeTextField } from '@vaadin/vaadin-form';

      const form = new VaadinForm();
      form.addField(new NativeTextField({
        label: 'User Name',
        name: 'username'
      });
      form.render(document.getElementById('myform'));
      ```

### Form and field properties: basic ([#5](https://github.com/vaadin/proposal-for-vaadin-form/issues/5))
A form instance detects user interaction with the form DOM and updates its state accordingly. It has a set of properties available through the API so that developers can create good user experiences with immediate response to user input and interactions.

Most properties are available both for the entire form and for individual fields.
However, some properties make sence only for a form or for a field.

  * `touched`: at least one of the form controls has had a `blur` event since the form was reset / first rendered
  * `untouched`: none of the form controls has has a `blur` event since it was reset / first rendered
  ---
  * `dirty`: at least one of the current form values is not shallow equal to its initial value
  * `pristine`: each of the current form values is shallow equal to its initial value
  ---
  * `valid`: no validators have (yet) reported an error in the last validation check
    There may be async validators still running.
  * `invalid`: at least one form or field validator has reported an error in the last validation check
  * `validating`: there is at least one async validator still running from the last validation check
  ---
  * `value` (field-only): the current field value
  * `initialValue` (field-only): the initial field value
  * `message` (field-only): the validation message (if any) from the last validation check
  ---
  * `values` (form-only): the current form values
  * `initialValues` (form-only): the initial form values
  * `messages` (form-only): an array of all field- and form-level validation messages from the last validation check
  * `submitting` (form-only): an async submission operation is progress

### Form and field properties: fine details ([#6](https://github.com/vaadin/proposal-for-vaadin-form/issues/6))
In addition to the form and field properties lised in the [basic set](https://github.com/vaadin/proposal-for-vaadin-form/issues/5), there may be cases that require more fine-graned details of the form state.

  * `visited`: the field has ever received a `focus` event
  * `modified`: the value has been ever modified from its original.
    After the values has been modified once, this flag remains set even if the value is modified again to undo the change.
    The flag is reset when the form is reset.
  * `focused`: the field currenty has focus

### Form properties: field lists for each flag ([#7](https://github.com/vaadin/proposal-for-vaadin-form/issues/7))
When a boolean flag like `touched` is set on a form instance, it may be useful to know which field(s) contribute to that. It can be done by iterating through all fields and checking the flag on each of them, but there could be a more conventient way. There are special properties listing the fields for each flag, already available on the form instance:

  * `touchedFields`: an array of field names that have the `touched` flag
  * `dirtyFields`: an array of field names that have the `dirty` flag
  * `invalidFields`: an array of field names that have the `invalid` flag
  * `validatingFields`: an array of field names that have the `validating` flag
  ---
  * `visitedFields`: an array of field names that have the `visited` flag
  * `modifiedFields`: an array of field names that have the `modified` flag
  * `focusedField`: the field name that currently has focus

### Form rendering
When a form instance detects user interactions with its linked DOM and updates its state, the form may need to be re-rendered (e.g. to show that a field has an invalid value).
This section contains several alternative approaches to form rendering: library-independent, optimized for use with the `lit-element` library, and two optimised for use with the `lit-html` library (one that introduces custom extentions to it, and one that does not).

#### Form rendering: library-independent `renderer()` property ([#8](https://github.com/vaadin/proposal-for-vaadin-form/issues/8))
A `renderer()` functional property is a universal approach for reactive form rendering, independent from any library.

 - In order to render a Vaadin Form one needs to set a `renderer` property on the form instance.
 - `renderer()` is a function that renders the form content based on its properties.
 - The form properties are provided into the `renderer()` function as a parameter.
 - The form reacts to user input, updates its properties and calls the `renderer()` function on every change.
 - The form is able to detect user interactions with the well-known form input elements like `<input>` out-of-the-box.
 - In a generic case the developer needs to hook up event listeners to the rendered DOM and call appropriate handlers on the form instance. Otherwise the form is not able to detect user interactions.
 - The `renderer()` function lets developers to use any rendering library of their choice (e.g. `lit-html`), or stay with plain JavaScript.

This approach is intended for use with the HTML-first API because a custom renderer property affecting the DOM children of a form works better with custom Web Components, than with standard DOM elements.

* HTML-first (+`lit-html`): both form-level and field-level `renderer()`s
  ```js
  import { render, html } from 'lit-html';

  html`<vaadin-form .renderer=${(form, root) => render(html`
    <vaadin-form-field .renderer=${(field, root) => render(html`
      <label>User Name <input type="text" name="username"></label>
      ${field.touched && field.invalid
        ? html`<p class="error">${field.message}</p>`
        : ''}
    `, root)}
    ></vaadin-form-field>

    <button @click=${form.submit} ?disabled=${!form.canSubmit}>
      Submit
    </button>
  `, root)}
  ></vaadin-form>`;
  ```

* Code-first: it would be much easier to compose form- and field-level rendering with a `render()` function that returns some DOM (as opposed to rendering into a given outlet).
  ```js
  import { VaadinForm, NativeTextField } from '@vaadin/vaadin-form';
  import { render, html } from 'lit-html';

  const form = new VaadinForm();
  form.addField(new NativeTextField({
    label: 'User Name',
    name: 'username',
    renderer: (field, root) => render(html`
      <label>User Name <input type="text" name="username"></label>
      ${field.touched && field.invalid
        ? html`<p class="error">${field.message}</p>`
        : ''}
    `, root)
  });
  form.renderer = (form, root) => render(html`
    <!-- The 'rederer()' functions are not easily composable in code
         without having a DOM structure. -->
    <div class="vaadin-form-field" name="username"></div>

    <button @click=${form.submit} ?disabled=${!form.canSubmit}>
      Submit
    </button>
  `, root);
  form.render(document.getElementById('myform'));
  ```

#### Form rendering: `LitElement` integration ([#9](https://github.com/vaadin/proposal-for-vaadin-form/issues/9))
This approach to reactive form rendering depends on the `lit-element` library. Vaadin Forms taps into the `LitElement`'s
change detection mechanism and calls the `renderer()` method on any form property change.

 - In order to render a Vaadin Form one needs to define a web component by extending `LitElement` and use the `render()` method inherited from `LitElement` to render a form
 - Vaadin Form instances should be _registered_ as additional source of observed properties that may trigger re-render for that component.
    - either adding a form instance as a class property and decorating it with `@form`
    - or by using the registration API directly: `registerForm(property, clazz)`
 - The form properties on _registered_ forms are available inside the `render()` method.
 - Changes to the form state are detected by the form instance and trigger a `LitElement` component re-render in the same way as component's own property changes.

```typescript
import { LitElement, customElement, html } from 'lit-element';
import { VaadinForm, form } from '@vaadin/form';

@customElement('field-validation')
class MyComponent extends LitElement {

  @form() form = new VaadinForm();

  render() {
    return html`
      <!-- well-known form inputs just work -->
      <div class="form-field">
        <label>User Name <input type="text" .value=${form.username}></label>
        ${form.username.touched && form.username.invalid
          ? html`<p class="error">${form.username.message}</p>`
          : ''}
      </div>

      <!-- arbitrary form inputs need explicit bindings between
          the DOM and the Vaadin Form's field instance -->
      <div class="form-field">
        <x-custom-form-field
          name="customProp"
          .value=${form.customProp.value}
          @input=${form.customProp.onInput}
          @change=${form.customProp.onChange}
          @blur=${form.customProp.onBlur}
          @focus=${form.customProp.onFocus}
        ></x-custom-form-field>
      </div>

      <button @click=${form.submit} ?disabled=${!form.canSubmit}>
        Submit
      </button>
    `;
  }
}
```

#### Form rendering: a custom `lit-html` directive ([#10](https://github.com/vaadin/proposal-for-vaadin-form/issues/10))
This approach to reactive form rendering depends on the `lit-html` library.
Vaadin Form comes with a set of custom _directives_ for `lit-html` that allow linking DOM elements to a form instance and using the form properties in the template.

 - In order to render a Vaadin Form one needs to define a `lit-html` template for it, and use the `form` and `form.field` directives in that template
 - The directives have a `renderer` parameter which is either a static template, or a renderer function (as described above).
 - The form properties are provided into the `renderer()` function as a parameter.
 - The form reacts to user input, updates its properties and calls the `renderer()` function on every change.
 - The `renderer()` function is expected to return a `lit-html`'s `TemplateResult` object.

```typescript
import { VaadinForm } from '@vaadin/form';
import { html } from 'lit-html';

const form = new VaadinForm();

const template = html`
  ${form(form => html`
    <!-- well-known form inputs just work -->
    <div class="form-field">
      ${form.field(field => html`
        <label>User Name <input type="text" name="username"></label>
        ${field.touched && field.invalid
          ? html`<p class="error">${field.message}</p>` : ''}
      `)}
    </div>

    <!-- arbitrary form inputs need explicit bindings between
         the DOM and the Vaadin Form's field instance -->
    <div class="form-field">
      ${form.field(field => html`
        <x-custom-form-field
          name="customProp"
          .value=${field.value}
          @input=${field.onInput}
          @change=${field.onChange}
          @blur=${field.onBlur}
          @focus=${field.onFocus}
        ></x-custom-form-field>
      `)}
    </div>

    <button @click=${form.submit} ?disabled=${!form.canSubmit}>
      Submit
    </button>
  `)}
`;
```

#### Form rendering: 2-way binding extention to `lit-html` ([#19](https://github.com/vaadin/proposal-for-vaadin-form/issues/19))
This approach to reactive form rendering depends on the `lit-html` library.
Vaadin Form comes with a custom `form` directive and an extention to the `lit-html` syntax to allow 2-way data binding.

 - In order to render a Vaadin Form one needs to define a `lit-html` template for it, and use the `form` directive with a custom 2-way data binding syntax in that template
 - The directive has a `renderer` parameter which is either a static template, or a renderer function (as described above).
 - The form properties are provided into the `renderer()` function as a parameter.
 - The form reacts to user input, updates its properties and calls the `renderer()` function on every change.
 - The `renderer()` function is expected to return a `lit-html`'s `TemplateResult` object.

```typescript
import { VaadinForm } from '@vaadin/form';
import { html } from 'lit-html';

const form = new VaadinForm();

const template = html`
  ${form(form => html`
    <!-- well-known form inputs just work -->
    <div class="form-field">
      <label>User Name <input type="text" !value=${form.username}></label>
      ${form.username.touched && form.username.invalid
        ? html`<p class="error">${form.username.message}</p>`
        : ''}
    </div>

    <button @click=${form.submit} ?disabled=${!form.canSubmit}>
      Submit
    </button>
  `)}
`;
```

### Form validation: a `validator()` function ([#11](https://github.com/vaadin/proposal-for-vaadin-form/issues/11))
 - A `validator()` funciton can be defined for an entire form or for individual form fields.
 - By default Vaadin Form runs `validator` functions on every state change (on every key stroke).
 - The `validator()` function could be used in the same way with any form API or rendering approach.
 - `validator()` functions can be combined in chains to create complex validation rules.

![form-validation](https://user-images.githubusercontent.com/22416150/64430232-811d7380-d0c0-11e9-9505-b07e4fdf6265.png)

```typescript
const form = new VaadinForm();
// init the form

const passwordRepeatedCorrectly = values => {
  if (values.password !== values.passwordRepeated) {
    return `Please check that you've repeated the password correctly.`;
  }
};

form.validator = [passwordRepeatedCorrectly];
```

### Basic validators ([#12](https://github.com/vaadin/proposal-for-vaadin-form/issues/12))
The Vaadin Form library comes with a set of ready-to-use validator functions:
  `min`
, `max`
, `required`
, `notBlank`
, `pattern`
, `minLength`
, `maxLength`
, `numeric`
, etc

### Async validators ([#14](https://github.com/vaadin/proposal-for-vaadin-form/issues/14))
 - `validator()` functions can by asynchronous.
 - The state of a form or a single field has a property to signal that an async validation is in progress. Async and sync validators can be combined.

```typescript
const isAvailableName = async (value) => {
  const response = await fetch(`/validate/name/${encodeURIComponent(value)}`);
  const result = await response.json();
  if (result.error) {
    return `Please pick another name. '${value}' is not available.`;
  }
};
```

### Limiting and debouncing validators ([#13](https://github.com/vaadin/proposal-for-vaadin-form/issues/13))
 - `validator()` functions have access to the source event that trigges validation, and to the target form / field instance.
 - Thus, they can be wrapped into filters or debouncers like `onBlur` or `onSubmit` to avoid running validations too often.
 - The Vaadin Form library comes with a set of ready-to-use filters and debouncers:
  `onBlur` (for fields)
, `onSubmit` (for forms)
, `debounce`
, etc

![field-validation](https://user-images.githubusercontent.com/22416150/64437516-0f98f180-d0cf-11e9-9eee-dc4b057fa03f.gif)

```typescript
import {VaadinForm, onBlur} from '@vaadin/vaadin-form';

const form = new VaadinForm();
// init the form

const isAvailableName = async (value) => {
  const response = await fetch(`/validate/name/${encodeURIComponent(value)}`);
  const result = await response.json();
  if (result.error) {
    return `Please pick another name. '${value}' is not available.`;
  }
};

form.username.validator = [onBlur(isAvailableName)];
```

### Pluggable validators ([#15](https://github.com/vaadin/proposal-for-vaadin-form/issues/15))
 - The `validator` property on a form / field can be added / removed / modified dynamically.
 - That allows using different validation rules depending on the global state of the app (external to the form), or on the values of the other form fields.

### Temporary disabling validation ([#1](https://github.com/vaadin/proposal-for-vaadin-form/issues/1))
Vaadin Form has a `novalidate` boolean property (false by default) that lets temporary disabling form validation (e.g. to save the intermediary form state even if it's invalid to be able to continue editing the form later)

### Support for the `formdata` event: use `VaadinFormField` inside native `<form>`s ([#16](https://github.com/vaadin/proposal-for-vaadin-form/issues/16))
The Vaadin Form library helps creating [_form-assocciated custom elements_](https://web.dev/more-capable-form-controls) by providing a `VaadinFormFieldMixin` (to be used with `LitElement`). With this mixin custom elements can participate in the native form validation and submission pipeline.

### Type-safe forms ([#17](https://github.com/vaadin/proposal-for-vaadin-form/issues/17))
The `VaadinForm` and `VaadinFormField` API have TypeScript type definitions that inclcude a type parameter to define the types of the form fields. That allows build-time type checking of all form-handling code.

```typescript
import {VaadinForm} from '@vaadin/vaadin-form';
import {Order} from './entities/order';

const form = new VaadinForm<Order>();
form.value = new Order(); // type-checked

// form properties are based on the form entity type
form.username.validator = [...];
```

### Cross-field validation ([#18](https://github.com/vaadin/proposal-for-vaadin-form/issues/18))
 - It should be possible to validate a combination of fields together.
 - It should be possible to create a change listener on one field that sets another field or group of fields as required.

## Prior art
When working on this library the core team has studied the examples, API designs and best practices from a number of other libraries, including the form libraries widely used in React, Angular and Vue apps.
 - Final Form for React (https://final-form.org/)
 - Formik for React (https://jaredpalmer.com/formik/)
 - Angular Forms (https://angular.io/guide/forms)
 - Vue Forms (https://vuejs.org/v2/guide/forms.html)
 - Polymer's `<iron-form>` (https://www.webcomponents.org/element/@polymer/iron-form)

## Out of scope
To be added later.

_I want this page to clearly state which use cases are out of scope so that it is easy to point to this list when steering the discussions on this proposal._
