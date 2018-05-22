[angular]: ./assets/logos/angular-small.svg
[react]: ./assets/logos/react-small.svg

<h1 align="center">Frontend Frameworks Code Comparison</h1>

<div align="center">

![./assets/logos/angular.svg](./assets/logos/angular.svg)
![./assets/logos/react.svg](./assets/logos/react.svg)

</div>

Comparison of different approaches in writing web applications. It is especially useful when migrating between frameworks or switching projects often.

All examples follow the current best practices and conventions that are used inside the community of a given framework. Angular code is written in TypeScript.


<h1 align="center">Table of contents</h1>

  * [Table of contents](#table-of-contents)
  * [Simple component](#simple-component)
  * [Dependency injection](#dependency-injection)
  * [Templates](#templates)
  * [Interpolation](#interpolation)
  * [Handling DOM Events](#handling-dom-events)
  * [Inputs and Outputs](#inputs-and-outputs)
  * [Default inputs](#default-inputs)
  * [Lifecycle methods](#lifecycle-methods)
  * [Conditional rendering](#conditional-rendering)
  * [Lists](#lists)
  * [Filters](#filters)
  * [Child nodes](#child-nodes)
  * [Transclusion and Containment](#transclusion-and-containment)
  * [Inject HTML template](#inject-html-template)
  * [Class toggling](#class-toggling)
  * [Data binding](#data-binding)
  * [Forms](#forms)
  * [Styling](#styling)

# Simple component


## ![angular] Angular

```ts
import { Component } from '@angular/core';
import { Logger } from 'services/logger';
import { Auth } from 'services/auth';
import { Notification } from 'services/notification';

@Component({
  selector: 'change-password',
  templateUrl: './ChangePassword.component.html',
  styleUrls: ['./ChangePassword.component.scss'],
})
export class ChangePasswordComponent {
  password: string = '';

  constructor(
    private logger: Logger,
    private auth: Auth,
    private notification: Notification,
  ) {}

  changePassword() {
    this.auth.changePassword(this.password).subscribe(() => {
      this.notification.info('Password has been changes successfully');
    }).catch(error => {
      this.logger.error(error);
      this.notification.error('There was an error. Please try again');
    });
  }
}
```

Every component has to be declared inside a module in order to be used within this module's other components (it's not available outside of).

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { ChangePasswordComponent } from './change-password.component';

@NgModule({
  imports: [CommonModule],
  declarations: [ChangePasswordComponent],
})
export class ChangePasswordModule {}
```

:link: https://angular.io/api/core/Component

## ![react] React

```jsx
import Logger from 'utils/logger';
import Auth from 'actions/auth';
import Notification from 'utils/notification';

export class ChangePassword {
  state = {
    password: '',
  };

  changePassword() {
    Auth.changePassword(this.state.password).then(() => {
      Notification.info('Password has been changed successfully.');
    }).catch(error => {
      Logger.error(error);
      Notification.error('There was an error. Please try again.');
    });
  }

  render() {
    return <div>{ /* template */ }</div>;
  }
}
```

:link: https://reactjs.org/docs/react-component.html


# Dependency injection

## ![angular] Angular

You specify the definition of the dependencies in the constructor (leveraging TypeScript's constructor syntax for declaring parameters and properties simultaneously).

```ts
import { Component } from '@angular/core';
import { Logger } from 'services/logger';
import { Auth } from 'services/auth';
import { Notification } from 'services/notification';

@Component({
  selector: 'change-password',
  templateUrl: './ChangePassword.component.html',
})
export class ChangePasswordComponent {
  constructor(
    private logger: Logger,
    private auth: Auth,
    private notification: Notification,
  ) {}

  handleEvent() {
    this.notification.info('Password changed successfully');
    this.logger.info('Password changed successfully');
  }
}
```

:link: https://angular.io/guide/dependency-injection

## ![react] React

There's no special injection mechanism. ES2015 modules are used for dependency management.

```jsx
import React, { Component } from 'react';
import Logger from 'utils/logger';
import Notification from 'utils/notification';

export class ChangePassword extends Component {
  handleEvent = () => {
    Notification.info('Password changed successfully');
    Logger.info('Password changed successfully');
  }

  render() {
    return <div>{ /* template */ }</div>;
  }
}
```


# Templates

## ![angular] Angular

There are three kinds of attributes that can be passed:
- text binding, e.g. `size="string"`
- property binding, e.g. `[disabled]="value"`
- event binding, e.g. `(click)="eventHandler()"`

```html
<primary-button size="big"
                [disabled]="true"
                (click)="saveContent()">
  Save
</primary-button>
```

## ![react] React

Templates in React are written inside the JavaScript file using the [JSX language](https://reactjs.org/docs/jsx-in-depth.html). This allows us to utilize all JavaScript capabilities. JSX uses the uppercase and lowercase convention to distinguish between the user-defined components and DOM elements.

```jsx
<PrimaryButton
  size="big"
  disabled
  onClick={ this.saveContent }
>
  Save
</PrimaryButton>;
```

# Interpolation

## ![angular] Angular

Angular is similar to AngularJS, so we use double curly braces (`{{ }}`) for interpolation. Since Angular offers property binding you often have a choice to use it [instead of interpolation](https://angular.io/guide/template-syntax#property-binding-or-interpolation).

```html
<img [src]="image.url" alt="{{ image.alt }}" />
```

`[src]` presents property binding while the `alt` attribute is being interpolated.

:link: https://angular.io/guide/template-syntax#interpolation

## ![react] React

React uses single curly braces for interpolation. Any JavaScript can be interpolated.

```jsx
<img src={ this.props.image.url } alt={ this.props.image.alt } />;
```

:link: https://reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx


# Handling DOM Events


## ![angular] Angular

There's a special syntax for binding to element's events with `()`. The target inside the `()` is an event we want to listen for.

```ts
export interface MenuItem {
  label: string;
  callback?: Function;
}
```

```ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { MenuItem } from './menu-item.interface';

@Component({
  selector: 'menu-topbar',
  template: require('./menuTopbar.html'),
})
export class MenuTopbarComponent {
  private selected = null;
  @Input() items: MenuItem[];

  handleClick(item: MenuItem) {
    if (this.selected !== item) {
      this.selected = item;
      if (item.callback) {
        item.callback();
      }
    }
  }
}
```

```html
<ul>
  <li *ngFor="let item of items"
      (click)="handleClick(item)">
    {{ item.label }}
  </li>
</ul>
```

To bind to component's host element events, you can use [`HostListener`](https://angular.io/api/core/HostListener) decorator.

```ts
@Component({
  selector: 'menu-topbar',
  template: require('./menuTopbar.html'),
})
export class MenuTopbarComponent {
  @HostListener('mouseenter') onMouseEnter() {
    this.highlight('#DDD');
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(null);
  }

  private highlight(color) {
    /* ... */
  }
```

## ![react] React

Handling events with React elements is very similar to handling events on DOM elements. There are two syntactic differences though.

- React events are named using camelCase, rather than lowercase e.g. (onClick, onFocus, onKeyPress).
- With JSX you pass a function as the event handler, rather than a string.

Your event handlers will be passed instances of [`SyntheticEvent`](https://reactjs.org/docs/events.html), a cross-browser wrapper around the browser’s native event. It has the same interface as the browser’s native event, including [`stopPropagation()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation) and [`preventDefault()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault), except the events work identically across all browsers.

```jsx
import React, { Component } from 'react';

export class MenuTopbar extends Component {
  state = {
    selected: null,
  };

  handleClick(item) {
    if (this.selected !== item) {
      this.setState({ selected: item });
      if (item.callback) {
        item.callback();
      }
    }
  }

  render() {
    return (
      <ul>
        { this.props.items.map(item => (
          <li
            key={ item.label }
            onClick={ () => this.handleClick(item) }
          >
            { item.label }
          </li>
        ))
        }
      </ul>
    );
  }
}
```

:link: https://reactjs.org/docs/handling-events.html


# Inputs and Outputs


## ![angular] Angular

Inputs are defined using the [@Input](https://angular.io/api/core/Input) decorator while outputs using the [@Output](https://angular.io/api/core/Output) decorator.

```ts
@Component({
  selector: 'user-preview',
  template: require('./userPreview.html'),
})
export class UserPreviewComponent {
  private editedUser: User;
  @Input() user: User;
  @Output() onEdit: EventEmitter = new EventEmitter<User>();

  ngOnInit() {
    this.editedUser = {
      name: this.user.name,
      email: this.user.email,
    };
  }

  submitEdit() {
    this.onEdit.emit(this.editedUser);
  }
}
```

```html
<form (ngSubmit)="submitEdit()">
  <input type="text" [(ngModel)]="editedUser.name">
  <input type="text" [(ngModel)]="editedUser.email">
  <input type="submit" value="Submit" />
</form>
```

In a parent component:

```ts
@Component({
  selector: 'settings',
  template: require('./settings.html'),
})
export class SettingsComponent {
  user: User = {
    name: 'John Smith',
    email: 'john.smith@example.com',
  };

  editUser(user: User) {
    this.user = Object.assign({}, this.user, user);
    console.log('User has been edited: ', user);
  }
}
```

```html
<user-preview [user]="user"
              (onEdit)="editUser($event)"
></user-preview>
```

:link: https://angular.io/guide/component-interaction

## ![react] React

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { User } from 'utils';

class UserPreviewComponent extends Component {
  submitEdit = () => {
    this.props.onEdit({
      name: this.state.name,
      email: this.state.email,
    });
  };

  handleInputChange({ target }) {
    this.setState({
      [target.name]: target.value,
    });
  }

  render() {
    return (
      <form onSubmit={ this.submitEdit }>
        <input
          type="text"
          name="name"
          value={ this.state.name }
          onChange={ this.handleInputChange }
        />
        <input
          type="email"
          name="email"
          value={ this.state.email }
          onChange={ this.handleInputChange }
        />
        <input type="submit" value="Submit" />
      </form>
    );
  }
}

UserPreviewComponent.propTypes = {
  user: PropTypes.instanceOf(User),
  onEdit: PropTypes.func,
};
```

In a parent component:

```jsx
import React, { Component } from 'react';
import { User } from 'utils';

export class SettingsComponent extends Component {
  state = {
    user: {
      name: 'John Smith',
      email: 'john.smith@example.com',
    },
  };

  editUser(user: User){
    this.setState({
      user: Object.assign({}, this.state.user, user),
    });
    console.log('User has been edited: ', user);
  }

  render() {
    return (
      <UserPreviewComponent
        user={ this.state.user }
        onEdit={ (user) => this.editUser(user) }
      />
    );
  }
}
```

# Default inputs


## ![angular] Angular

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'courses-list',
  templateUrl: './coursesList.component.html',
})
export class CoursesListController {
  displayPurchased: boolean = true;
  displayAvailable: boolean = true;
}
```

## ![react] React

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';

export class CoursesListController extends { Component } {
  static propTypes = {
    displayPurchased: PropTypes.bool,
    displayAvailable: PropTypes.bool,
  };

  static defaultProps = {
    displayPurchased: true,
    displayAvailable: true,
  };

  render() {
    return <div>{ /* template */ }</div>;
  }
}
```

:link: https://reactjs.org/docs/typechecking-with-proptypes.html#default-prop-values


# Lifecycle methods


### ![angular] Angular

#### [`ngOnChanges()`](https://angular.io/guide/lifecycle-hooks#onchanges)

Respond when Angular (re)sets data-bound input properties. The method receives a `SimpleChanges` object of current and previous property values.

Called before `ngOnInit()` and whenever one or more data-bound input properties change.

```ts
export class PeekABooComponent extends PeekABoo implements OnChanges {
  // only called for/if there is an @input variable set by parent.
  ngOnChanges(changes: SimpleChanges) {
    let changesMsgs: string[] = [];
    for (let propName in changes) {
      if (propName === 'name') {
        let name = changes['name'].currentValue;
        changesMsgs.push(`name ${this.verb} to "${name}"`);
      } else {
        changesMsgs.push(propName + ' ' + this.verb);
      }
    }
    this.logIt(`OnChanges: ${changesMsgs.join('; ')}`);
    this.verb = 'changed'; // next time it will be a change
  } 
}
```

#### [`ngOnInit()`](https://angular.io/guide/lifecycle-hooks#oninit)

Initialize the directive/component after Angular first displays the data-bound properties and sets the directive/component's input properties.

Called once, after the first `ngOnChanges()`.

```ts
export class PeekABoo implements OnInit {
  constructor(private logger: LoggerService) { }

  // implement OnInit's `ngOnInit` method
  ngOnInit() { this.logIt(`OnInit`); }

  logIt(msg: string) {
    this.logger.log(`#${nextId++} ${msg}`);
  }
}
```

#### [`ngDoCheck()`](https://angular.io/guide/lifecycle-hooks#docheck)

Detect and act upon changes that Angular can't or won't detect on its own.

Called during every change detection run, immediately after `ngOnChanges()` and `ngOnInit()`.

```ts
export class PeekABooComponent extends PeekABoo implements DoCheck {
  ngDoCheck() { 
    this.logIt(`DoCheck`);
  }
}
```

#### [`ngAfterContentInit()`](https://angular.io/guide/lifecycle-hooks#aftercontent-hooks)

Respond after Angular projects external content into the component's view.
Called once after the first `ngDoCheck()`.

```ts
export class PeekABooComponent extends PeekABoo implements AfterContentInit {
  ngAfterContentInit() { this.logIt(`AfterContentInit`);  }
}
```

#### [`ngAfterContentChecked()`](https://angular.io/guide/lifecycle-hooks#aftercontent-hooks)

Respond after Angular checks the content projected into the component.

Called after the `ngAfterContentInit()` and every subsequent `ngDoCheck()`

```ts
export class PeekABooComponent extends PeekABoo implements AfterContentChecked {
  // Beware! Called frequently!
  // Called in every change detection cycle anywhere on the page
  ngAfterContentChecked() { this.logIt(`AfterContentChecked`); }
}
```

#### [`ngAfterViewInit()`](https://angular.io/guide/lifecycle-hooks#afterview)

Respond after Angular initializes the component's views and child views.

Called once after the first `ngAfterContentChecked()`.

```ts
export class AfterViewComponent implements  AfterViewChecked, AfterViewInit {  
  ngAfterViewInit() {
    // viewChild is set after the view has been initialized
    this.logIt('AfterViewInit');
    this.doSomething();
  }
}
```

#### [`ngAfterViewChecked()`](https://angular.io/guide/lifecycle-hooks#afterview)

Respond after Angular checks the component's views and child views.

Called after the `ngAfterViewInit` and every subsequent `ngAfterContentChecked()`

```ts
export class AfterViewComponent implements  AfterViewChecked, AfterViewInit 
  ngAfterViewChecked() {
    // viewChild is updated after the view has been checked
    if (this.prevHero === this.viewChild.hero) {
      this.logIt('AfterViewChecked (no change)');
    } else {
      this.prevHero = this.viewChild.hero;
      this.logIt('AfterViewChecked');
      this.doSomething();
    }
  }
}
```

#### [`ngOnDestroy()`](https://angular.io/guide/lifecycle-hooks#ondestroy)

Cleanup just before Angular destroys the directive/component. Unsubscribe Observables and detach event handlers to avoid memory leaks.

Called just before Angular destroys the directive/component.

```ts
@Directive({
    selector: '[destroyDirective]'
})

export class OnDestroyDirective implements OnDestroy {
  sayHello: number;
  constructor() {
    this.sayHiya = window.setInterval(() => console.log('hello'),     1000);
  }
  ngOnDestroy() {
     window.clearInterval(this.sayHiya);
  }
}
```

## ![react] React

#### [`componentWillMount()`](https://reactjs.org/docs/react-component.html#componentwillmount) 

Is invoked just before rendering. Modifying the state here won't trigger a re-render.

#### [`componentDidMount()`](https://reactjs.org/docs/react-component.html#componentdidmount) 

Is invoked after render. Useful for the initialization that require DOM nodes.

#### [`componentWillReceiveProps(nextProps)`](https://reactjs.org/docs/react-component.html#componentwillreceiveprops) 

Is only called after rendering, but before receiving new props. Because React may call this method without props changing, it is recommended to manually implement a check to see if there's a difference.

#### [`shouldComponentUpdate(nextProps, nextState)`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 

This method is called before receiving new props or a change of state. By default, it returns true which means re-rendering is triggered by any change. Modifying this method allows you to only re-render in intended scenarios.

#### [`componentWillUpdate(nextProps, nextState)`](https://reactjs.org/docs/react-component.html#componentwillupdate) 

Is invoked whenever `shouldComponentUpdate` returns true before rendering. Note: You can't use `this.setState()` here.

#### [`componentDidUpdate(prevProps, prevState)`](https://reactjs.org/docs/react-component.html#componentdidupdate) 

Is invoked after rendering, but not after the initial render. This method is useful for manipulating the DOM when updated.

#### [`componentWillUnmount()`](https://reactjs.org/docs/react-component.html#componentwillunmount) 

Is invoked immediately before a component is unmounted and destroyed. Useful for resource cleanup.

#### [`componentDidCatch(error,info)`](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html) 

Is invoked when Javascript throws an error anywhere in the component's tree. Useful for catching errors, showing a fallback interface, and logging errors without breaking the entire application.


# Conditional rendering


## ![angular] Angular

Since Angular 4.0.0, alongside standard `ngIf`, it is possible to use `ngIf;else` or `ngIf;then;else` using `<ng-template>` with an alias `#aliasName`.

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'registration',
  template: '',
})
export class RegistrationComponent {
  registrationCompleted: boolean = false;
  displaySpecialOffer: boolean = false;
}
```

```html
<div *ngIf="displaySpecialOffer">
  <special-offer></special-offer>
</div>

<div *ngIf="registrationCompleted;else registrationForm">
  <registration-completed></registration-completed>
</div>

<ng-template #registrationForm>
  <registration-form></registration-form>
</ng-template>
```

:link: https://angular.io/api/common/NgIf

## ![react] React

The most common approach to conditional rendering is by using the ternary operator:
`{ condition ? <Component /> : null }`

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';

export class Registration extends Component {
  state = {
    registrationCompleted: false,
  };

  propTypes = {
    displaySpecialOffer: PropTypes.bool,
  }

  render() {
    return (
      <div>
        { this.props.displaySpecialOffer ? <SpecialOffer /> : null }

        { this.state.registrationCompleted ? (
          <RegistrationCompleted />
        ) : (
          <RegistrationForm />
        ) }
      </div>
    );
  }
}
```


# Lists


## ![angular] Angular

[ngFor](https://angular.io/guide/template-syntax#ngfor)

```ts
export interface Book {
  id: number;
  title: string;
  author: string;
}
```

```ts
import { Component } from '@angular/core';
import { Book } from './book.interface';

@Component({
  selector: 'book-list',
  template: `
    <ul>
      <li *ngFor="let book of books; trackBy: trackById">
        {{ book.title }} by {{ book.author }}
      </li>
    </ul>
  `
})
export class BookListComponent {
  this.books: Book[] = [
    {
      id: 1,
      title: "Eloquent JavaScript",
      author: "Marijn Haverbeke"
    },
    {
      id: 2,
      title: "JavaScript: The Good Parts",
      author: "Douglas Crockford"
    },
    {
      id: 3,
      title: "JavaScript: The Definitive Guide",
      author: "David Flanagan"
    }
  ];

  trackById(book) {
    return book.id;
  }
}
```

## ![react] React

[Lists and Keys](https://reactjs.org/docs/lists-and-keys.html)

```jsx
import React, { Component } from 'react';

export class BookList extends Component {
  state = {
    books: [
      {
        id: 1,
        title: 'Eloquent JavaScript',
        author: 'Marijn Haverbeke',
      },
      {
        id: 2,
        title: 'JavaScript: The Good Parts',
        author: 'Douglas Crockford',
      },
      {
        id: 3,
        title: 'JavaScript: The Definitive Guide',
        author: 'David Flanagan',
      },
    ],
  };

  render() {
    const { books } = this.state;

    return (
      <ul>
        { books.map(book => {
          return (
            <li key="{ book.id }">
              { book.title } by { book.author }
            </li>
          );
        }) }
      </ul>
    );
  }
}
```

# Filters


## ![angular] Angular

In Angular filters are called [pipes](https://angular.io/guide/pipes). The built-in pipes available in Angular are: DatePipe, UpperCasePipe, LowerCasePipe, CurrencyPipe, and PercentPipe.

Apart from the built-in pipes, you can create your own, custom pipes.

Create a custom pipe:
this pipe transforms a given URL to a safe style URL, that way it can be used in hyperlinks, for example <img src> or <iframe src>, etc..

```ts
import { Pipe, PipeTransform } from '@angular/core';
import { DomSanitizer} from '@angular/platform-browser';

@Pipe({ name: 'safe' })
export class SafePipe implements PipeTransform {
  constructor(public sanitizer: DomSanitizer) {}
  transform(url) {
    return this.sanitizer.bypassSecurityTrustResourceUrl(url);
  }
}
```

Use the custom pipe in a template:
The given `someUrl` is filtered through the `safe` pipe which transforms it trough the [`DomSanitizer`](https://angular.io/api/platform-browser/DomSanitizer) function `bypassSecurityTrustResourceUrl`.

```html
  <iframe [src]="someUrl | safe"></iframe>
```

Note: `[src]` above is an input to the component which 'lives' above the `iframe`.

:link: https://angular.io/guide/pipes

## ![react] React

React doesn't provide any specific filtering mechanism. This can simply be achieved by using ordinary JavaScript functions:

```jsx
export function reverse(input = '', uppercase = false) {
  const out = input.split('').reverse().join('');

  return uppercase ? out.toUpperCase() : out;
}
```

```jsx
import React, { Component } from 'react';
import { reverse } from 'utils';

export class App extends Component {
  render() {
    return (
      <div>
        { reverse(this.props.input) }
      </div>
    );
  }
}
```

Filter chaining can be achieved using function composition:

```jsx
<div>
  { truncate(reverse(this.props.input)) }
</div>;
```


# Child nodes


## ![angular] Angular

Angular provides two ways to deal with child nodes: `ViewChild` and `ContentChild`. They both have the same purpose, but there are different use cases for them.

* [`ViewChild`](https://angular.io/api/core/ViewChild) works with the **internal DOM of your component**, defined by you in the component's template. You have to use the `@ViewChild` decorator to get the DOM element reference.

* [`ContentChild`](https://angular.io/api/core/ContentChild) works with de **DOM supplied to your component by its end-user** (See [Transclusion and Containment](#transclusion-and-containment)). You have to use the `@ContentChild` decorator to get the DOM element reference.

```ts
import {
  Component,
  Input,
  ViewChild,
  ContentChild,
  AfterViewInit,
  AfterContentInit,
} from '@angular/core';

@Component({
  selector: 'child',
  template: `
    <p>Hello, I'm your child #{{ number }}!</p>
  `,
})
export class Child {
  @Input() number: number;
}

@Component({
  selector: 'parent',
  template: `
    <child number="1"></child>
    <ng-content></ng-content>
  `,
})
export class Parent implements AfterViewInit, AfterContentInit {
  @ViewChild(Child) viewChild: Child;
  @ContentChild(Child) contentChild: Child;

  ngAfterViewInit() {
    // ViewChild element is only available when the
    // ngAfterViewInit lifecycle hook is reached.
    console.log(this.viewChild);
  }

  ngAfterContentInit() {
    // ContentChild element is only available when the
    // ngAfterContentInit lifecycle hook is reached.
    console.log(this.contentChild);
  }
}

@Component({
  selector: 'app',
  template: `
    <parent>
      <child number="2"></child>
      <child number="3"></child>
    </parent>
  `,
})
export class AppComponent { }
```

`ViewChild` and `ContentChild` only work with a **single** DOM element. You can use [`ViewChildren`](https://angular.io/api/core/ViewChildren) and [`ContentChildren`](https://angular.io/api/core/ContentChildren) in order to get **multiple elements**. Both return the elements wrapped in a [`QueryList`](https://angular.io/api/core/QueryList).

## ![react] React

In React, we have two options to deal with child nodes: [`refs`](https://reactjs.org/docs/refs-and-the-dom) and [`children`](https://reactjs.org/docs/jsx-in-depth.html#children-in-jsx). With `refs`, you have access to the real DOM element. The `children` property lets you manipulate the underlying [React elements](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html).

#### refs

`ref` is a special attribute we can pass to a React element that receives a callback and call it with the corresponding DOM node.

```jsx
import React, { Component } from 'react';

// In order to access child nodes from parents, we can pass the `ref` callback
// to the children as props.
const TextInput = ({ inputRef }) => (
  <div>
    <input ref={inputRef} type="text" />
  </div>
);

class Parent extends Component {
  componentDidMount() {
    // Refs are only executed after mounting and unmounting. Now `this.textInput`
    // references a real DOM node. So, we can use the raw DOM API
    // (to focus the input, for example)
    this.textInput.focus();
  }

  render() {
    // The child's `inputRef` prop receives the `ref` callback.
    // We can use the callback to store the DOM element in an instance variable.
    return (
      <div>
        <label>This is my child: </label>
        <TextInput
          inputRef={node => { this.textInput = node; }} />
      </div>
    );
  }
}

```

#### children

`children` is a special prop available in all React component instances. You can use it to control _how_ and _where_ the underlying React elements will be rendered.

```jsx
import React, { Component } from 'react';

// children is just a prop. In this case, the value of `children` will be
// what you pass to the <Heading /> component as a child node.
const Heading = ({ children }) => (
  <h1 className="Heading">
    {children}
  </h1>
);

// `this.props.children` refers to whatever is a valid node inside the <Layout /> element.
class Layout extends Component {
  render() {
    return (
      <div class="Layout">
        {this.props.children}
      </div>
    );
  }
}

const App = () => (
  <div>
    <Heading>I am the child!</Heading>
    <Layout>
      We are
      {'the'}
      <strong>Children!</strong>
    </Layout>
  </div>
);
```

# Transclusion and Containment

## Basic

## ![angular] Angular

```ts
@Component({
  selector: 'layout',
  template: `
    <div>
      <ng-content></ng-content>
    </div>
  `,
})
export class Layout {}

@Component({
  selector: 'page-content',
  template: '<div>Some content</div>',
})
export class PageContent {}

@Component({
  selector: 'page-footer',
  template: '<footer>Some content</footer>',
})
export class PageFooter {}
```
```html
<layout>
  <page-content></page-content>
  <page-footer></page-footer>
</layout>
```

## ![react] React

```jsx
const Layout = ({ children, theme }) => (
  <div className={`theme-${theme}`}>
    {children}
  </div>
);

const PageContent = () => (
  <div>Some content</div>
);

const PageFooter = () => (
  <div>Some content</div>
);

<Layout theme='dark'>
  <PageContent />
  <PageFooter />
</Layout>;
```

## Multiple slots


## ![angular] Angular

> TODO

## ![react] React

```jsx
const Layout = ({ children, theme }) => (
  <div className={`theme-${theme}`}>
    <header>{children.header}</header>
    <main>{children.content}</main>
    <footer>{children.footer}</footer>
  </div>
);

const Header = () => (
  <h1>My Header</h1>
);

const Footer = () => (
  <div>My Footer</div>
);

const Content = () => (
  <div>Some fancy content</div>
);

<Layout theme='dark'>
  {{
    header: <Header />,
    content: <Content />,
    footer: <Footer />,
  }}
</Layout>;
```


# Class toggling


## ![angular] Angular

In Angular, the [ngClass](https://angular.io/guide/ajs-quick-reference#ngclass) directive works similarly. It includes/excludes CSS classes based on an expression.

```html
<main-header
  [ngClass]="{ 'mainNavbar--fluid': isFluid }">
...
</main-header>
```

## ![react] React

The React approach allows us to construct className string with JavaScript.

```jsx
<MainHeader
  className={ this.props.isFluid ? 'mainNavbar--fluid' : '' }
/>;
```

Many utility libraries have emerged - [classnames](https://github.com/JedWatson/classnames) being among the most popular.

```jsx
class App {
  /* ... */
  render() {
    const classNames = classNames({
      'mainNavbar--fluid': this.props.isFluid,
    });

    return (
      <MainHeader
        className={ classNames }
      />
    );
  }
}
```

# Data binding


## ![angular] Angular

We use [`[(ngModel)]`](https://angular.io/api/forms/NgModel) to have a two-way data binding inside our forms. The value in the UI will always be synced with the domain model in the component.

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'registration',
  templateUrl: require('registration.component.html'),
})
export class RegistrationComponent {
  name: string = '';
}
```

```html
<input [(ngModel)]="name" />
<p>Name: {{ name }}</p>
```

:link: https://angular.io/guide/template-syntax#two-way-binding

## ![react] React

> TODO


# Forms


## ![angular] Angular

Angular offers two ways to build forms:

* [Reactive forms](https://angular.io/guide/reactive-forms)
* [Template-driven forms](https://angular.io/guide/forms#template-driven-forms)

The former uses a reactive (or model-driven) approach to build forms. The latter allows you to build forms by writing templates in the Angular template syntax with the form-specific directives and techniques.

### Reactive forms example

```ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormBuilder } from '@angular/forms';

@Component({
  selector: 'reactive-form',
  template: `
    <div>
      <form [formGroup]="form"
            (ngSubmit)="onSubmit(form.value, form.valid)"
            novalidate>
        <div>
          <label>
            Name:
            <input type="text" formControlName="name">
          </label>
        </div>
        <div>
          <label>
            Email:
             <input type="email" formControlName="email">
          </label>
        </div>
      </form>
    </div>
  `
})

export class ReactiveFormComponent implements OnInit {
  public form: FormGroup;

  constructor(private formBuilder: FormBuilder) { }

  ngOnInit() {
    this.form = this.formBuilder.group({name: [''], email: ['']});
  }
}
```
#### Template-driven forms example

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'template-driven-form',
  template: `
    <div>
      <form (ngSubmit)="onSubmit()" #templateDrivenForm="ngForm" novalidate>
        <div>
          <label>
            Name:
            <input type="text" [(ngModel)]="model.name" required>
          </label>
        </div>
        <div>
          <label>
            Email:
            <input type="email" [(ngModel)]="model.email" required>
          </label>
        </div>
        <button type="submit" [disabled]="!templateDrivenForm.form.valid">Submit</button>
        </form>
    </div>
  `
})

export class TemplateDrivenFormComponent {
  public model = { name: '', email: '' };
}
```

The `novalidate` attribute in the `<form>` element prevents the browser from attempting native HTML validations.

## ![react] React

Two techniques exists in React to handle form data: [Controlled Components](https://reactjs.org/docs/forms.html#controlled-components) and [Uncontrolled Components](https://reactjs.org/docs/uncontrolled-components.html). A controlled component keeps the input's value in the state and updates it via `setState()`. While in an uncontrolled component, form data is handled by DOM itself and referenced via `ref`. In most cases, it is recommended to use controlled components.

```js
import React, { Component } from 'react';

export default class ReactForm extends Component {
  state = {
    email: '',
    password:'',
  };

  handleChange = ({ name, value}) => {
    if (name === 'email') {
      this.setState({ email: value });
    } else if (name === 'password') {
      this.setState({ password: value });
    }
  };

  render() {
    return (
      <form>
        <label>
          Email:
          <input
            name="email"
            type="email"
            value= {this.state.email }
            onChange={ this.handleChange }
          />
        </label>
        <label>
          Password:
          <input
            name="password"
            type="password"
            value={ this.state.password }
            onChange={ this.handleChange }
          />
        </label>
      </form>
    );
  }
}
```

:link: https://reactjs.org/docs/forms.html


# Styling

## ![angular] Angular

When defining Angular components you may also include the CSS styles that will go with the template. By default the styles will be compiled as shadow DOM, which basically means you don't need any namespacing strategy for CSS classes.

The [`ngStyle`](https://angular.io/api/common/NgStyle) directive allows you to set custom CSS styles dynamically.

```ts
@Component({
  selector: 'ng-header',
  template: require('./header.html'),
  styles: [require('./header.scss')],
})
export class HeaderComponent {
  headerStyles = {};

  constructor(private themeProvider: ThemeProvider) {}

  ngOnInit() {
    this.headerStyles.color = ThemeProvider.getTextPrimaryColor();
  }
}
```

```html
<h1 [ngStyle]="headerStyles"
    class="Header">
    Welcome
</h1>
```

```scss
.Header {
    font-weight: normal;
}
```

:link: https://angular.io/guide/component-styles

## ![react] React

In the React community there are many approaches to styling your app, ranging from traditional preprocessors (like in the Angular world) to so-called [CSS in JS](https://github.com/MicheleBertoli/css-in-js). The most popular include:

* [css-modules](https://github.com/gajus/react-css-modules)
* [styled-components](https://github.com/styled-components/styled-components)
* [aphrodite](https://github.com/Khan/aphrodite)

To dynamically apply styles you can directly pass an object to the [style](https://reactjs.org/docs/dom-elements.html#style) attribute.

```jsx
export class HeaderComponent {
  state = {
    color: null,
  };

  componentDidMount() {
    this.setState({
      color: ThemeProvider.getTextPrimaryColor(),
    });
  }

  render() {
    return (
      <h1 styles={ { color: this.state.color } }>
        Welcome
      </h1>
    );
  }
}
```



# Inject HTML template

Also known as innerHTML.

## ![angular] Angular

The values are automatically sanitized before displaying them using [DomSanitizer](https://angular.io/docs/ts/latest/api/platform-browser/index/DomSanitizer-class.html).

```html
<p [innerHTML]="article.content"></p>
```

## ![react] React

All string values are sanitized before being inserted into the DOM. No more details are currently available.
You need to pass an object containing the `__html` property with the desired template contents.

```jsx
<p dangerouslySetInnerHTML={ { __html: article.content } } />;
```
