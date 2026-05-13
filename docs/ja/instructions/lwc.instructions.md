---
description: 'Salesforce Platform 上で Lightning Web Components (LWC) を開発するためのガイドラインとベストプラクティス。'
applyTo: 'force-app/main/default/lwc/**'
---

# LWC 開発

## 一般指示

- 各 LWC は `force-app/main/default/lwc/` 配下の独立したフォルダーに配置する。
- フォルダー名は component 名と一致させる（例: `myComponent` component なら `myComponent` フォルダー）。
- 各 component フォルダーには次の file を含める。
    - `myComponent.html`: HTML template file。
    - `myComponent.js`: JavaScript controller file。
    - `myComponent.js-meta.xml`: metadata configuration file。
    - 任意: component 固有 style 用の `myComponent.css`。
    - 任意: Jest unit test 用の `myComponent.test.js`。

## コア原則

### 1. HTML Tag より Lightning Component を使う
一貫性、アクセシビリティ、将来性のため、通常の HTML 要素より Lightning Web Component library の component を優先してください。

#### 推奨アプローチ
```html
<!-- Use Lightning components -->
<lightning-button label="Save" variant="brand" onclick={handleSave}></lightning-button>
<lightning-input type="text" label="Name" value={name} onchange={handleNameChange}></lightning-input>
<lightning-combobox label="Type" options={typeOptions} value={selectedType}></lightning-combobox>
<lightning-radio-group name="duration" label="Duration" options={durationOptions} value={duration} type="radio"></lightning-radio-group>
```

#### 通常の HTML は避ける
```html
<!-- Avoid these -->
<button onclick={handleSave}>Save</button>
<input type="text" onchange={handleNameChange} />
<select onchange={handleTypeChange}>
    <option value="option1">Option 1</option>
</select>
```

### 2. Lightning Component 対応表

| HTML Element | Lightning Component | Key Attributes |
|--------------|-------------------|----------------|
| `<button>` | `<lightning-button>` | `variant`, `label`, `icon-name` |
| `<input>` | `<lightning-input>` | `type`, `label`, `variant` |
| `<select>` | `<lightning-combobox>` | `options`, `value`, `placeholder` |
| `<textarea>` | `<lightning-textarea>` | `label`, `max-length` |
| `<input type="checkbox">` | `<lightning-input type="checkbox">` | `checked`, `label` |
| `<input type="radio">` | `<lightning-radio-group>` | `options`, `type`, `name` |
| `<input type="toggle">` | `<lightning-input type="toggle">` | `checked`, `variant` |
| Custom pills | `<lightning-pill>` | `label`, `name`, `onremove` |
| Icons | `<lightning-icon>` | `icon-name`, `size`, `variant` |

### 3. Lightning Design System への準拠

#### SLDS Utility Class を使う
モダンな実装では、常に `slds-var-` prefix を持つ Salesforce Lightning Design System utility class を使ってください。

```html
<!-- Spacing -->
<div class="slds-var-m-around_medium slds-var-p-top_large">
    <div class="slds-var-m-bottom_small">Content</div>
</div>

<!-- Layout -->
<div class="slds-grid slds-wrap slds-gutters_small">
    <div class="slds-col slds-size_1-of-2 slds-medium-size_1-of-3">
        <!-- Content -->
    </div>
</div>

<!-- Typography -->
<h2 class="slds-text-heading_medium slds-var-m-bottom_small">Section Title</h2>
<p class="slds-text-body_regular">Description text</p>
```

#### SLDS Component Pattern
```html
<!-- Card Layout -->
<article class="slds-card slds-var-m-around_medium">
    <header class="slds-card__header">
        <h2 class="slds-text-heading_small">Card Title</h2>
    </header>
    <div class="slds-card__body slds-card__body_inner">
        <!-- Card content -->
    </div>
    <footer class="slds-card__footer">
        <!-- Card actions -->
    </footer>
</article>

<!-- Form Layout -->
<div class="slds-form slds-form_stacked">
    <div class="slds-form-element">
        <lightning-input label="Field Label" value={fieldValue}></lightning-input>
    </div>
</div>
```

### 4. Custom CSS を避ける

#### SLDS Class を使う
```html
<!-- Color and theming -->
<div class="slds-theme_success slds-text-color_inverse slds-var-p-around_small">
    Success message
</div>

<div class="slds-theme_error slds-text-color_inverse slds-var-p-around_small">
    Error message
</div>

<div class="slds-theme_warning slds-text-color_inverse slds-var-p-around_small">
    Warning message
</div>
```

#### Custom CSS を避ける（アンチパターン）
```css
/* Don't create custom styles that override SLDS */
.custom-button {
    background-color: red;
    padding: 10px;
}

.my-special-layout {
    display: flex;
    justify-content: center;
}
```

#### Custom CSS が必要な場合
やむを得ず custom CSS を使う場合は、次のガイドラインに従ってください。
- 可能なら CSS custom property（design token）を使う
- 競合を避けるため custom class に prefix を付ける
- SLDS base class は決して上書きしない

```css
/* Custom CSS example */
.my-component-special {
    border-radius: var(--lwc-borderRadiusMedium);
    box-shadow: var(--lwc-shadowButton);
}
```

### 5. Component Architecture のベストプラクティス

#### Reactive Property
```javascript
import { LightningElement, track, api } from 'lwc';

export default class MyComponent extends LightningElement {
    // Use @api for public properties
    @api recordId;
    @api title;

    // Primitive properties (string, number, boolean) are automatically reactive
    // No decorator needed - reassignment triggers re-render
    simpleValue = 'initial';
    count = 0;

    // Computed properties
    get displayName() {
        return this.name ? `Hello, ${this.name}` : 'Hello, Guest';
    }

    // @track is NOT needed for simple property reassignment
    // This will trigger reactivity automatically:
    handleUpdate() {
        this.simpleValue = 'updated'; // Reactive without @track
        this.count++; // Reactive without @track
    }

    // @track IS needed when mutating nested properties without reassignment
    @track complexData = {
        user: {
            name: 'John',
            preferences: {
                theme: 'dark'
            }
        }
    };

    handleDeepUpdate() {
        // Requires @track because we're mutating a nested property
        this.complexData.user.preferences.theme = 'light';
    }

    // BETTER: Avoid @track by using immutable patterns
    regularData = {
        user: {
            name: 'John',
            preferences: {
                theme: 'dark'
            }
        }
    };

    handleImmutableUpdate() {
      // No @track needed - we're creating a new object reference
      this.regularData = {
        ...this.regularData,
        user: {
          ...this.regularData.user,
          preferences: {
            ...this.regularData.user.preferences,
            theme: 'light'
          }
        }
      };
    }

    // Arrays: @track is needed only for mutating methods
    @track items = ['a', 'b', 'c'];

    handleArrayMutation() {
      // Requires @track
      this.items.push('d');
      this.items[0] = 'z';
    }

    // BETTER: Use immutable array operations
    regularItems = ['a', 'b', 'c'];

    handleImmutableArray() {
      // No @track needed
      this.regularItems = [...this.regularItems, 'd'];
      this.regularItems = this.regularItems.map((item, idx) =>
        idx === 0 ? 'z' : item
      );
    }

    // Use @track only for complex objects/arrays when you mutate nested properties.
    // For example, updating complexObject.details.status without reassigning complexObject.
    @track complexObject = {
      details: {
        status: 'new'
      }
    };
}
```

#### Event Handling Pattern
```javascript
// Custom event dispatch
handleSave() {
    const saveEvent = new CustomEvent('save', {
        detail: {
            recordData: this.recordData,
            timestamp: new Date()
        }
    });
    this.dispatchEvent(saveEvent);
}

// Lightning component event handling
handleInputChange(event) {
    const fieldName = event.target.name;
    const fieldValue = event.target.value;

    // For lightning-input, lightning-combobox, etc.
    this[fieldName] = fieldValue;
}

handleRadioChange(event) {
    // For lightning-radio-group
    this.selectedValue = event.detail.value;
}

handleToggleChange(event) {
    // For lightning-input type="toggle"
    this.isToggled = event.detail.checked;
}
```

### 6. データ処理と Wire Service

#### データアクセスには @wire を使う
```javascript
import { getRecord } from 'lightning/uiRecordApi';
import { getObjectInfo } from 'lightning/uiObjectInfoApi';

const FIELDS = ['Account.Name', 'Account.Industry', 'Account.AnnualRevenue'];

export default class MyComponent extends LightningElement {
    @api recordId;

    @wire(getRecord, { recordId: '$recordId', fields: FIELDS })
    record;

    @wire(getObjectInfo, { objectApiName: 'Account' })
    objectInfo;

    get recordData() {
        return this.record.data ? this.record.data.fields : {};
    }
}
```

### 7. Error Handling と User Experience

#### 適切な Error Boundary を実装する
```javascript
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class MyComponent extends LightningElement {
    isLoading = false;
    error = null;

    async handleAsyncOperation() {
        this.isLoading = true;
        this.error = null;

        try {
            const result = await this.performOperation();
            this.showSuccessToast();
        } catch (error) {
            this.error = error;
            this.showErrorToast(error.body?.message || 'An error occurred');
        } finally {
            this.isLoading = false;
        }
    }

    performOperation() {
        // Developer-defined async operation
    }

    showSuccessToast() {
        const event = new ShowToastEvent({
            title: 'Success',
            message: 'Operation completed successfully',
            variant: 'success'
        });
        this.dispatchEvent(event);
    }

    showErrorToast(message) {
        const event = new ShowToastEvent({
            title: 'Error',
            message: message,
            variant: 'error',
            mode: 'sticky'
        });
        this.dispatchEvent(event);
    }
}
```

### 8. パフォーマンス最適化

#### Conditional Rendering
conditional rendering には `lwc:if`, `lwc:elseif`, `lwc:else` を優先してください（API v58.0+）。legacy の `if:true` / `if:false` も引き続きサポートされますが、新規 component では避けてください。

```html
<!-- Use template directives for conditional rendering -->
<template lwc:if={isLoading}>
    <lightning-spinner alternative-text="Loading..."></lightning-spinner>
</template>
<template lwc:elseif={error}>
    <div class="slds-theme_error slds-text-color_inverse slds-var-p-around_small">
        {error.message}
    </div>
</template>
<template lwc:else>
    <template for:each={items} for:item="item">
        <div key={item.id} class="slds-var-m-bottom_small">
            {item.name}
        </div>
    </template>
</template>
```

```html
<!-- Legacy approach (avoid in new components) -->
<template if:true={isLoading}>
    <lightning-spinner alternative-text="Loading..."></lightning-spinner>
</template>
<template if:true={error}>
    <div class="slds-theme_error slds-text-color_inverse slds-var-p-around_small">
        {error.message}
    </div>
</template>
<template if:false={isLoading}>
  <template if:false={error}>
    <template for:each={items} for:item="item">
        <div key={item.id} class="slds-var-m-bottom_small">
            {item.name}
        </div>
    </template>
  </template>
</template>
```

### 9. アクセシビリティのベストプラクティス

#### 適切な ARIA Label と Semantic HTML を使う
```html
<!-- Use semantic structure -->
<section aria-label="Product Selection">
    <h2 class="slds-text-heading_medium">Products</h2>

    <lightning-input
        type="search"
        label="Search Products"
        placeholder="Enter product name..."
        aria-describedby="search-help">
    </lightning-input>

    <div id="search-help" class="slds-assistive-text">
        Type to filter the product list
    </div>
</section>
```

## 避けるべきよくあるアンチパターン
- **Direct DOM Manipulation**: `document.querySelector()` などは決して使わない
- **jQuery や External Library**: Lightning と互換性のない library は避ける
- **Inline Style**: `style` attribute ではなく SLDS class を使う
- **Global CSS**: すべての style は component にスコープされるべき
- **Hardcoded Value**: custom label、custom metadata、constant を使う
- **Imperative API Call**: 可能なら imperative な `import` call より `@wire` を優先する
- **Memory Leak**: `disconnectedCallback()` で event listener を必ず cleanup する
