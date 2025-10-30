<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Creating StencilJS Components and Exporting as a Package with Storybook Integration

## Creating Components in StencilJS

StencilJS provides a powerful compiler-based approach to building reusable web components. To get started with creating components and preparing them for distribution, follow this comprehensive workflow.[^1_1][^1_2]

### Initial Project Setup

Initialize a new StencilJS project using the component starter template:

```bash
npm init stencil
```

Select the "component" option when prompted, which creates a project structure optimized for building reusable component libraries. The CLI generates a complete project with a components folder where all exportable components will reside.[^1_3][^1_1]

### Component Structure

StencilJS components are TypeScript files with a `.tsx` extension, leveraging JSX for templating. Here's the anatomy of a well-structured component:

```typescript
import { Component, Prop, State, Event, EventEmitter, Watch, h } from '@stencil/core';

@Component({
  tag: 'my-button',
  styleUrl: 'my-button.scss',
  shadow: true  // or scoped: true
})
export class MyButton {
  // Props - public API
  @Prop() label: string;
  @Prop() disabled: boolean = false;
  
  // State - internal reactive state
  @State() isPressed: boolean = false;
  
  // Events - custom events
  @Event() buttonClick: EventEmitter<void>;
  
  // Watchers for prop/state changes
  @Watch('disabled')
  handleDisabledChange(newValue: boolean) {
    // Handle prop changes
  }
  
  // Lifecycle methods
  componentWillLoad() {
    // Initialize before first render
  }
  
  componentDidLoad() {
    // Component fully loaded
  }
  
  // Render method
  render() {
    return (
      <button 
        disabled={this.disabled}
        onClick={() => this.buttonClick.emit()}
      >
        {this.label}
      </button>
    );
  }
}
```


## Configuring Output Targets for Package Distribution

The `stencil.config.ts` file controls how your components are built and distributed. Configure multiple output targets to support various consumption patterns:[^1_4][^1_5][^1_6]

```typescript
import { Config } from '@stencil/core';
import { sass } from '@stencil/sass';

export const config: Config = {
  namespace: 'my-component-library',
  outputTargets: [
    {
      type: 'dist',
      esmLoaderPath: '../loader',
    },
    {
      type: 'dist-custom-elements',
      customElementsExportBehavior: 'auto-define-custom-elements',
    },
    {
      type: 'docs-readme',
    },
    {
      type: 'www',
      serviceWorker: null
    }
  ],
  plugins: [
    sass()
  ]
};
```

The **dist** output target generates a lazy-loadable library suitable for runtime loading, while **dist-custom-elements** creates direct custom element exports that integrate well with frameworks.[^1_7][^1_8]

## Package.json Configuration for NPM Publishing

Configure your `package.json` to properly expose the built components:[^1_9][^1_10][^1_4]

```json
{
  "name": "my-component-library",
  "version": "1.0.0",
  "main": "dist/index.cjs.js",
  "module": "dist/index.js",
  "es2015": "dist/esm/index.mjs",
  "es2017": "dist/esm/index.mjs",
  "types": "dist/types/components.d.ts",
  "unpkg": "dist/my-component-library/my-component-library.esm.js",
  "collection": "dist/collection/collection-manifest.json",
  "collection:main": "dist/collection/index.js",
  "files": [
    "dist/",
    "loader/"
  ],
  "scripts": {
    "build": "stencil build",
    "start": "stencil build --dev --watch --serve",
    "test": "stencil test --spec",
    "test.watch": "stencil test --spec --watch",
    "test.e2e": "stencil test --e2e"
  }
}
```

Build and publish your package:

```bash
npm run build
npm publish
```


## Integrating Storybook with StencilJS

Storybook provides documentation and interactive component exploration. The integration process has been streamlined with the official Stencil plugin.[^1_11][^1_12]

### Installing Storybook Dependencies

For Stencil projects, install Storybook with web components support:[^1_13][^1_12][^1_11]

```bash
npm install --save-dev storybook@8 @storybook/addon-essentials@8 @storybook/addon-links@8 @storybook/addon-interactions@8 @stencil/storybook-plugin
```


### Configuring Storybook

Create `.storybook/main.ts`:

```typescript
import type { StorybookConfig } from '@storybook/web-components-vite';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
  ],
  framework: {
    name: '@stencil/storybook-plugin'
  },
  docs: {
    autodocs: true
  }
};

export default config;
```

Create `.storybook/preview.tsx` to register all components globally:[^1_13][^1_11]

```typescript
import { defineCustomElements } from '../loader/index.js';

// Registers all custom elements globally
defineCustomElements();

export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};
```


### Creating Stories with Documentation

Create a story file alongside your component (e.g., `my-button.stories.ts`):[^1_14][^1_11]

```typescript
import type { Meta, StoryObj } from '@storybook/web-components';
import './my-button';

const meta: Meta = {
  title: 'Components/MyButton',
  component: 'my-button',
  tags: ['autodocs'],
  argTypes: {
    label: {
      control: 'text',
      description: 'Button label text'
    },
    disabled: {
      control: 'boolean',
      description: 'Disables the button'
    },
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
      description: 'Button style variant'
    }
  }
};

export default meta;
type Story = StoryObj;

export const Primary: Story = {
  args: {
    label: 'Click Me',
    disabled: false,
    variant: 'primary'
  }
};

export const Disabled: Story = {
  args: {
    label: 'Disabled Button',
    disabled: true
  }
};
```


### Creating MDX Documentation

For comprehensive documentation, create MDX files that combine prose with interactive examples:[^1_15]

```mdx
import { Meta, Canvas, Controls, ArgTypes } from '@storybook/blocks';
import * as MyButtonStories from './my-button.stories';

<Meta of={MyButtonStories} />

# MyButton Component

A reusable button component with multiple variants and states.

## Usage

```

<my-button label="Click Me" variant="primary"></my-button>

```

## Properties

<ArgTypes of={MyButtonStories} />

## Interactive Examples

<Canvas of={MyButtonStories.Primary} />
<Controls of={MyButtonStories.Primary} />

## Best Practices

- Use semantic button variants for clear intent
- Always provide descriptive labels for accessibility
- Consider disabled states for form validation
```

Add Storybook scripts to `package.json`:[^1_11]

```json
{
  "scripts": {
    "storybook": "storybook dev -p 6006 --no-open",
    "build-storybook": "storybook build"
  }
}
```

Run Storybook:

```bash
npm run storybook
```


## Best Practices for Reusable Component Creation

### Component Design Principles

**Single Responsibility**: Each component should have one clear purpose. Keep components focused and composable rather than creating monolithic components.[^1_16][^1_17]

**Use Shadow DOM or Scoped Styles Appropriately**: Shadow DOM (`shadow: true`) provides complete style encapsulation, preventing external styles from affecting your component and vice versa. Use this for components that need strict isolation.[^1_18][^1_19][^1_20]

```typescript
@Component({
  tag: 'isolated-component',
  styleUrl: 'isolated-component.scss',
  shadow: true  // Complete style isolation
})
```

Scoped CSS (`scoped: true`) provides partial encapsulation by adding unique data attributes to styles. Use this when you want some external style control:[^1_19][^1_18]

```typescript
@Component({
  tag: 'flexible-component',
  styleUrl: 'flexible-component.scss',
  scoped: true  // Scoped but allows external styling
})
```

**Prop Design Best Practices**:[^1_21][^1_22][^1_23]

- Use primitive types for props that need HTML attribute reflection (string, number, boolean)
- Props automatically convert from camelCase to kebab-case in HTML attributes[^1_24]
- Mark props as readonly - avoid modifying them internally[^1_25][^1_21]
- Provide sensible defaults using property initializers

```typescript
@Prop() variant: 'primary' | 'secondary' = 'primary';
@Prop() size: 'small' | 'medium' | 'large' = 'medium';
@Prop({ reflect: true }) disabled: boolean = false;
```

**State Management**: Use `@State()` only for internal reactive state that should trigger re-renders. Keep state minimal and consider external state management for complex scenarios.[^1_26][^1_21]

**Event Naming and Design**: Custom events should use clear, descriptive names and emit meaningful data:[^1_22][^1_25]

```typescript
@Event() selectionChange: EventEmitter<{ value: string; selected: boolean }>;
```


### Component Lifecycle Best Practices

Follow the proper lifecycle order for initialization and cleanup:[^1_27][^1_28][^1_22]

1. **connectedCallback()**: Called every time the component is attached to the DOM
2. **componentWillLoad()**: Called once before first render - ideal for async data loading
3. **componentDidLoad()**: Called once after first render
4. **componentWillUpdate()**: Called before re-renders
5. **componentDidUpdate()**: Called after re-renders
6. **disconnectedCallback()**: Cleanup when removed from DOM
```typescript
export class MyComponent {
  private timer: number;
  
  componentWillLoad() {
    // One-time initialization
  }
  
  connectedCallback() {
    // Set up listeners (can be called multiple times)
    this.timer = window.setInterval(() => {
      // Update logic
    }, 1000);
  }
  
  disconnectedCallback() {
    // Always clean up resources
    window.clearInterval(this.timer);
  }
}
```


### Accessibility Considerations

Make components accessible by default:[^1_29][^1_30][^1_31]

- Use semantic HTML elements as the foundation
- Add ARIA attributes only when native HTML semantics are insufficient
- Ensure keyboard navigation support for interactive elements
- Provide meaningful labels and descriptions

```typescript
render() {
  return (
    <button
      aria-label={this.ariaLabel || this.label}
      aria-disabled={this.disabled ? 'true' : 'false'}
      disabled={this.disabled}
      role="button"
      tabindex={this.disabled ? -1 : 0}
    >
      {this.label}
    </button>
  );
}
```


### Testing Strategy

Implement comprehensive testing with both unit and E2E tests:[^1_32][^1_33][^1_34]

**Unit Tests** (`.spec.ts`) focus on component methods and logic:

```typescript
import { newSpecPage } from '@stencil/core/testing';
import { MyButton } from './my-button';

describe('my-button', () => {
  it('renders with default props', async () => {
    const page = await newSpecPage({
      components: [MyButton],
      html: `<my-button label="Test"></my-button>`,
    });
    expect(page.root).toEqualHtml(`
      <my-button label="Test">
        <button>Test</button>
      </my-button>
    `);
  });
});
```

**E2E Tests** (`.e2e.ts`) verify rendering and interactions in a real browser:[^1_33][^1_35]

```typescript
import { newE2EPage } from '@stencil/core/testing';

describe('my-button e2e', () => {
  it('emits event on click', async () => {
    const page = await newE2EPage();
    await page.setContent('<my-button label="Click"></my-button>');
    
    const button = await page.find('my-button');
    const clickEvent = await page.spyOnEvent('buttonClick');
    
    await button.click();
    await page.waitForChanges();
    
    expect(clickEvent).toHaveReceivedEvent();
  });
});
```


### Styling Best Practices

Use CSS custom properties for themability:[^1_36][^1_37]

```scss
:host {
  --button-bg: #007bff;
  --button-color: white;
  --button-padding: 0.5rem 1rem;
  
  display: inline-block;
}

button {
  background: var(--button-bg);
  color: var(--button-color);
  padding: var(--button-padding);
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
}
```


### Component Documentation

Document your components thoroughly:[^1_28]

```typescript
/**
 * @slot - Default slot for button content
 * @part button - The native button element
 */
@Component({
  tag: 'my-button',
  styleUrl: 'my-button.scss',
  shadow: true
})
export class MyButton {
  /**
   * The button label text
   */
  @Prop() label: string;
  
  /**
   * Disables the button when true
   */
  @Prop() disabled: boolean = false;
  
  /**
   * Emitted when the button is clicked
   */
  @Event() buttonClick: EventEmitter<void>;
}
```


### Naming Conventions

Follow consistent naming patterns:[^1_38][^1_39][^1_24]

- **Component tags**: Use kebab-case with a namespace prefix (e.g., `my-lib-button`)
- **File names**: Match component tag name (`my-lib-button.tsx`)
- **Class names**: Use PascalCase (`MyLibButton`)
- **Props**: Use camelCase in TypeScript, automatically converted to kebab-case in HTML


### Performance Optimization

- Use functional components for presentational components without state[^1_40]
- Leverage lazy loading with the `dist` output target[^1_8][^1_4]
- Minimize Shadow DOM complexity for better performance[^1_17]
- Use `componentShouldUpdate()` to prevent unnecessary re-renders[^1_27]

By following these practices, you'll create a robust, reusable component library that's well-documented, tested, and ready for distribution across multiple frameworks and projects.[^1_41][^1_2][^1_37][^1_42][^1_43][^1_3][^1_16]
<span style="display:none">[^1_44][^1_45][^1_46][^1_47][^1_48][^1_49][^1_50][^1_51][^1_52][^1_53][^1_54][^1_55][^1_56][^1_57][^1_58][^1_59][^1_60][^1_61][^1_62][^1_63][^1_64][^1_65][^1_66][^1_67][^1_68][^1_69][^1_70][^1_71][^1_72][^1_73][^1_74][^1_75][^1_76][^1_77][^1_78][^1_79][^1_80][^1_81][^1_82][^1_83][^1_84][^1_85][^1_86][^1_87][^1_88]</span>

<div align="center">⁂</div>

[^1_1]: https://stenciljs.com/docs/getting-started

[^1_2]: https://dev.to/johnbwoodruff/component-libraries-with-stenciljs---getting-started-4jej

[^1_3]: https://dev.to/seanmclem/consuming-a-stencil-js-components-in-several-frameworks-72p

[^1_4]: https://stenciljs.com/docs/publishing

[^1_5]: https://techblog.skeepers.io/unlocking-cross-framework-power-stenciljs-configuration-demystified-cd12933b1aaf

[^1_6]: https://stenciljs.com/docs/output-targets

[^1_7]: https://stenciljs.com/docs/custom-elements

[^1_8]: https://stenciljs.com/docs/distribution

[^1_9]: https://stenciljs.com/docs/v2/publishing

[^1_10]: https://stenciljs.com/docs/v3/publishing

[^1_11]: https://stenciljs.com/docs/next/storybook

[^1_12]: https://dev.to/jsenosiain/setting-up-stencil-4-with-storybook-8-499c

[^1_13]: https://dev.to/jfgmdev/stenciljs-with-storybook-3027

[^1_14]: https://storybook.js.org/addons/storybook-addon-stencil

[^1_15]: https://storybook.js.org/docs/writing-docs/mdx

[^1_16]: https://blog.pixelfreestudio.com/best-practices-for-building-reusable-components-in-web-development/

[^1_17]: https://dev.to/dhrumitdk/designing-with-web-components-a-modern-approach-to-modular-design-4de0

[^1_18]: https://forum.ionicframework.com/t/faq-what-is-the-difference-between-the-shadow-dom-and-scoped-css/218131

[^1_19]: https://stenciljs.jp/docs/styling/

[^1_20]: https://stenciljs.com/docs/v4.23/styling

[^1_21]: https://stenciljs.com/docs/state

[^1_22]: https://stenciljs.com/docs/api

[^1_23]: https://www.squash.io/optimal-practices-for-every-javascript-component/

[^1_24]: https://terodox.tech/stencil-js-part-3/

[^1_25]: https://blog.enable.engineering/building-better-design-systems-stencil-js-vs-react-vs-custom-elements-b3ef41b66885

[^1_26]: https://stackoverflow.com/questions/60083915/how-to-share-state-between-stenciljs-components

[^1_27]: https://stenciljs.com/docs/component-lifecycle

[^1_28]: https://stenciljs.com/docs/style-guide

[^1_29]: https://accessibilityspark.com/aria-accessibility/

[^1_30]: https://www.a11y-collective.com/blog/aria-labels/

[^1_31]: https://www.accessibilitychecker.org/blog/aria-accessibility/

[^1_32]: https://stenciljs.com/docs/v3/testing-overview

[^1_33]: https://stenciljs.com/docs/end-to-end-testing

[^1_34]: https://stencil.docschina.org/docs/testing-overview

[^1_35]: https://blog.pixelfreestudio.com/best-practices-for-testing-web-components/

[^1_36]: https://stenciljs.com/docs/styling

[^1_37]: https://giancarlobuomprisco.com/stencil/creating-and-integrating-design-systems-with-stenciljs

[^1_38]: https://github.com/stenciljs/core/issues/6194

[^1_39]: https://stenciljs.com/docs/v2/style-guide

[^1_40]: https://blog.logrocket.com/building-reusable-web-components-with-stencil-js/

[^1_41]: https://github.com/alesgenova/stenciljs-in-react

[^1_42]: https://github.com/ranjeetsinghbnl/stenciljs-react

[^1_43]: https://blog.pixelfreestudio.com/best-practices-for-building-reusable-web-components/

[^1_44]: https://whoisryosuke.com/blog/2019/using-stencil-with-storybook

[^1_45]: https://www.reddit.com/r/ionic/comments/dgtmnd/best_practices_for_reusable_web_components_using/

[^1_46]: https://techblog.skeepers.io/stenciljs-a-deep-dive-on-the-inner-workings-of-output-targets-91199ace026e

[^1_47]: https://www.reddit.com/r/reactjs/comments/1acdozf/is_it_always_a_good_practice_to_do_reusable/

[^1_48]: https://pusher.com/blog/getting-started-stenciljs/

[^1_49]: https://www.youtube.com/watch?v=3ioIwFx_tKc

[^1_50]: https://dev.to/theodesp/creating-reusable-web-components-with-stencil-js-4oc6

[^1_51]: https://auth0.com/blog/creating-web-components-with-stencil/

[^1_52]: https://stackoverflow.com/questions/74032076/integrating-storybook-with-stencil-app-starter

[^1_53]: https://stackoverflow.com/questions/77524513/reusable-styles-with-stenciljs-web-components

[^1_54]: https://www.youtube.com/watch?v=xKvhWKo-XGA

[^1_55]: https://www.theksquaregroup.com/software-engineering/stenciljs/

[^1_56]: https://www.youtube.com/watch?v=58q09WeX2zk

[^1_57]: https://stenciljs.com/docs/design-systems

[^1_58]: https://stenciljs.com/docs/v2/distribution

[^1_59]: https://github.com/emrekeskinmac/stenciljs-example/blob/master/package.json

[^1_60]: https://stenciljs.com/docs/v4.8/publishing

[^1_61]: https://github.com/marksy/storybook-8-web-components

[^1_62]: https://github.com/johnmcase/storybook-stencil-mdx

[^1_63]: https://stenciljs.jikun.dev/docs/components/styling

[^1_64]: https://stenciljs.com/docs/storybook

[^1_65]: https://www.youtube.com/watch?v=6AwDZf9KwEY

[^1_66]: https://blog.pixelfreestudio.com/how-to-use-stencil-js-for-building-web-components/

[^1_67]: https://www.designsystemscollective.com/how-to-use-storybook-with-stencil-in-2025-and-why-lit-isnt-the-best-choice-81fb5c2d521e

[^1_68]: https://forum.ionicframework.com/t/stencil-publishing-a-stencil-component-to-npm/120940

[^1_69]: https://storybook.js.org/docs/api/arg-types

[^1_70]: https://storybook.js.org/docs/essentials/controls

[^1_71]: https://www.browserstack.com/guide/how-to-use-storybook-argtypes

[^1_72]: https://github.com/ionic-team/stencil/issues/502

[^1_73]: https://stackoverflow.com/questions/73690449/storybook-with-web-components-changing-arguments-dynamically-on-code

[^1_74]: https://www.reddit.com/r/DesignSystems/comments/1agr1qc/how_reusable_can_a_design_system_truly_be/

[^1_75]: https://github.com/storybookjs/storybook/discussions/17752

[^1_76]: https://cianfrani.dev/posts/web-component-best-practices/

[^1_77]: https://github.com/storybookjs/storybook/issues/16089

[^1_78]: https://www.reddit.com/r/reactjs/comments/18r3wvu/recommendations_on_reusable_component_framework/

[^1_79]: https://www.youtube.com/watch?v=us5IhAIgtWE

[^1_80]: https://sandroroth.com/blog/storybook-controlled-components/

[^1_81]: https://www.reddit.com/r/reactjs/comments/s2wbfx/jsts_frameworks_file_naming_convention_camelcase/

[^1_82]: http://stencil.docschina.org/docs/testing-overview/

[^1_83]: https://stackoverflow.com/questions/59712809/stencil-namespacing-custom-elements-names-to-avoid-collisions

[^1_84]: https://www.w3.org/WAI/ARIA/apg/

[^1_85]: https://dev.to/adamgolan/building-accessible-web-components-a-deep-dive-into-aria-best-practices-2e7i

[^1_86]: https://eliteionic.com/tutorials/tdd-with-stenciljs-refactoring-page-ojects-beforeeach/

[^1_87]: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA

[^1_88]: https://www.w3.org/WAI/standards-guidelines/aria/


---

# StencilJS Presentation Document

A comprehensive, ready-to-present Markdown document has been prepared, covering StencilJS usage, packaging for NPM, Storybook integration (including Docs and usage), and best practices for creating reusable components.

You can download or view the complete presentation here:

