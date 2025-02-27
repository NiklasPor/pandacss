# @pandacss/generator

## 0.15.5

### Patch Changes

- d12aed2b: Fix issue where unused recipes and slot recipes doesn't get treeshaken properly
- 909fcbe8: - Fix issue with `Promise.all` where it aborts premature ine weird events. Switched to `Promise.allSettled`
- 3d5971e5: - **Vue**: Fix issue where elements created from styled factory does not forward DOM attributes and events
  to the underlying element.
  - **Vue**: Fix regression in generated types
  - **Preact**: Fix regression in generated types
  - @pandacss/core@0.15.5
  - @pandacss/is-valid-prop@0.15.5
  - @pandacss/logger@0.15.5
  - @pandacss/shared@0.15.5
  - @pandacss/token-dictionary@0.15.5
  - @pandacss/types@0.15.5

## 0.15.4

### Patch Changes

- bf0e6a30: Fix issues with class merging in the `styled` factory fn for Qwik, Solid and Vue.
- 69699ba4: Improved styled factory by adding a 3rd (optional) argument:

  ```ts
  interface FactoryOptions<TProps extends Dict> {
    dataAttr?: boolean
    defaultProps?: TProps
    shouldForwardProp?(prop: string, variantKeys: string[]): boolean
  }
  ```

  - Setting `dataAttr` to true will add a `data-recipe="{recipeName}"` attribute to the element with the recipe name.
    This is useful for testing and debugging.

  ```jsx
  import { styled } from '../styled-system/jsx'
  import { button } from '../styled-system/recipes'

  const Button = styled('button', button, { dataAttr: true })

  const App = () => (
    <Button variant="secondary" mt="10px">
      Button
    </Button>
  )
  // Will render something like <button data-recipe="button" class="btn btn--variant_purple mt_10px">Button</button>
  ```

  - `defaultProps` allows you to skip writing wrapper components just to set a few props. It also allows you to locally
    override the default variants or base styles of a recipe.

  ```jsx
  import { styled } from '../styled-system/jsx'
  import { button } from '../styled-system/recipes'

  const Button = styled('button', button, {
    defaultProps: {
      variant: 'secondary',
      px: '10px',
    },
  })

  const App = () => <Button>Button</Button>
  // Will render something like <button class="btn btn--variant_secondary px_10px">Button</button>
  ```

  - `shouldForwardProp` allows you to customize which props are forwarded to the underlying element. By default, all
    props except recipe variants and style props are forwarded.

  ```jsx
  import { styled } from '../styled-system/jsx'
  import { button } from '../styled-system/recipes'
  import { isCssProperty } from '../styled-system/jsx'
  import { motion, isValidMotionProp } from 'framer-motion'

  const StyledMotion = styled(
    motion.div,
    {},
    {
      shouldForwardProp: (prop, variantKeys) =>
        isValidMotionProp(prop) || (!variantKeys.includes(prop) && !isCssProperty(prop)),
    },
  )
  ```

  - @pandacss/types@0.15.4
  - @pandacss/core@0.15.4
  - @pandacss/is-valid-prop@0.15.4
  - @pandacss/logger@0.15.4
  - @pandacss/shared@0.15.4
  - @pandacss/token-dictionary@0.15.4

## 0.15.3

### Patch Changes

- d34c8b48: Fix issue where HMR does not work for Vue JSX factory and patterns
- 1ac2011b: Add a new `config.importMap` option that allows you to specify a custom module specifier to import from
  instead of being tied to the `outdir`

  You can now do things like leverage the native package.json
  [`imports`](https://nodejs.org/api/packages.html#subpath-imports):

  ```ts
  export default defineConfig({
    outdir: './outdir',
    importMap: {
      css: '#panda/styled-system/css',
      recipes: '#panda/styled-system/recipes',
      patterns: '#panda/styled-system/patterns',
      jsx: '#panda/styled-system/jsx',
    },
  })
  ```

  Or you could also make your outdir an actual package from your monorepo:

  ```ts
  export default defineConfig({
    outdir: '../packages/styled-system',
    importMap: {
      css: '@monorepo/styled-system',
      recipes: '@monorepo/styled-system',
      patterns: '@monorepo/styled-system',
      jsx: '@monorepo/styled-system',
    },
  })
  ```

  Working with tsconfig paths aliases is easy:

  ```ts
  export default defineConfig({
    outdir: 'styled-system',
    importMap: {
      css: 'styled-system/css',
      recipes: 'styled-system/recipes',
      patterns: 'styled-system/patterns',
      jsx: 'styled-system/jsx',
    },
  })
  ```

- 1eb31118: Automatically allow overriding config recipe compoundVariants styles within the `styled` JSX factory,
  example below

  With this config recipe:

  ```ts file="panda.config.ts"
  const button = defineRecipe({
    className: 'btn',
    base: { color: 'green', fontSize: '16px' },
    variants: {
      size: { small: { fontSize: '14px' } },
    },
    compoundVariants: [{ size: 'small', css: { color: 'blue' } }],
  })
  ```

  This would previously not merge the `color` property overrides, but now it does:

  ```tsx file="example.tsx"
  import { styled } from '../styled-system/jsx'
  import { button } from '../styled-system/recipes'

  const Button = styled('button', button)

  function App() {
    return (
      <>
        <Button size="small" color="red.100">
          Click me
        </Button>
      </>
    )
  }
  ```

  - Before: `btn btn--size_small text_blue text_red.100`
  - After: `btn btn--size_small text_red.100`

- Updated dependencies [95b06bb1]
- Updated dependencies [1ac2011b]
- Updated dependencies [58743bc4]
  - @pandacss/shared@0.15.3
  - @pandacss/core@0.15.3
  - @pandacss/types@0.15.3
  - @pandacss/token-dictionary@0.15.3
  - @pandacss/is-valid-prop@0.15.3
  - @pandacss/logger@0.15.3

## 0.15.2

### Patch Changes

- 6d15776c: When bundling the `outdir` in a library, you usually want to generate type declaration files (`d.ts`).
  Sometimes TS will complain about types not being exported.

  - Export all types from `{outdir}/types/index.d.ts`, this fixes errors looking like this:

  ```
  src/components/Checkbox/index.tsx(8,7): error TS2742: The inferred type of 'Root' cannot be named without a reference to '../../../node_modules/@acmeorg/styled-system/types/system-types'. This is likely not portable. A type annotation is necessary.
  src/components/Checkbox/index.tsx(8,7): error TS2742: The inferred type of 'Root' cannot be named without a reference to '../../../node_modules/@acmeorg/styled-system/types/csstype'. This is likely not portable. A type annotation is necessary.
  src/components/Checkbox/index.tsx(8,7): error TS2742: The inferred type of 'Root' cannot be named without a reference to '../../../node_modules/@acmeorg/styled-system/types/conditions'. This is likely not portable. A type annotation is necessary.
  ```

  - Export generated recipe interfaces from `{outdir}/recipes/{recipeFn}.d.ts`, this fixes errors looking like this:

  ```
  src/ui/avatar.tsx (16:318) "AvatarRecipe" is not exported by "styled-system/recipes/index.d.ts", imported by "src/ui/avatar.tsx".
  src/ui/card.tsx (2:164) "CardRecipe" is not exported by "styled-system/recipes/index.d.ts", imported by "src/ui/card.tsx".
  src/ui/checkbox.tsx (19:310) "CheckboxRecipe" is not exported by "styled-system/recipes/index.d.ts", imported by "src/ui/checkbox.tsx".
  ```

  - Export type `ComponentProps` from `{outdir}/types/jsx.d.ts`, this fixes errors looking like this:

  ```
   "ComponentProps" is not exported by "styled-system/types/jsx.d.ts", imported by "src/ui/form-control.tsx".
  ```

- 26a788c0: - Switch to interface for runtime types
  - Create custom partial types for each config object property
- Updated dependencies [26a788c0]
  - @pandacss/types@0.15.2
  - @pandacss/core@0.15.2
  - @pandacss/token-dictionary@0.15.2
  - @pandacss/is-valid-prop@0.15.2
  - @pandacss/logger@0.15.2
  - @pandacss/shared@0.15.2

## 0.15.1

### Patch Changes

- 7e8bcb03: Fix an issue when wrapping a component with `styled` would display its name as `styled.[object Object]`
- 433f88cd: Fix issue in css reset where number input field spinner still show.
- 7499bbd2: Add the property `-moz-osx-font-smoothing: grayscale;` to the `reset.css` under the `html` selector.
- Updated dependencies [848936e0]
- Updated dependencies [26f6982c]
- Updated dependencies [4e003bfb]
  - @pandacss/core@0.15.1
  - @pandacss/shared@0.15.1
  - @pandacss/token-dictionary@0.15.1
  - @pandacss/types@0.15.1
  - @pandacss/is-valid-prop@0.15.1
  - @pandacss/logger@0.15.1

## 0.15.0

### Patch Changes

- 9f429d35: Fix issue where slot recipe did not apply rules when variant name has the same key as a slot
- 93d9ee7e: Refactor: Prefer `NativeElements` type for vue jsx elements
- 35793d85: Fix issue with cva when using compoundVariants and not passing any variants in the usage (ex: `button()`
  with `const button = cva({ ... })`)
- 39298609: Make the types suggestion faster (updated `DeepPartial`)
- f27146d6: Fix an issue where some JSX components wouldn't get matched to their corresponding recipes/patterns when
  using `Regex` in the `jsx` field of a config, resulting in some style props missing.

  issue: https://github.com/chakra-ui/panda/issues/1315

- Updated dependencies [4bc515ea]
- Updated dependencies [9f429d35]
- Updated dependencies [bc3b077d]
- Updated dependencies [39298609]
- Updated dependencies [dd47b6e6]
- Updated dependencies [f27146d6]
  - @pandacss/types@0.15.0
  - @pandacss/shared@0.15.0
  - @pandacss/core@0.15.0
  - @pandacss/token-dictionary@0.15.0
  - @pandacss/is-valid-prop@0.15.0
  - @pandacss/logger@0.15.0

## 0.14.0

### Patch Changes

- bdd30d18: Fix issue where `pattern.raw(...)` did not share the same signature as `pattern(...)`
- bff17df2: Add each condition raw value information on hover using JSDoc annotation
- 6548f4f7: Add missing types (`StyledComponents`, `RecipeConfig`, `PatternConfig` etc) to solve a TypeScript issue (The
  inferred type of xxx cannot be named without a reference...) when generating declaration files in addition to using
  `emitPackage: true`
- 6f7ee198: Add `{svaFn}.raw` function to get raw styles and allow reusable components with style overrides, just like
  with `{cvaFn}.raw`
- 623e321f: Fix `config.strictTokens: true` issue where some properties would still allow arbitrary values
- 542d1ebc: Change the typings for the `css(...args)` function so that you can pass possibly undefined values to it.

  This is mostly intended for component props that have optional values like `cssProps?: SystemStyleObject` and would
  use it like `css({ ... }, cssProps)`

- 39b20797: Change the `css.raw` function signature to match the one from
  [`css()`](https://github.com/chakra-ui/panda/pull/1264), to allow passing multiple style objects that will be smartly
  merged.
- Updated dependencies [b1c31fdd]
- Updated dependencies [8106b411]
- Updated dependencies [9e799554]
- Updated dependencies [e6459a59]
- Updated dependencies [6f7ee198]
- Updated dependencies [623e321f]
- Updated dependencies [02161d41]
  - @pandacss/token-dictionary@0.14.0
  - @pandacss/types@0.14.0
  - @pandacss/core@0.14.0
  - @pandacss/is-valid-prop@0.14.0
  - @pandacss/logger@0.14.0
  - @pandacss/shared@0.14.0

## 0.13.1

### Patch Changes

- a5d7d514: Add `forceConsistentTypeExtension` config option for enforcing consistent file extension for emitted type
  definition files. This is useful for projects that use `moduleResolution: node16` which requires explicit file
  extensions in imports/exports.

  > If set to `true` and `outExtension` is set to `mjs`, the generated typescript `.d.ts` files will have the extension
  > `.d.mts`.

- 192d5e49: Fix issue where `cva` is undefined in preact styled factory
  - @pandacss/core@0.13.1
  - @pandacss/is-valid-prop@0.13.1
  - @pandacss/logger@0.13.1
  - @pandacss/shared@0.13.1
  - @pandacss/token-dictionary@0.13.1
  - @pandacss/types@0.13.1

## 0.13.0

### Patch Changes

- a9690110: Fix issue where `defineTextStyle` and `defineLayerStyle` return types are incompatible with `config.theme`
  type.
- 32ceac3f: Fix an issue with custom JSX components not finding their matching patterns
- Updated dependencies [04b5fd6c]
  - @pandacss/core@0.13.0
  - @pandacss/is-valid-prop@0.13.0
  - @pandacss/logger@0.13.0
  - @pandacss/shared@0.13.0
  - @pandacss/token-dictionary@0.13.0
  - @pandacss/types@0.13.0

## 0.12.2

### Patch Changes

- 6588c8e0: - Change the `css` function signature to allow passing multiple style objects that will be smartly merged.

  - Rename the `{cvaFn}.resolve` function to `{cva}.raw` for API consistency.
  - Change the behaviour of `{patternFn}.raw` to return the resulting `SystemStyleObject` instead of the arguments
    passed in. This is to allow the `css` function to merge the styles correctly.

  ```tsx
  import { css } from '../styled-system/css'
  css({ mx: '3', paddingTop: '4' }, { mx: '10', pt: '6' }) // => mx_10 pt_6
  ```

  > ⚠️ This approach should be preferred for merging styles over the current `cx` function, which will be reverted to
  > its original classname concatenation behaviour.

  ```diff
  import { css, cx } from '../styled-system/css'

  const App = () => {
    return (
      <>
  -      <div className={cx(css({ mx: '3', paddingTop: '4' }), css({ mx: '10', pt: '6' }))}>
  +      <div className={css({ mx: '3', paddingTop: '4' }, { mx: '10', pt: '6' })}>
          Will result in `class="mx_10 pt_6"`
        </div>
      </>
    )
  }
  ```

  To design a component that supports style overrides, you can now provide the `css` prop as a style object, and it'll
  be merged correctly.

  ```tsx title="src/components/Button.tsx"
  import { css } from '../../styled-system/css'

  export const Button = ({ css: cssProp = {}, children }) => {
    const className = css({ display: 'flex', alignItem: 'center', color: 'black' }, cssProp)
    return <button className={className}>{children}</button>
  }
  ```

  Then you can use the `Button` component like this:

  ```tsx title="src/app/page.tsx"
  import { css } from '../../styled-system/css'
  import { Button, Thingy } from './Button'

  export default function Page() {
    return (
      <Button css={{ color: 'pink', _hover: { color: 'red' } }}>
        will result in `class="d_flex items_center text_pink hover:text_red"`
      </Button>
    )
  }
  ```

  ***

  You can use this approach as well with the new `{cvaFn}.raw` and `{patternFn}.raw` functions, will allow style objects
  to be merged as expected in any situation.

  **Pattern Example:**

  ```tsx title="src/components/Button.tsx"
  import { hstack } from '../../styled-system/patterns'
  import { css, cva } from '../../styled-system/css'

  export const Button = ({ css: cssProp = {}, children }) => {
    // using the flex pattern
    const hstackProps = hstack.raw({
      border: '1px solid',
      _hover: { color: 'blue.400' },
    })

    // merging the styles
    const className = css(hstackProps, cssProp)

    return <button className={className}>{children}</button>
  }
  ```

  **CVA Example:**

  ```tsx title="src/components/Button.tsx"
  import { css, cva } from '../../styled-system/css'

  const buttonRecipe = cva({
    base: { display: 'flex', fontSize: 'lg' },
    variants: {
      variant: {
        primary: { color: 'white', backgroundColor: 'blue.500' },
      },
    },
  })

  export const Button = ({ css: cssProp = {}, children }) => {
    const className = css(
      // using the button recipe
      buttonRecipe.raw({ variant: 'primary' }),

      // adding style overrides (internal)
      { _hover: { color: 'blue.400' } },

      // adding style overrides (external)
      cssProp,
    )

    return <button className={className}>{props.children}</button>
  }
  ```

- 36fdff89: Fix bug in generated js code for atomic slot recipe produce where `splitVariantProps` didn't work without
  the first slot key.
  - @pandacss/core@0.12.2
  - @pandacss/is-valid-prop@0.12.2
  - @pandacss/logger@0.12.2
  - @pandacss/shared@0.12.2
  - @pandacss/token-dictionary@0.12.2
  - @pandacss/types@0.12.2

## 0.12.1

### Patch Changes

- 599fbc1a: Fix issue where `AnimationName` type was generated wrongly if keyframes were not resolved
  - @pandacss/core@0.12.1
  - @pandacss/is-valid-prop@0.12.1
  - @pandacss/logger@0.12.1
  - @pandacss/shared@0.12.1
  - @pandacss/token-dictionary@0.12.1
  - @pandacss/types@0.12.1

## 0.12.0

### Patch Changes

- a41515de: Fix issue where styled factory does not respect union prop types like `type Props = AProps | BProps`
- bf2ff391: Add `animationName` utility
- ad1518b8: fix failed styled component for solid-js when using recipe
  - @pandacss/core@0.12.0
  - @pandacss/token-dictionary@0.12.0
  - @pandacss/is-valid-prop@0.12.0
  - @pandacss/logger@0.12.0
  - @pandacss/shared@0.12.0
  - @pandacss/types@0.12.0

## 0.11.1

### Patch Changes

- c07e1beb: Make the `cx` smarter by merging and deduplicating the styles passed in

  Example:

  ```tsx
  <h1 className={cx(css({ mx: '3', paddingTop: '4' }), css({ mx: '10', pt: '6' }))}>Will result in "mx_10 pt_6"</h1>
  ```

- dfb3f85f: Add missing svg props types
- 23b516f4: Make layers customizable
- Updated dependencies [c07e1beb]
- Updated dependencies [dfb3f85f]
- Updated dependencies [23b516f4]
  - @pandacss/shared@0.11.1
  - @pandacss/is-valid-prop@0.11.1
  - @pandacss/types@0.11.1
  - @pandacss/core@0.11.1
  - @pandacss/token-dictionary@0.11.1
  - @pandacss/logger@0.11.1

## 0.11.0

### Patch Changes

- 5b95caf5: Add a hook call when the final `styles.css` content has been generated, remove cyclic (from an unused hook)
  dependency
- 39b80b49: Fix an issue with the runtime className generation when using an utility that maps to multiple shorthands
- 1dc788bd: Fix issue where some style properties shows TS error when using `!important`
- Updated dependencies [5b95caf5]
  - @pandacss/types@0.11.0
  - @pandacss/core@0.11.0
  - @pandacss/token-dictionary@0.11.0
  - @pandacss/is-valid-prop@0.11.0
  - @pandacss/logger@0.11.0
  - @pandacss/shared@0.11.0

## 0.10.0

### Minor Changes

- a669f4d5: Introduce new slot recipe features.

  Slot recipes are useful for styling composite or multi-part components easily.

  - `sva`: the slot recipe version of `cva`
  - `defineSlotRecipe`: the slot recipe version of `defineRecipe`

  **Definition**

  ```jsx
  import { sva } from 'styled-system/css'

  const button = sva({
    slots: ['label', 'icon'],
    base: {
      label: { color: 'red', textDecoration: 'underline' },
    },
    variants: {
      rounded: {
        true: {},
      },
      size: {
        sm: {
          label: { fontSize: 'sm' },
          icon: { fontSize: 'sm' },
        },
        lg: {
          label: { fontSize: 'lg' },
          icon: { fontSize: 'lg', color: 'pink' },
        },
      },
    },
    defaultVariants: {
      size: 'sm',
    },
  })
  ```

  **Usage**

  ```jsx
  export function App() {
    const btnClass = button({ size: 'lg', rounded: true })

    return (
      <button>
        <p class={btnClass.label}> Label</p>
        <p class={btnClass.icon}> Icon</p>
      </button>
    )
  }
  ```

### Patch Changes

- 24e783b3: Reduce the overall `outdir` size, introduce the new config `jsxStyleProps` option to disable style props and
  further reduce it.

  `config.jsxStyleProps`:

  - When set to 'all', all style props are allowed.
  - When set to 'minimal', only the `css` prop is allowed.
  - When set to 'none', no style props are allowed and therefore the `jsxFactory` will not be usable as a component:
    - `<styled.div />` and `styled("div")` aren't valid
    - but the recipe usage is still valid `styled("div", { base: { color: "red.300" }, variants: { ...} })`

- 2d2a42da: Fix staticCss recipe generation when a recipe didnt have `variants`, only a `base`
- 386e5098: Update `RecipeVariantProps` to support slot recipes
- 6d4eaa68: Refactor code
- Updated dependencies [24e783b3]
- Updated dependencies [9d4aa918]
- Updated dependencies [2d2a42da]
- Updated dependencies [386e5098]
- Updated dependencies [6d4eaa68]
- Updated dependencies [a669f4d5]
  - @pandacss/is-valid-prop@0.10.0
  - @pandacss/shared@0.10.0
  - @pandacss/types@0.10.0
  - @pandacss/token-dictionary@0.10.0
  - @pandacss/core@0.10.0
  - @pandacss/logger@0.10.0

## 0.9.0

### Minor Changes

- c08de87f: ### Breaking

  - Renamed the `name` property of a config recipe to `className`. This is to ensure API consistency and express the
    intent of the property more clearly.

  ```diff
  export const buttonRecipe = defineRecipe({
  -  name: 'button',
  +  className: 'button',
    // ...
  })
  ```

  - Renamed the `jsx` property of a pattern to `jsxName`.

  ```diff
  const hstack = definePattern({
  -  jsx: 'HStack',
  +  jsxName: 'HStack',
    // ...
  })
  ```

  ### Feature

  Update the `jsx` property to be used for advanced tracking of custom pattern components.

  ```jsx
  import { Circle } from 'styled-system/jsx'
  const CustomCircle = ({ children, ...props }) => {
    return <Circle {...props}>{children}</Circle>
  }
  ```

  To track the `CustomCircle` component, you can now use the `jsx` property.

  ```js
  import { defineConfig } from '@pandacss/dev'

  export default defineConfig({
    patterns: {
      extend: {
        circle: {
          jsx: ['CustomCircle'],
        },
      },
    },
  })
  ```

### Patch Changes

- Updated dependencies [c08de87f]
  - @pandacss/types@0.9.0
  - @pandacss/core@0.9.0
  - @pandacss/token-dictionary@0.9.0
  - @pandacss/is-valid-prop@0.9.0
  - @pandacss/logger@0.9.0
  - @pandacss/shared@0.9.0

## 0.8.0

### Minor Changes

- 9ddf258b: Introduce the new `{fn}.raw` method that allows for a super flexible usage and extraction :tada: :

  ```tsx
  <Button rootProps={css.raw({ bg: "red.400" })} />

  // recipe in storybook
  export const Funky: Story = {
  	args: button.raw({
  		visual: "funky",
  		shape: "circle",
  		size: "sm",
  	}),
  };

  // mixed with pattern
  const stackProps = {
    sm: stack.raw({ direction: "column" }),
    md: stack.raw({ direction: "row" })
  }

  stack(stackProps[props.size]))
  ```

### Patch Changes

- 3f1e7e32: Adds the `{recipe}.raw()` in generated runtime
- ac078416: Fix issue with extracting nested tokens as color-palette. Fix issue with extracting shadow array as a
  separate unnamed block for the custom dark condition.
- be0ad578: Fix parser issue with TS path mappings
- b75905d8: Improve generated react jsx types to remove legacy ref. This fixes type composition issues.
- 0520ba83: Refactor generated recipe js code
- 156b6bde: Fix issue where generated package json does not respect `outExtension` when `emitPackage` is `true`
- Updated dependencies [fb449016]
- Updated dependencies [ac078416]
- Updated dependencies [be0ad578]
  - @pandacss/core@0.8.0
  - @pandacss/token-dictionary@0.8.0
  - @pandacss/types@0.8.0
  - @pandacss/is-valid-prop@0.8.0
  - @pandacss/logger@0.8.0
  - @pandacss/shared@0.8.0

## 0.7.0

### Patch Changes

- a9c189b7: Fix issue where `splitVariantProps` in cva doesn't resolve the correct types
- Updated dependencies [f59154fb]
- Updated dependencies [a9c189b7]
  - @pandacss/shared@0.7.0
  - @pandacss/types@0.7.0
  - @pandacss/core@0.7.0
  - @pandacss/token-dictionary@0.7.0
  - @pandacss/is-valid-prop@0.7.0
  - @pandacss/logger@0.7.0

## 0.6.0

### Patch Changes

- cd912f35: Fix `definePattern` module overriden type, was missing an `extends` constraint which lead to a type error:

  ```
  styled-system/types/global.d.ts:14:58 - error TS2344: Type 'T' does not satisfy the constraint 'PatternProperties'.

  14   export function definePattern<T>(config: PatternConfig<T>): PatternConfig
                                                              ~

    styled-system/types/global.d.ts:14:33
      14   export function definePattern<T>(config: PatternConfig<T>): PatternConfig
                                         ~
      This type parameter might need an `extends PatternProperties` constraint.

  ```

- dc4e80f7: Export `isCssProperty` helper function from styled-system/jsx
- 5bd88c41: Fix JSX recipe extraction when multiple recipes were used on the same component, ex:

  ```tsx
  const ComponentWithMultipleRecipes = ({ variant }) => {
    return (
      <button className={cx(pinkRecipe({ variant }), greenRecipe({ variant }), blueRecipe({ variant }))}>Hello</button>
    )
  }
  ```

  Given a `panda.config.ts` with recipes each including a common `jsx` tag name, such as:

  ```ts
  recipes: {
      pinkRecipe: {
          className: 'pinkRecipe',
          jsx: ['ComponentWithMultipleRecipes'],
          base: { color: 'pink.100' },
          variants: {
              variant: {
              small: { fontSize: 'sm' },
              },
          },
      },
      greenRecipe: {
          className: 'greenRecipe',
          jsx: ['ComponentWithMultipleRecipes'],
          base: { color: 'green.100' },
          variants: {
              variant: {
              small: { fontSize: 'sm' },
              },
          },
      },
      blueRecipe: {
          className: 'blueRecipe',
          jsx: ['ComponentWithMultipleRecipes'],
          base: { color: 'blue.100' },
          variants: {
              variant: {
              small: { fontSize: 'sm' },
              },
          },
      },
  },
  ```

  Only the first matching recipe would be noticed and have its CSS generated, now this will properly generate the CSS
  for each of them

- ef1dd676: Fix issue where `staticCss` did not generate all variants in the array of `css` rules
- b50675ca: Refactor parser to support extracting `css` prop in JSX elements correctly.
- Updated dependencies [12c900ee]
- Updated dependencies [5bd88c41]
- Updated dependencies [ef1dd676]
- Updated dependencies [b50675ca]
  - @pandacss/core@0.6.0
  - @pandacss/types@0.6.0
  - @pandacss/token-dictionary@0.6.0
  - @pandacss/is-valid-prop@0.6.0
  - @pandacss/logger@0.6.0
  - @pandacss/shared@0.6.0

## 0.5.1

### Patch Changes

- 53fb0708: Fix `config.staticCss` by filtering types on getPropertyKeys

  It used to throw because of them:

  ```bash
  <css input>:33:21: Missed semicolon
   ELIFECYCLE  Command failed with exit code 1.
  ```

  ```css
  @layer utilities {
      .m_type\:Tokens\[\"spacing\"\] {
          margin: type:Tokens["spacing"]
      }
  }
  ```

- 1ed239cd: Add feature where `config.staticCss.recipes` can now use [`*`] to generate all variants of a recipe.

  before:

  ```ts
  staticCss: {
    recipes: {
      button: [{ size: ['*'], shape: ['*'] }]
    }
  }
  ```

  now:

  ```ts
  staticCss: {
    recipes: {
      button: ['*']
    }
  }
  ```

- 78ed6ed4: Fix issue where using a nested outdir like `src/styled-system` with a baseUrl like `./src` would result on
  parser NOT matching imports like `import { container } from "styled-system/patterns";` cause it would expect the full
  path `src/styled-system`
- b8f8c2a6: Fix reset.css (generated when config has `preflight: true`) import order, always place it first so that it
  can be easily overriden
- Updated dependencies [8c670d60]
- Updated dependencies [c0335cf4]
- Updated dependencies [762fd0c9]
- Updated dependencies [f9247e52]
- Updated dependencies [1ed239cd]
- Updated dependencies [78ed6ed4]
  - @pandacss/types@0.5.1
  - @pandacss/shared@0.5.1
  - @pandacss/logger@0.5.1
  - @pandacss/core@0.5.1
  - @pandacss/token-dictionary@0.5.1
  - @pandacss/is-valid-prop@0.5.1

## 0.5.0

### Minor Changes

- ead9eaa3: Add support for tagged template literal version.

  This features is pure css approach to writing styles, and can be a great way to migrate from styled-components and
  emotion.

  Set the `syntax` option to `template-literal` in the panda config to enable this feature.

  ```js
  // panda.config.ts
  export default defineConfig({
    //...
    syntax: 'template-literal',
  })
  ```

  > For existing projects, you might need to run the `panda codegen --clean`

  You can also use the `--syntax` option to specify the syntax type when using the CLI.

  ```sh
  panda init -p --syntax template-literal
  ```

  To get autocomplete for token variables, consider using the
  [CSS Var Autocomplete](https://marketplace.visualstudio.com/items?itemName=phoenisx.cssvar) extension.

### Patch Changes

- Updated dependencies [60df9bd1]
- Updated dependencies [ead9eaa3]
  - @pandacss/shared@0.5.0
  - @pandacss/types@0.5.0
  - @pandacss/core@0.5.0
  - @pandacss/token-dictionary@0.5.0
  - @pandacss/is-valid-prop@0.5.0
  - @pandacss/logger@0.5.0

## 0.4.0

### Minor Changes

- 5b344b9c: Add support for disabling shorthand props

  ```ts
  import { defineConfig } from '@pandacss/dev'

  export default defineConfig({
    // ...
    shorthands: false,
  })
  ```

### Patch Changes

- 54a8913c: Fix issue where patterns that include css selectors doesn't work in JSX
- a48e5b00: Add support for watch mode in codegen command via the `--watch` or `-w` flag.

  ```bash
  panda codegen --watch
  ```

- Updated dependencies [2a1e9386]
- Updated dependencies [54a8913c]
- Updated dependencies [c7b42325]
- Updated dependencies [5b344b9c]
  - @pandacss/core@0.4.0
  - @pandacss/is-valid-prop@0.4.0
  - @pandacss/types@0.4.0
  - @pandacss/token-dictionary@0.4.0
  - @pandacss/logger@0.4.0
  - @pandacss/shared@0.4.0

## 0.3.2

### Patch Changes

- @pandacss/core@0.3.2
- @pandacss/is-valid-prop@0.3.2
- @pandacss/logger@0.3.2
- @pandacss/shared@0.3.2
- @pandacss/token-dictionary@0.3.2
- @pandacss/types@0.3.2

## 0.3.1

### Patch Changes

- efd79d83: Baseline release for the launch
- Updated dependencies [efd79d83]
  - @pandacss/core@0.3.1
  - @pandacss/is-valid-prop@0.3.1
  - @pandacss/logger@0.3.1
  - @pandacss/shared@0.3.1
  - @pandacss/token-dictionary@0.3.1
  - @pandacss/types@0.3.1

## 0.3.0

### Minor Changes

- 6d81ee9e: - Set default jsx factory to 'styled'
  - Fix issue where pattern JSX was not being generated correctly when properties are not defined

### Patch Changes

- Updated dependencies [6d81ee9e]
  - @pandacss/types@0.3.0
  - @pandacss/core@0.3.0
  - @pandacss/token-dictionary@0.3.0
  - @pandacss/is-valid-prop@0.3.0
  - @pandacss/logger@0.3.0
  - @pandacss/shared@0.3.0

## 0.0.2

### Patch Changes

- fb40fff2: Initial release of all packages

  - Internal AST parser for TS and TSX
  - Support for defining presets in config
  - Support for design tokens (core and semantic)
  - Add `outExtension` key to config to allow file extension options for generated javascript. `.js` or `.mjs`
  - Add `jsxElement` option to patterns, to allow specifying the jsx element rendered by the patterns.

- Updated dependencies [c308e8be]
- Updated dependencies [fb40fff2]
  - @pandacss/types@0.0.2
  - @pandacss/core@0.0.2
  - @pandacss/is-valid-prop@0.0.2
  - @pandacss/logger@0.0.2
  - @pandacss/shared@0.0.2
  - @pandacss/token-dictionary@0.0.2
