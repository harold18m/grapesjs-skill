# GrapesJS API Reference

## Table of Contents
- Components
- Custom Component Types
- Component Lifecycle Hooks
- Blocks
- Panels & Buttons
- Commands
- Storage
- Devices & Responsive
- Events
- Pages
- Assets
- Layers
- Modal
- Selectors
- Component CSS
- Data Sources

## Components

### Reading & Updating

```js
const wrapper = editor.DomComponents.getWrapper();
const component = editor.getSelected();

// Properties
component.get('tagName');
component.set('tagName', 'span');
component.props(); // all properties

// Attributes
component.getAttributes();
component.setAttributes({ id: 'test', 'data-key': 'value' });
component.addAttributes({ title: 'Test' });
component.removeAttributes(['some-attr']);

// Classes
component.addClass('class1 class2');
component.removeClass('class1');
component.getClasses(); // ['class1', 'class2']
component.setClass('new-class');

// Style
component.getStyle();
component.setStyle({ color: 'red', width: '100px' });

// Children
component.components(); // get children collection
component.components('<div>Replace children</div>');
component.append('<div>New child</div>');
component.append(otherComponent, { at: 0 }); // prepend
component.empty(); // remove all children
component.getChildAt(0);
component.getLastChild();

// Traversal
component.parent();
component.parents();
component.find('div.my-class'); // CSS query (rendered only)
component.findType('image'); // by type (works before render)
component.closest('.some-class');
component.closestType('section');
component.contains(otherComponent);
component.index(); // position among siblings

// Output
component.toHTML();
component.toHTML({ withProps: true }); // re-importable HTML
component.getInnerHTML();
JSON.stringify(component); // JSON

// Other
component.getName();
component.setName('My Component');
component.getId();
component.remove();
component.move(destComponent, { at: 0 });
component.replaceWith('<div>New</div>');
component.is('image'); // check component type
component.getView(); // get the View instance
component.getEl(); // get the DOM element in canvas
```

### Traits

```js
component.getTraits();
component.setTraits([{ type: 'checkbox', name: 'disabled' }]);
component.getTrait('title');
component.addTrait({ type: 'text', name: 'placeholder' }, { at: 1 });
component.updateTrait('title', { type: 'select', options: ['A', 'B'] });
component.removeTrait('title');
```

## Custom Component Types

Define inside a plugin. Use `isComponent` for HTML recognition. Use `extend` to inherit from other types.

```js
editor.DomComponents.addType('my-input', {
  isComponent: el => el.tagName === 'INPUT',
  extend: 'default', // optional, inherit from another type
  extendFn: ['init'], // extend parent's init instead of overriding

  model: {
    defaults: {
      tagName: 'input',
      draggable: 'form, form *',
      droppable: false,
      attributes: { type: 'text', placeholder: 'Type here' },
      traits: ['name', 'placeholder', { type: 'checkbox', name: 'required' }],
      // Component-scoped CSS
      styles: `.my-input { border: 1px solid #ccc; }`,
      // Inner components
      components: '<span>Inner</span>',
      // Restrict stylable properties
      stylable: ['width', 'height'],
      // Or hide specific properties
      unstylable: ['color'],
    },

    init() {
      this.on('change:attributes:title', this.handleTitleChange);
    },
    updated(property, value, prevValue) { /* on any prop change */ },
    removed() { /* cleanup */ },
    handleTitleChange() {
      console.log('Title:', this.getAttributes().title);
    },
  },

  view: {
    events: {
      click: 'handleClick',
      'dblclick .inner': 'handleInnerDblClick',
    },
    init({ model }) {
      this.listenTo(model, 'change:someProp', this.onPropChange);
    },
    onRender({ el, model }) {
      // Add custom canvas-only UI (not in export HTML)
    },
    removed() { /* cleanup external listeners */ },
    handleClick() {},
    handleInnerDblClick(ev) { ev.stopPropagation(); },
  },
});
```

### Extending Component Types

```js
const domc = editor.DomComponents;

// Update an existing type (merges with existing)
domc.addType('some-component', {
  model: {
    defaults: { someNewProp: 'Hello' },
    init() { /* overrides init */ },
  },
});

// Extend from a different type
domc.addType('my-new-component', {
  isComponent: el => { /* ... */ },
  extend: 'other-defined-component',
  model: { /* extends model from 'other-defined-component' */ },
  view: { /* extends view from 'other-defined-component' */ },
});

// Extend with different model and view parents
domc.addType('my-new-component', {
  extend: 'some-type',        // model extends from this
  extendView: 'other-type',   // view extends from this
  extendFn: ['init'],          // extend parent model functions
  extendFnView: ['onRender'],  // extend parent view functions
  model: { /* ... */ },
  view: { /* ... */ },
});

// Get all component types
domc.getTypes().forEach(t => console.log(t.id));
```

### Component Definition

When adding components via HTML strings, they are parsed to **Component Definitions**:

```js
// HTML parsing result:
{
  tagName: 'div',
  components: [
    { type: 'image', attributes: { src: 'https://...' } },
    {
      tagName: 'span', type: 'text',
      attributes: { title: 'foo' },
      components: [{ type: 'textnode', content: 'Hello world!!!' }]
    }
  ]
}
```

**Skip parsing** by passing objects or using `data-gjs-type`:
```js
editor.addComponents({ type: 'my-type' }); // isComponent NOT called
editor.addComponents('<div data-gjs-type="my-type">...</div>'); // isComponent NOT called
editor.addComponents('<div>...</div>'); // isComponent IS called
```

**Defaults with `components` as function:**
```js
defaults: {
  components: model => `<h1>Header: ${model.get('type')}</h1>`,
}
```

## Component Lifecycle Hooks

### Order
1. `model.init()` → `component:create` event
2. `view.init()` → `view.onRender()` → `component:mount` event
3. `model.updated()` → `component:update` / `component:update:{prop}` event
4. `model.removed()` → `component:remove` event

### Built-in Component Types
`default`, `text`, `textnode`, `image`, `video`, `link`, `label`, `map`, `table`, `row`, `cell`, `thead`, `tbody`, `tfoot`, `svg`, `script`, `comment`, `wrapper`

## Blocks

```js
const bm = editor.BlockManager; // also editor.Blocks

bm.add('my-block', {
  id: 'my-block',
  label: '<b>My Block</b>',
  media: '<svg>...</svg>',     // icon/thumbnail
  category: 'Basic',
  content: '<div class="my-block">Content</div>',
  // Component-oriented (recommended):
  content: { type: 'my-component', components: [{ tagName: 'span', content: 'Hello' }] },
  // Mixed:
  content: [{ type: 'image' }, '<div>Extra</div>'],
  attributes: { class: 'gjs-block-section' },
  select: true,    // select on drop
  activate: true,  // trigger active event on drop
});

bm.get('my-block');
bm.getAll();
bm.remove('my-block');
block.set({ label: 'Updated' });
```

### Block Content Best Practices
- **Prefer component-oriented** content over HTML strings
- **Don't put styles** in blocks — define them in component types via `styles`
- **Don't put functions** (like `script`) in blocks — keep in component type definitions
- Use `data-gjs-type` attributes to bind HTML elements to component types

### Block Custom UI

```js
grapesjs.init({
  blockManager: { custom: true },
});

editor.on('block:custom', ({ blocks, dragStart, dragStop, container }) => {
  // Render your custom block UI
  // Use dragStart(block) and dragStop(block) for drag & drop
});
```

## Panels & Buttons

```js
editor.Panels.addPanel({
  id: 'my-panel',
  el: '.my-panel-container',
  resizable: { maxDim: 350, minDim: 200, tc: false, cl: true, cr: false, bc: false, keyWidth: 'flex-basis' },
  buttons: [
    {
      id: 'my-btn',
      className: 'btn-class',
      label: 'Click',
      command: 'my-command', // string ID or function
      active: true,
      togglable: false,
      context: 'group-name', // group buttons
    },
  ],
});
```

## Commands

```js
// Object with run/stop (stateful)
editor.Commands.add('my-command', {
  run(editor, sender, options) { /* activate */ },
  stop(editor, sender, options) { /* deactivate */ },
});

// Simple function (not stateful)
editor.Commands.add('my-simple-cmd', editor => {
  editor.Modal.setTitle('Hello').setContent('<p>Hi</p>').open();
});

editor.runCommand('my-command', { someOpt: 1 });
editor.stopCommand('my-command');

// Check if active
editor.Commands.isActive('my-command');
editor.Commands.getActive(); // { 'cmd-id': returnValue }

// Extend existing command
editor.Commands.extend('existing-cmd', {
  someMethod() { /* override */ },
});

// Command events
editor.on('run:my-command:before', opts => { /* opts.abort = 1 to cancel */ });
editor.on('run:my-command', () => { /* after run */ });
editor.on('abort:my-command', () => { /* if aborted */ });
editor.on('command:run', commandId => { /* any command run */ });
editor.on('command:stop', commandId => { /* any command stop */ });
```

## Storage

```js
// Local storage
grapesjs.init({
  storageManager: {
    type: 'local',
    autosave: true,
    autoload: true,
    stepsBeforeSave: 1,
    options: {
      local: { key: 'gjsProject' },
    },
  },
});

// Remote storage
grapesjs.init({
  storageManager: {
    type: 'remote',
    autosave: true,
    stepsBeforeSave: 10,
    options: {
      remote: {
        headers: {},
        urlStore: 'https://api.example.com/store',
        urlLoad: 'https://api.example.com/load',
        fetchOptions: opts => (opts.method === 'POST' ? { method: 'PATCH' } : {}),
        onStore: (data, editor) => ({ id: projectID, data }),
        onLoad: result => result.data,
      },
    },
  },
});

// Disable storage
grapesjs.init({ storageManager: false });

// Manual save/load
await editor.store();
await editor.load();
const data = editor.getProjectData();
editor.loadProjectData(data);

// Define custom storage
editor.Storage.add('session', {
  async load(options = {}) {
    return JSON.parse(sessionStorage.getItem(options.key));
  },
  async store(data, options = {}) {
    sessionStorage.setItem(options.key, JSON.stringify(data));
  },
});

// Replace existing storage (e.g., with axios)
editor.Storage.add('remote', {
  async load() { return await axios.get(`projects/${id}`); },
  async store(data) { return await axios.patch(`projects/${id}`, { data }); },
});

// Skip initial load with projectData
grapesjs.init({
  projectData: existingData || { pages: [{ component: '<div>Initial</div>' }] },
  storageManager: { type: 'remote' },
});

// Store HTML/CSS with project data
grapesjs.init({
  storageManager: {
    type: 'remote',
    options: {
      remote: {
        onStore: (data, editor) => {
          const pagesHtml = editor.Pages.getAll().map(page => {
            const component = page.getMainComponent();
            return { html: editor.getHtml({ component }), css: editor.getCss({ component }) };
          });
          return { id: projectID, data, pagesHtml };
        },
        onLoad: result => result.data,
      },
    },
  },
});
```

## Devices & Responsive

```js
grapesjs.init({
  // mediaCondition: 'min-width', // for mobile-first
  deviceManager: {
    devices: [
      { name: 'Desktop', width: '' },
      { name: 'Tablet', width: '768px', widthMedia: '992px' },
      { name: 'Mobile', width: '320px', widthMedia: '480px' },
    ],
  },
});

editor.setDevice('Mobile');
editor.getDevice();
editor.on('change:device', () => console.log(editor.getDevice()));

// Mobile-first approach
grapesjs.init({
  mediaCondition: 'min-width',
  deviceManager: {
    devices: [
      { name: 'Mobile', width: '320', widthMedia: '' },
      { name: 'Desktop', width: '', widthMedia: '1024' },
    ],
  },
});
editor.setDevice('Mobile');
```

## Events

### Editor Events
- `update` - any project change
- `undo` / `redo`
- `load` - editor loaded and rendered
- `project:load` / `project:loaded`
- `project:get` - extend project data on request
- `destroy` / `destroyed`

### Component Events
- `component:create` - component created
- `component:mount` - rendered in canvas
- `component:update` / `component:update:{prop}`
- `component:remove`
- `component:selected` / `component:deselected`
- `component:toggled` - selection toggled

### Block Events
- `block:drag:stop` - block dropped
- `block:custom` - custom block UI update

### Style Events
- `style:sector:add/remove/update`
- `style:property:add/remove/update`
- `style:target` - target selection changed
- `style:custom` - for custom UI

### Trait Events
- `trait:custom` - custom trait UI update

### Canvas Events
- `canvas:spot` - canvas spot update

### Storage Events
- `storage:start` / `storage:end`
- `storage:error`

### Asset Events
- `asset:custom` - custom asset UI
- `asset:upload:start` / `asset:upload:end`
- `asset:upload:error` / `asset:upload:response`

### Layer Events
- `layer:custom` - custom layer UI
- `layer:root` - root layer changed
- `layer:component` - component layer update

### Command Events
- `command:run` / `command:stop` - any command
- `command:run:before:{id}` / `command:run:{id}` / `command:stop:{id}`

```js
editor.on('component:selected', model => {
  console.log('Selected:', model.get('type'));
});
```

## Pages

```js
const pm = editor.Pages;
pm.add({ id: 'page-2', component: '<div>Page 2</div>' });
pm.getAll();
pm.getSelected();
pm.select('page-2');
pm.remove('page-2');

// Get main component of a page
const mainCmp = pm.getSelected().getMainComponent();

// Get HTML/CSS of specific page
const component = pm.getSelected().getMainComponent();
const html = editor.getHtml({ component });
const css = editor.getCss({ component });
```

## Assets

```js
const am = editor.AssetManager; // also editor.Assets

// Configure
grapesjs.init({
  assetManager: {
    assets: [
      'https://path/to/image.jpg',
      { type: 'image', src: 'https://...', height: 350, width: 250, name: 'displayName' },
    ],
    upload: 'https://endpoint/upload/assets', // set false to disable
    uploadName: 'files',
  },
});

// Programmatic
am.add([{ category: 'c1', src: 'https://...' }]);
am.getAll();
am.getAllVisible();
am.get('https://.../img.jpg');
am.remove('https://.../img.jpg');
am.render(); // render all
am.render(am.getAll().filter(a => a.get('category') === 'c1')); // filter

// Open with custom select logic
am.open({
  types: ['image'],
  select(asset, complete) {
    const selected = editor.getSelected();
    if (selected && selected.is('image')) {
      selected.addAttributes({ src: asset.getSrc() });
      complete && am.close();
    }
  },
});

// Upload events
editor.on('asset:upload:start', () => { /* loading animation */ });
editor.on('asset:upload:end', () => { /* end animation */ });
editor.on('asset:upload:error', err => { /* handle error */ });
editor.on('asset:upload:response', response => { /* handle response */ });

// Custom Asset Manager UI
grapesjs.init({ assetManager: { custom: true } });
editor.on('asset:custom', ({ open, assets, types, close, remove, select, container }) => {
  // Render your custom asset UI
});
```

## Layers

```js
// Configuration
grapesjs.init({
  layerManager: {
    appendTo: '.layers-container',
    root: '#my-custom-root', // optional
    sortable: true,
    hidable: true,
  },
});

// Layers API
const lm = editor.Layers;
lm.getLayerData(component); // { name, components, visible, open, selected, hovered }
lm.setVisible(component, false);
lm.setOpen(component, true);
lm.setName(component, 'New Name');
lm.setLayerData(component, { hovered: true });
lm.setLayerData(component, { selected: true }, { event });

// Custom Layer UI
grapesjs.init({ layerManager: { custom: true } });
editor.on('layer:custom', ({ container, root }) => { /* mount your UI */ });
editor.on('layer:root', root => { /* root layer changed */ });
editor.on('layer:component', component => { /* update specific layer */ });
```

## Modal

```js
const md = editor.Modal;

md.setTitle('Title');
md.setContent('<p>Content</p>');
md.open();
md.close();
md.isOpen();
md.getContentEl(); // HTMLElement for content

md.onceClose(() => { /* on close */ });

// Open with options
md.open({
  title: 'My Modal',
  content: '<p>My content</p>',
});
```

## Selectors

```js
// Configuration
grapesjs.init({
  selectorManager: {
    appendTo: '.styles-container',
    componentFirst: true, // style component directly, not shared class
  },
});

const sm = editor.SelectorManager;
// Manage selectors/classes on selected component
```

## Component CSS

### Component-scoped styles (in type definition)
```js
editor.DomComponents.addType('my-card', {
  model: {
    defaults: {
      attributes: { class: 'card' },
      styles: `
        .card { border: 1px solid #ddd; border-radius: 8px; padding: 16px; }
        @media (max-width: 768px) { .card { padding: 8px; } }
      `,
    },
  },
});
```

Styles are grouped per component type and auto-removed when all instances are removed.

**Best practice:** Follow component-oriented styling — declare styles only in the component that owns them:

```js
// BAD: Parent declares styles for children
domc.addType('parent', {
  model: {
    defaults: {
      styles: `.child-a { color: green } .child-b { color: blue }`,
      components: [{ type: 'child-a' }, { type: 'child-b' }],
    },
  },
});

// GOOD: Each component owns its own styles
domc.addType('child-a', {
  model: { defaults: { attributes: { class: 'child-a' }, styles: `.child-a { color: green }` } },
});
domc.addType('child-b', {
  model: { defaults: { attributes: { class: 'child-b' }, styles: `.child-b { color: blue }` } },
});
```

### Add CSS rules programmatically
```js
editor.addStyle('.my-class { color: red; font-size: 14px; }');
editor.setStyle('.my-class { color: blue; }'); // replaces all styles
```

## Data Sources

```js
const dsm = editor.DataSourceManager;

// Add a data source
dsm.add({
  id: 'my-datasource',
  records: [{ id: '1', name: 'Item 1' }, { id: '2', name: 'Item 2' }],
});

// Get/remove data sources
dsm.get('my-datasource');
dsm.getAll();
dsm.remove('my-datasource');
```
