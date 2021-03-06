# Angular 2 component for Google reCAPTCHA

## ng2-recaptcha [![npm version](https://badge.fury.io/js/ng2-recaptcha.svg)](http://badge.fury.io/js/ng2-recaptcha)

[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/dethariel/ng2-recaptcha/master/LICENSE)
[![Build Status](https://travis-ci.org/DethAriel/ng2-recaptcha.svg?branch=master)](https://travis-ci.org/DethAriel/ng2-recaptcha)
[![devDependency Status](https://david-dm.org/dethariel/ng2-recaptcha/dev-status.svg)](https://david-dm.org/dethariel/ng2-recaptcha?type=dev)

A simple, configurable, easy-to-start component for handling reCAPTCHA.

## Installation

The easiest way is to install trough [npm](https://www.npmjs.com/package/ng2-recaptcha):
```
npm i ng2-recaptcha --save
```

In order to take advantage of type-checking system you should also install `grecaptcha` typings:

```
typings install dt~grecaptcha --save --global
```

Or, if you're using TypeScript 2 or `angular-cli`:

```
npm install @types/grecaptcha --save-dev
```

## <a name="example-basic"></a>Usage [(see in action)](https://dethariel.github.io/ng2-recaptcha/basic)

To start with, you need to add one of the `Recaptcha` modules (more on that [later](#modules)):

```typescript
import { RecaptchaModule } from 'ng2-recaptcha';
import { BrowserModule }  from '@angular/platform-browser';

@NgModule({
  bootstrap: [MyApp],
  declarations: [MyApp],
  imports: [
    BrowserModule,
    RecaptchaModule.forRoot(), // Keep in mind the "forRoot"-magic nuances!
  ],
})
export class MyAppModule { }
```

Once you have done that, the rest is simple:

```typescript
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { Component } from '@angular/core';

@Component({
    selector: 'my-app',
    template: `<recaptcha (resolved)="resolved($event)" siteKey="YOUR_SITE_KEY"></recaptcha>`,
}) export class MyApp {
    resolved(captchaResponse: string) {
        console.log(`Resolved captcha with response ${captchaResponse}:`);
    }
}

platformBrowserDynamic().bootstrapModule(MyAppModule);
```

## <a name="modules"></a>Modules: "Forms"-ready and "No-forms"

There are two modules available for you:

```typescript
import { RecaptchaModule } from 'ng2-recaptcha';
import { RecaptchaNoFormsModule } from 'ng2-recaptcha/ng2-recaptcha.noforms';
```

The difference between them consists in dependencies - `RecaptchaModule` depends on
`@angular/forms`, while `RecaptchaNoFormsModule` does not. If you do not rely on
Angular 2 forms in your project, you should use the "no-forms" module version, as
it does not require the `@angular/forms` package to be bundled with your code.

## Options

The component supports this options:

* `siteKey`
* `theme`
* `type`
* `size`
* `tabIndex`

They are all pretty well described in the [reCAPTCHA docs](https://developers.google.com/recaptcha/docs/display),
so I won't duplicate it here.

## Events

* `resolved(response: string)`. Occurs when the captcha resolution value changed.
  When user resolves captcha, use `response` parameter to send to the server for verification.
  This parameter is equivalent to calling [`grecaptcha.getResponse`](https://developers.google.com/recaptcha/docs/display#js_api).

  If the captcha has expired prior to submitting its value to the server, the component
  will reset the captcha, and trigger the `resolved` event with `response === null`.

## Methods

* `reset`. Performs a manual captcha reset. This method might be useful if your form
validation failed, and you need the user to re-enter the captcha.

## <a name="example-language"></a>Specifying a different language [(see in action)](https://dethariel.github.io/ng2-recaptcha/language)

`<recaptcha>` supports various languages. But this settings is global, and cannot be set
on a per-captcha basis. This can be overridden by providing your own instance of
`RecaptchaLoaderService` for a particular module:

```typescript
import { RecaptchaLoaderService } from 'ng2-recaptcha';

@NgModule({
  providers: [
    {
      provide: RecaptchaLoaderService,
      useValue: new RecaptchaLoaderService("fr"), // use French language
    },
  ],
}) export class MyModule { }
```

You can find the list of supported languages in [reCAPTCHA docs](https://developers.google.com/recaptcha/docs/language).

## <a name="example-preload-api"></a>Loading the reCAPTCHA API by yourself [(see in action)](https://dethariel.github.io/ng2-recaptcha/preload-api)

By default, the component assumes that the reCAPTCHA API loading will be handled
by the `RecaptchaLoaderService`. However, you can override that by providing your
instance of this service to the Angular DI.

The below code snippet is an example of how such a provider can be implemented.

**TL;DR**: there should be an `Observable` that eventually resolves to a
`grecaptcha`-compatible object (e.g. `grecaptcha` itself).

```html
<script src="https://www.google.com/recaptcha/api.js?render=explicit&amp;onload=onloadCallback"></script>

<script>
    // bootstrap the application once the reCAPTCHA api has loaded
    function onloadCallback() {
        System.import('app').catch(function(err) { console.error(err); });
    }
</script>
```

```typescript
import { RecaptchaLoaderService } from 'ng2-recaptcha';

@Injectable()
export class PreloadedRecaptchaAPIService {
  public ready: Observable<ReCaptchaV2.ReCaptcha>;

  constructor() {
    let readySubject = new BehaviorSubject<ReCaptchaV2.ReCaptcha>(grecaptcha);
    this.ready = readySubject.asObservable();
  }
}

@NgModule({
  providers: [
    {
      provide: RecaptchaLoaderService,
      useValue: new PreloadedRecaptchaAPIService(),
    },
  ],
}) export class MyModule { }
```

## <a name="example-forms"></a>Usage with `required` in forms [(see in action)](https://dethariel.github.io/ng2-recaptcha/forms)

It's very easy to put `recaptcha` in an Angular2 form and have it `require`d - just
add the `required` attribute to the `<recaptcha>` element

```typescript
@Component({
  selector: 'my-form',
  template: `
  <form>
    <recaptcha
      [(ngModel)]="formModel.captcha"
      name="captcha"
      required
      siteKey="YOUR_SITE_KEY"
    ></recaptcha>
  </form>`,
}) export class MyForm {
  formModel = new MyFormModel();
}
```
