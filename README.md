# Proposal for a `lit-form` library
A helper library for creating forms in [lit-html](https://lit-html.polymer-project.org/).

## `lit-form` goals
_You need a form in your app? Not sure how to do it best? `lit-form` explains what makes a good form UX, and how to implement it in `lit-html` so that the code is clean, bug free and easy to maintain._

 - explain how to create great forms in `lit-html`
 - give ready-to-use examples for common use cases
 - make typical tasks like validation, submission handling, and form state management much easier to implement than with plain `lit-html`
 - support TypeScript to allow static type-checking
 - be small and fast with little run-time overhead

## Status
We are validating the idea, there is no implementation yet.
Your feedback is welcome! Please [create issues](https://github.com/vaadin/proposal-for-lit-form/issues/new) in this repo.

## Overview and motivation
Forms are overwhelmingly common in the kinds of apps people are building with Vaadin. When done without enough effort forms often suck. Both for users, and for developers maintaining the application code later. That's why there are helper libraries out there whose sole purpose is to help developers create forms.

Vaadin helps developers build web apps that users love. Forms have been a strong side of Vaadin so far, and we want it to remain as the front-end technology used in Vaadin evolves over time. When building UIs with `lit-html` templates, developers would be looking to answer the same quesitons about making forms:
 - how to show backend data in a form
 - how to submit a form to the backend
 - how to validate a form (per field / as a whole)
 - how to show that the form does not pass validation
 - how to run validations asynchronously on the backend
 - how to render a form conditionally
 - how to create multi-page forms (aka wizards)
 
There are good and bad examples out there addressing many of these questions for developers building apps with React, Angular and Vue. There is no good place to look for answers and guidance for developers building apps with TypeScript and `lit-html`. The `lit-form` library would become such place. It would be the go-to starting point for developers creating forms in `lit-html`.

## Code samples
_I want this page to give specific examples in TS / HTML so that the discussion can be specific, and cented around a shared context._

In scope
 - simple example (binding, no validation)
 - field-level valiation
 - form-level validation
 - async validation

## Prior art
This library takes inspiration from the form libraries widely used in React, Angular and Vue apps:
 - Final Form for React (https://final-form.org/)
 - Formik for React (https://jaredpalmer.com/formik/)
 - Angular Forms (https://angular.io/guide/forms)
 - Vue Forms (https://vuejs.org/v2/guide/forms.html)

## Out of scope
To be added later.

_I want this page to clearly state which use cases are out of scope so that they can be easily excluded from any discussions, and the dscussion can stay fosued on the featues in scope._
