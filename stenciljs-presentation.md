# StencilJS Components: Usage, Packaging, Storybook Integration & Best Practices

---

## 1. Introduction
StencilJS provides a standards-based way to create reusable, distributable web components. This guide covers component creation, packaging for NPM, Storybook integration for documentation, and best practices for design and testing.

---

## 2. Creating Components in StencilJS

**Workflow**:
- Initialize your project: `npm init stencil`
- Scaffold generates a `src/components` directory.

**Example Component Structure**:
```typescript
import { Component, Prop, State, Event, EventEmitter, Watch, h } from '@stencil/core';

@Component({
  tag: 'my-button',
  styleUrl: 'my-button.scss',
  shadow: true
})
export class MyButton {
  @Prop() label: string;
  @Prop() disabled: boolean = false;
  @Event() buttonClick: EventEmitter<void>;

  render() {
    return (
      <button disabled={this.disabled} onClick={() => this.buttonClick.emit()}>
        {this.label}
      </button>
    );
  }
}
```

---

## 3. Packaging Components for Distribution

**Output Targets Configuration (stencil.config.ts):**
```typescript
outputTargets: [
  { type: 'dist', esmLoaderPath: '../loader' },
  { type: 'dist-custom-elements', customElementsExportBehavior: 'auto-define-custom-elements' },
  { type: 'docs-readme' },
]
```

**package.json Setup:**
```json
{
  "main": "dist/index.cjs.js",
  "module": "dist/index.js",
  "types": "dist/types/components.d.ts",
  "files": ["dist/", "loader/"]
}
```

**Publish:**
- Build: `npm run build`
- Publish: `npm publish`

---

## 4. Storybook Integration & Docs

**Install Storybook:**
```sh
npm install --save-dev storybook@8 @stencil/storybook-plugin @storybook/addon-essentials
```

**Configure Storybook (.storybook/main.ts):**
```typescript
framework: { name: '@stencil/storybook-plugin' },
docs: { autodocs: true },
stories: ['../src/**/*.stories.@(ts|mdx)']
```

**Preview Setup (.storybook/preview.tsx):**
```typescript
import { defineCustomElements } from '../loader';
defineCustomElements();
```

**Create Stories (my-button.stories.ts):**
```typescript
import type { Meta, StoryObj } from '@storybook/web-components';
const meta: Meta = { title: 'Components/MyButton', component: 'my-button' };
export default meta;
export const Primary = { args: { label: 'Click Me', disabled: false } };
```

**MDX Documentation Example:**
```mdx
# MyButton Component
<Canvas of={MyButtonStories.Primary} />
<Controls of={MyButtonStories.Primary} />
```

Run Storybook:
```sh
npm run storybook
```

---

## 5. Best Practices for Reusable Component Creation

### Component Design
- **Single Responsibility:** One component, one function.
- **Shadow DOM:** Use when strict encapsulation needed (`shadow: true`).
- **Props:** Use primitive types, provide sensible defaults, leverage reflection.
- **Event Naming:** Prefer descriptive, meaningful names.

### Lifecycle and Accessibility
- Use lifecycle methods (`componentWillLoad`, etc.) for setup/cleanup.
- Use semantic HTML, ARIA attributes, and keyboard navigation.

### Styling & Theming
- Prefer CSS custom properties and themable design:
  ```scss
  :host {
    --button-bg: #007bff;
  }
  button {
    background: var(--button-bg);
  }
  ```

### Testing
- **Unit Tests:** Focus on logic and render output.
- **E2E Tests:** Validate user interaction and event emission.

### Documentation
- Use JSDoc comments; autogenerate docs with Storybook and output targets.

### Naming Conventions
- Component tag: kebab-case, use prefixes (`my-lib-button`).
- File/class names: PascalCase for classes, kebab-case for tags/files.

---

## 6. Summary
StencilJS enables robust component libraries that can be distributed via NPM and integrated across frameworks. Combine strong encapsulation, comprehensive docs via Storybook, accessible design, and thorough testing for professional reusable components.