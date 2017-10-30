# Ng[SweetAlert2](https://github.com/limonte/sweetalert2) [![npm version](https://img.shields.io/npm/v/@toverux/ngsweetalert2.svg?style=flat-square)](https://www.npmjs.com/package/@toverux/ngsweetalert2) ![license](https://img.shields.io/github/license/toverux/ngsweetalert2.svg?style=flat-square) ![npm total downloads](https://img.shields.io/npm/dt/@toverux/ngsweetalert2.svg?style=flat-square)

[SweetAlert2](https://github.com/limonte/sweetalert2) integration for Angular. This is not a regular API wrapper for SweetAlert (which already works very well alone), it intends to provide Angular-esque utilities on top of it.

**Before posting an issue**, please check that the problem isn't on SweetAlert's side.

----------------

 - [Installation & Usage](#package-installation--usage)
 - [`[swal]` directive](#swaldirective) — for simple, one-liner dialogs
 - [`<swal>` component](#swalcomponent) — for advanced use cases and extended Swal2 API coverage
 - [`*swalPartial` directive](#swalpartialdirective) — use Angular dynamic templates in `<swal>`

----------------

## :package: Installation & Usage

1) Install NgSweetAlert2 and SweetAlert2 via the npm registry:

```bash
npm install --save sweetalert2 @toverux/ngsweetalert2
```

:arrow_double_up: Always upgrade _SweetAlert2_ when you upgrade _NgSweetAlert2_. The latter is statically linked with SweetAlert2's type definitions.

2) Import SweetAlert's CSS (or SCSS) file, do it like you're usually doing with vendor styles.
   Using Angular CLI for example, you can put this in your `.angular-cli.json`:

```json
{
  "apps": [
    {
      "styles": ["styles.css", "../node_modules/sweetalert2/dist/sweetalert2.css"]
```
  
3) Finally, import the module:

```typescript
import { SweetAlert2Module } from '@toverux/ngsweetalert2';

@NgModule({
    //=> Basic usage
    imports: [SweetAlert2Module.forRoot()],
    
    //=> Or provide default SweetAlert2-native options
    //   (here, we make Swal more Bootstrap-friendly)
    imports: [
        SweetAlert2Module.forRoot({
            buttonsStyling: false,
            customClass: 'modal-content',
            confirmButtonClass: 'btn btn-primary',
            cancelButtonClass: 'btn'
        })
    ],
    
    //=> In submodules only:
    imports: [SweetAlert2Module]
})
export class AppModule {
}
```

## :link: API

### `SwalDirective`

Add the `[swal]` attribute to an element to show a simple modal when that element is clicked.

To define the modal contents, you can pass a [`SweetAlertOptions` (provided by sweetalert2)](https://github.com/limonte/sweetalert2/blob/master/sweetalert2.d.ts#L225) object, or a simple array of strings, of format `[title: string, text: string (, type: string)]`.

Simple dialog:

```html
<button [swal]="['Oops!', 'This is not implemented yet :/', 'warning']">
  Do it!
</button>
```

More advanced (input in dialog, dismissal handling):

```html
<button 
  [swal]="{ title: 'Enter your email', input: 'email' }"
  (confirm)="saveEmail($event)"
  (cancel)="handleRefusalToSetEmail($event)">
  
  Set my e-mail address
</button>
```

```typescript
export class MyComponent {
  public saveEmail(email: string): void {
    // ... save user email
  }

  public handleRefusalToSetEmail(dismissMethod: string): void {
    // dismissMethod can be 'cancel', 'overlay', 'close', and 'timer'
    // ... do something
  }
}
```

The directive can also take a reference to a [`<swal>` component](#swalcomponent) for more advanced use cases:

```html
<button [swal]="deleteSwal" (confirm)="deleteFile(file)">
  Delete {{ file.name }}
</button>

<swal #deleteSwal ...></swal>
```

### `SwalComponent`

The library also provides a component, that can be useful for advanced use cases, or when you `[swal]` has too much options.

The component also allows you to use Angular dynamic templates inside the SweetAlert (see the [`*swalPartial` directive](#swalpartial) for that).

Simple example:

```html
<swal 
  #deleteSwal
  title="Delete {{ file.name }}?"
  text="This cannot be undone"
  type="question"
  [showCancelButton]="true"
  [focusCancel]="true"
  (confirm)="deleteFile(file)">
</swal>

With [swal]:
<button [swal]="deleteSwal">Delete {{ file.name }}</button>

Or DIY:
<button (click)="deleteSwal.show()">Delete {{ file.name }}</button>
```

You can access the dialog from your TypeScript code-behind like this:

```typescript
class MyComponent {
  @ViewChild('deleteSwal') private deleteSwal: SwalComponent;
}
```

You can pass native SweetAlert2 options via the `options` input, just in case you need that:

```html
<swal [options]="{ confirmButtonText: 'I understand' }"></swal>
```

You can catch other modal lifecycle events than (confirm) or (cancel):

```html
<swal (onBeforeOpen)="onBeforeOpen($event)" (onOpen)="onOpen($event)" (onClose)="onClose($event)"></swal>
```

```typescript
public onBeforeOpen(event: OnBeforeOpenEvent): void {
  // You can access the modal's native DOM node:
  console.log(event.modalElement);
}
```

### `SwalPartialDirective`

So you really want more, huh?

The `*swalPartial` directive lets you use Angular dynamic templates inside SweetAlerts. The directive will replace certain parts of the modal (aka. _swal targets_) with embedded Angular views.

This allows you to have data binding, use directives and components and benefit from the Angular template syntax like if the SweetAlert was a normal Angular component (it's not. at all).

```html
<swal title="SweetAlert2 Timer">
  <div *swalPartial class="alert alert-info">
    <strong>{{ elapsedSeconds }}</strong> seconds elapsed since then.
  </div>
<swal>
```

The other cool thing about using a structural directive is that the modal's contents won't be instantiated before the modal is shown.

> But, I want to use a template inside the modal's _title_. Your example only sets the main content.

You just have to change the _target_ of the partial view (_`content`_ is the default target). First, inject this service in your component:

```typescript
export class MyComponent {
  public constructor(public readonly swalTargets: SwalPartialTargets) {
  }
}
```

And then, set the appropriate target as the value of `*swalPartial`.

```html
<swal title="Fill the form, rapidly">
  <!-- This form will be displayed as the alert main content
       Targets the alert's main content zone by default -->
  <form *swalPartial [formControl]="myForm">
    ...
  </form>
        
  <!-- This targets the confirm button's inner content
       Notice the usage of ng-container to avoid creating an useless DOM element inside the button -->
  <ng-container *swalPartial="swalTargets.confirmButton">
    Send ({{ secondsLeft }} seconds left)
  </ng-container>
<swal>
```

We have the following targets: `title`, `content`, `confirmButton`, `cancelButton`, `buttonsWrapper`. These are provided by SweetAlert2.
