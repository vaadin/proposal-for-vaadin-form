# Vaadin Form: Proposal
We are planning to build a small and fast framework-agnostic library for creating forms with Web Components. Strong typing using TypeScript. Lot of examples to use good UX patterns.

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

The specific APIs used in the code snippets below are a very early sketch and may very well change in the final product. The snippets are here to illustrate the key concepts and use cases described after each code snippet.

_NOTE: All code samples in this section show how to use Vaadin Form in [`lit-html`](https://lit-html.polymer-project.org/) templates._

### Render a basic form
This example shows how to use Vaadin Form to render a basic form with a single text input and a submit button.
The submit button enabled / disabled state changes dynamically depending on the form state.

![basic-form](https://user-images.githubusercontent.com/22416150/64426539-75797f00-d0b7-11e9-9fb3-ed1ca9ff98db.png)
```typescript
html`
<vaadin-form
  .renderer=${(root, form) => render(html`
    <vaadin-form-field>
      <label>First Name
        <input type="text" name="firstName" placeholder="First Name" maxlength="16">
      </label>
    </vaadin-form-field>

    <button @click=${form.submit}
      ?disabled=${form.pristine || form.submitting || form.error}
    >
      Submit
    </button>
  `, root)}
></vaadin-form>`
```

Key points this example illusrates:
 - the form is created using the `<vaadin-form>` and `<vaadin-form-field>` Web Components
 - it is possible to put any standard or custom HTML elements inside `<vaadin-form>` and `<vaadin-form-field>`
 - the contents of the form is rendered using its `renderer` functional property
 - the `renderer` function allows using the form state (`form.pristine`, `form.submitting`, and `form.error`), and actions (`form.submit`) when rendering the form
 - the `renderer` function works well with [`lit-html`](https://lit-html.polymer-project.org/), but can be used also without it

### Validate a form before submitting
This example shows how to use Vaadin Form to validate a form before it is submitted.

![form-validation](https://user-images.githubusercontent.com/22416150/64430232-811d7380-d0c0-11e9-9505-b07e4fdf6265.png)

```typescript
import {onSubmit} from '@vaadin/vaadin-form';

const passwordRepeatedCorrectly = values => {
  if (values.password !== values.passwordRepeated) {
    return `Please check that you've repeated the password correctly.`;
  }
};

html`
<vaadin-form
  .validator=${onSubmit(passwordRepeatedCorrectly)}
  .renderer=${(root, form) => render(html`
    <vaadin-form-field>
      <label>Password <input type="password" name="password"></label>
    </vaadin-form-field>

    <vaadin-form-field>
      <label>Repeat password <input type="password" name="passwordRepeated"></label>
    </vaadin-form-field>

    ${form.error ? html`<p class="error">${form.errorMessage}</p>` : ''}

    <button @click=${form.submit}
      ?disabled=${form.pristine || form.submitting || form.error}
    >
      Submit
    </button>
  `, root)}
></vaadin-form>`
```

Key points this example illusrates:
 - the form is validated by calling the `validator` function and giving it all form fields as a `values` object parameter
 - the `validator` function may return an error message (which means a validation error) or nothing (which means no errors)
 - the `validator` function runs on every field change (every key stroke), but that can be debounced or limited to 'run only on submit'
 - the validation status and the error message from the `validator` function are available to the `renderer` function via `form.error` and `form.errorMessage`

### Validate a single field (async)
This example shows how to use Vaadin Form to validate a single field, and create a custom field renderer to show field-related error messages. It also shows how to run validation asynchronously.

![field-validation](https://user-images.githubusercontent.com/22416150/64437516-0f98f180-d0cf-11e9-9eee-dc4b057fa03f.gif)

```typescript
import {onBlur} from '@vaadin/vaadin-form';

const isAvailableName = async (value) => {
  const response = await fetch(`/validate/name/${encodeURIComponent(value)}`);
  const result = await response.json();
  if (result.error) {
    return `Please pick another name. '${value}' is not available.`;
  }
};

html`
<vaadin-form
  .renderer=${(root, form) => render(html`
    <vaadin-form-field
      name="username"
      .validator=${onBlur(isAvailableName)}
      .renderer=${(root, field) => render(html`
      <label>
        Username
        <input type="text"
          .value=${field.value}
          @input=${field.onInput}
          @change=${field.onChange}
          @blur=${field.onBlur}
          @focus=${field.onFocus}>
        ${field.validating ? html`<div class="spinner">validating...</div>` : ''}
      </label>
      <p class="error">
        ${field.error && field.touched ? field.errorMessage : ''}
      </p>`, root)}
    ></vaadin-form-field>
  `, root)}
></vaadin-form>`
```

Key points this example illustrates:
 - the `validator` and `renderer` function properties can be set on individual `<vaadin-form-field>` elements as well
 - when a field has a `renderer` function, it calls that function to create its light DOM
 - field `validator` functions run on every keystroke targeting that field, but that can be debounced or limited to 'run only on blur'
 - `validator` functions may be async _(that applies both to form-level and field-level validators)_
 - the validation status and the error message from the field's `validator` function are available to the field's `renderer` function via `field.validating`, `field.error` and `field.errorMessage`

### Integration with Vaadin components
This example shows how to use Vaadin Form together with Vaadin components such as `<vaadin-text-field>`.

![vaadin-components](https://user-images.githubusercontent.com/22416150/64441727-175c9400-d0d7-11e9-9284-9a99109ef1e9.gif)

```typescript
import {onBlur} from '@vaadin/vaadin-form';

const isAvailableName = async (value) => {
  const response = await fetch(`/validate/name/${encodeURIComponent(value)}`);
  const result = await response.json();
  if (result.error) {
    return `Please pick another name. '${value}' is not available.`;
  }
};

html`
<vaadin-form
  .renderer=${(root, form) => render(html`
    <vaadin-form-field name="username" .validator=${onBlur(isAvailableName)}>
      <vaadin-text-field label="Username"></vaadin-text-field>
    </vaadin-form-field>

    <vaadin-button theme="primary"
      @click=${form.submit}
      ?disabled=${form.pristine || form.submitting || form.error}
    >
      Submit
    </vaadin-button>
  `, root)}
></vaadin-form>`
```

Key points this example illusrates:
 - when using Vaadin components inside a `<vaadin-form>` there is no need to create any markup in order to display a validation spinner or an error message. These features are available out-of-the-box.
 - when `<vaadin-form-field>` finds a light DOM child that implements `VaadinFormFieldMixin`, it automatically hooks up the event handlers and field properties, so that there is no need to define a custom field renderer
 - it is possible to create non-Vaadin Web Components that implement `VaadinFormFieldMixin`. Such Web Components would require less code to be used inside Vaadin Forms.

## Individual features
This section describes individual features that could be included into the Vaadin Forms library.
Some of the features are about alternative ways of getting the same outcome (e.g. Custom Elements for the API access vs. JS classes for API access), and some are complementary (e.g. Custom Elements for API access and rendering forms with a `renderer()` functional-reactive property).

Listing features separately allows discussing them separately and eventually prioritizing between them. If you think that the feature would be useful to you, please +1 it (each feature has its own GitHub issue that allows you to add reacrions and comments). If the feature you want to see in the Vaadin Form library is missing from this list, please open a new issue in this repo.

### API access: Custom Elements `<vaadin-form>` and `<vaadin-form-field>` ([#3](https://github.com/vaadin/proposal-for-vaadin-form/issues/3))
 - In order to use the Vaadin Form APIs one would need to add a number of custom elements into the DOM.
 - The library functionality is exposed via the custom elements' attributes, properties and DOM events.

### API access: JS classes `VaadinForm` and `VaadinFormField` ([#4](https://github.com/vaadin/proposal-for-vaadin-form/issues/4))
 - In order to use the Vaadin Form APIs one would need to create an instance of the `VaadinForm` class in their JavaScript / TypeScript code.
 - The library functionality is exposed via JS properties and methods.

### Form and field properties: basic ([#5](https://github.com/vaadin/proposal-for-vaadin-form/issues/5))
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
  * `visited`: the field has ever received a `focus` event
  * `modified`: the value has been ever modified from its original.
    After the values has been modified once, this flag remains set even if the value is modified again to undo the change.
    The flag is reset when the form is reset.
  * `focused`: the field currenty has focus

### Form properties: field lists for each flag ([#7](https://github.com/vaadin/proposal-for-vaadin-form/issues/7))
  * `touchedFields`: an array of field names that have the `touched` flag
  * `dirtyFields`: an array of field names that have the `dirty` flag
  * `invalidFields`: an array of field names that have the `invalid` flag
  * `validatingFields`: an array of field names that have the `validating` flag
  ---
  * `visitedFields`: an array of field names that have the `visited` flag
  * `modifiedFields`: an array of field names that have the `modified` flag
  * `focusedField`: the field names that currently has focus

### Form rendering: functional-reactive `renderer()` property ([#8](https://github.com/vaadin/proposal-for-vaadin-form/issues/8))
 - In order to render a Vaadin Form one needs to create a `<vaadin-form>` element in the DOM and set a `renderer` property on it.
 - `renderer()` is a function that renders the form content based on its properties.
 - The form properties are provided into the `renderer()` function as a parameter.
 - The form reacts to user input, updates its properties and calls the `renderer()` function on every change.
 - The `renderer()` function lets developers to use any rendering library of their choice (e.g. `lit-html`).

### Form rendering: `LitElement` + `VaadinFormMixin` ([#9](https://github.com/vaadin/proposal-for-vaadin-form/issues/9))
 - In order to render a Vaadin Form one needs to define a web component by extending `VaadinFormMixin(LitElement)` and use the `render()` method inherited from `LitElement`.
 - The form properties are available inside the `render()` method through the `this.form` property (defined by the mixin).
 - Changes to the form state are detected by `VaadinFormMixin` and trigger a re-render in the same way as component's own property changes.

### Form rendering: a custom `lit-html` directive ([#10](https://github.com/vaadin/proposal-for-vaadin-form/issues/10))
 - In order to use Vaadin Form one needs to use a `lit-html` template with a `form.field` directive.
 - The directive has a `renderer` parameter which is either a static template, or a renderer function (as described above).
 - The form properties are provided into the `renderer()` function as a parameter.
 - The form reacts to user input, updates its properties and calls the `renderer()` function on every change.
 - The `renderer()` function is expected to return a `lit-html`'s `TemplateResult` object.

### Form validation: a `validator()` function ([#11](https://github.com/vaadin/proposal-for-vaadin-form/issues/11))
 - A `validator()` funciton can be defined for an entire form or for individual form fields.
 - By default Vaadin Form runs `validator` functions on every state change (on every key stroke).
 - The `validator()` function could be used in the same way with any form API or rendering approach.
 - `validator()` functions can be combined in chains to create complex validation rules.

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

### Limiting and debouncing validators ([#13](https://github.com/vaadin/proposal-for-vaadin-form/issues/13))
 - `validator()` functions have access to the source event that trigges validation, and to the target form / field instance.
 - Thus, they can be wrapped into filters or debouncers like `onBlur` or `onSubmit` to avoid running validations too often.
 - The Vaadin Form library comes with a set of ready-to-use filters and debouncers:
  `onBlur` (for fields)
, `onSubmit` (for forms)
, `debounce`
, etc

### Async validators ([#14](https://github.com/vaadin/proposal-for-vaadin-form/issues/14))
 - `validator()` functions can by asynchronous.
 - The state of a form or a single field has a property to signal that an async validation is in progress. Async and sync validators can be combined.

### Pluggable validators ([#15](https://github.com/vaadin/proposal-for-vaadin-form/issues/15))
 - The `validator` property on a form / field can be added / removed / modified dynamically.
 - That allows using different validation rules depending on the global state of the app (external to the form), or on the values of the other form fields.

### Temporary disabling validation ([#1](https://github.com/vaadin/proposal-for-vaadin-form/issues/1))
Vaadin Form has a `novalidate` boolean property (false by default) that lets temporary disabling form validation (e.g. to save the intermediary form state even if it's invalid to be able to continue editing the form later)

### Support for the `formdata` event: use `VaadinFormField` inside native `<form>`s ([#16](https://github.com/vaadin/proposal-for-vaadin-form/issues/16))
The Vaadin Form library helps creating [_form-assocciated custom elements_](https://web.dev/more-capable-form-controls) by providing a `VaadinFormFieldMixin` (to be used with `LitElement`). With this mixin custom elements can participate in the native form validation and submission pipeline.

### Type-safe forms ([#17](https://github.com/vaadin/proposal-for-vaadin-form/issues/17))
The `VaadinForm` and `VaadinFormField` API have TypeScript type definitions that inclcude a type parameter to define the types of the form fields. That allows build-time type checking of all form-handling code.

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
