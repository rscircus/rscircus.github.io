---
layout: post
title:  "Render objects based on their type in TypeScript"
sub_title: "Rendering things upon passing things"
excerpt_separator:  <!-- more -->
categories:
  - Code
tags:
  - TypeScript
  - Frontend
  - Typing
  - Web
---

Recently, my colleague T — a big fan of statically typed languages and working with [Angular](angular.io) and [TypeScript](www.typescriptlang.org) — came over and asked a simple question: *"Hey R, do you have an idea how to realize Reflection in TypeScript? Actually, I need to render a site a little bit differentlty based upon a set of passed types."*. Naturally, Reflection sounds like a good idea. But you know... JavaScript. 🤦‍ That's what TypeScript is transpiled into.

Having worked with TypeScript and React for a while, I suggested a few ideas from the top of my had head but soon discovered a few limits. Therefore, let's see if we can come up with a solution to this problem using TypeScript's inherent features.

<!-- more --->

At the beginning of my journey somebody told me: *"Do you know TypeScript? It's JavaScript with documentation."*. I was hooked and went down the modern rabbit hole of Frontend Development. Lately, I discovered this:

![](https://rscircus.github.io/assets/img/20200206_TsAngularSpec.png)

🤔 - I'm not sure if things have changed... So here comes a solution based on the various sources referenced at the bottom.


## Frontend

Being a big fan of first principles we start at the bottom:

```bash
mkdir inferType
cd inferType
yarn init -y
```

Because yarn is more focused on stability than npm. Let's follow with some simple content:

```bash
touch index.html
mkdir src
touch src/index.ts
```

and we'll fill `index.html` with this:

```html
<html>

  <head>
    <title>Playing with types</title>
  </head>

  <body>
    <div id="root" />
    <script src="src/index.ts"></script>
  </body>

</html>
```

and `index.ts` with this:

```typescript
console.log("Hello, world!")
```

Now our tools:

```bash
yarn add typescript
yarn add parcel-bundler
```

We can work with TypeScript's default settings. Finally, a `package.json` should have been there since `yarn init -y`. Let's create some `scripts` inside it:

```json
{
  "name": "inferType",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "parcel index.html --open",
    "build": "parcel build index.html"
  },
  "dependencies": {
    "parcel-bundler": "^1.12.4",
    "typescript": "^3.7.5"
  }
}
```

If we perform a

```bash
yarn start
```

parcel will start serving at `localhost:1234`.

Finally, things should look like this:

![](https://rscircus.github.io/assets/img/20200208_TsAngularParcel.png)

We keep it running for the rest of this article, as it supports hot module replacement. Speaking of hot module replacement, let's make ourselves a fresh brew of ☕️ on this Saturday early morning. 😊.

### Types and Interfaces

Having laid out all tools, we can now dig into TypeScript to achieve our loosely stated goal of rendering incoming objects depending upon the incoming type. These objects will somehow be similar, so we'll use a button, which will be rendered slightly different for this.

Let's create our types in `src/buttonTypes.ts` and start with a very simple button:

```typescript
export interface iButton {
  type: ButtonType
  value?: string
  action?: ButtonAction
}
```

There is only one mandatory parameter: `type`. Let's define it in an unnecessarily complex way to illustrate the union better later on:

```typescript
type ButtonTypeMain = typeof buttonTypeNormal | typeof buttonTypeGrey
export type ButtonType =
  | typeof buttonTypeWarning
  | ButtonTypeMain
```

which is a union type and uses these `const`s to later differentiate our UI:

```typescript
export const buttonTypeNormal = "normal"
export const buttonTypeGrey = "grey"
export const buttonTypeWarning = "warning"
```

Further we need a function type to handle the `action`:

```typescript
export type ButtonAction = () => void
```


OK. This looks robust enough. Let's extend this interface and export the union of these as singular value and array:

```typescript
export interface iButtonNormal extends iButton {
  type: typeof buttonTypeNormal
}

export interface iButtonGrey extends iButton {
  type: typeof buttonTypeGrey
  youropt?: boolean
}

export interface iButtonWarning extends iButton {
  type: typeof buttonTypeWarning
  handleSomeKey?: string
}

type ButtonUnion =
  | ButtonTypeMain
  | iButtonNormal
  | iButtonGrey
  | iButtonWarning

export type Button = ButtonUnion | ButtonUnion[]
```

Here you can see, that we extended `iButton` in `iButtonGrey` with optional parameters. We have to export quite a bit, as we'll build a parser in a moment. Note, however, that the Union type is hidden in `Button` and even gives us the opportunity to digest arrays *and* singular instances.

## Application

At this point in time our `src/buttonTypes.ts` should look OK and should not throw any error if you run a linter over it. Next, we will look at an application of such a convoluted button. Let's open our `src/index.ts` and replace its single line of content with this:

```typescript
import { Button } from "./buttonTypes"

const buttons: Button = [
  "normal",
  {
    type: "warning",
    action: () => alert("WARNING!")
  },
  {
    type: "grey",
    value: "Ignore me."
  }
]

// Create a view
const layout = parse(buttons)
const root = document.getElementById("root")
layout.forEach(element => root.append(element))
```

which hopefully shows you this error in your browser (as parcel is still running):

![](https://rscircus.github.io/assets/img/20200208_TsAngularError.png)

We ignore it for now.

Let's look with awe at where we are heading to. We defined an array of Buttons, which are in fact various simple JavaScript objects. However, TypeScript accepts these to be Buttons as their structure matches the types buried inside the Button type and TypeScript has a structural type system.

Now let's convert this array of Buttons into something which could be an user interface using the `parse` function. We create a parser for this inside `src/parse.ts`:

```typescript
import {
  Button,
  buttonTypeWarning,
  buttonTypeNormal,
  ButtonAction,
  buttonTypeGrey }
from "./buttonTypes"

// Give button a label
function title(type: string, label?: string): string {

  if (typeof label === "string" && label) return label
  if (type === buttonTypeNormal) return "OK"
  if (type === buttonTypeWarning) return "Warning!"
}

// Handle button click
function click(type: string, action?: ButtonAction): ButtonAction {

  if (typeof action !== "function") {
    if (type === buttonTypeNormal) return () => console.log("OK clicked.")
    if (type === buttonTypeGrey) return () => console.log("Grey clicked.")
    if (type === buttonTypeWarning) return () => console.log("Warning clicked.")
  }

  return action
}

// Create the button - probably your framework does this for you,
// but we do it all by hand, because we can.
function create(
  type: string,
  label?: string,
  action?: ButtonAction
): HTMLButtonElement {

  const button = document.createElement("button")

  button.innerText = title(type, label)
  button.onclick = click(type, action)

  return button
}

// Assemble and publish the parser
export function parse(buttons: Button): HTMLButtonElement[] {

  const htmlButtons: HTMLButtonElement[] = []

  if (buttons instanceof Array) {
    buttons.forEach(button => {
      if (typeof button === "string") {
        htmlButtons.push(create(button))
      }
      if (typeof button === "object" && button.type) {
        htmlButtons.push(
          create(button.type, button.value, button.action)
        );
      }
    })
  }

  // TODO: Handle a 'singleton'

  return htmlButtons
}
```

after importing it in `index.ts`, the overall result should display us:

![](https://rscircus.github.io/assets/img/20200108_TsAngularButtons.png)

Great success! And one exercise left for you. 😉

## Conclusion

We used TypeScript and Parcel to create a very simple WebApp without any framework to demonstrate how we can handle multiple simple objects and leverage TypeScripts structural typing system to parse these objects and represent them as DOM objects. We didn't cover styling at all and do this another day. 😊



## Sources

- https://typescriptlang.org
- https://javascript.info/proxy
- https://dev.to/saltyshiomix/the-flexible-type-power-of-typescript-fp7
- https://dev.to/peacefullatom/introduction-angular-13pn
- https://parceljs.org
