---
name: grapesjs
description: GrapesJS web builder framework - API usage and style customization. Use when working with GrapesJS editor initialization, component creation, block management, style manager configuration, theming, CSS custom properties, custom component types, plugins, storage, panels, commands, traits, canvas, layers, assets, responsive design, modal, pages, data sources, i18n, rich text editor, keymaps, undo manager, or any task involving building/customizing a drag-and-drop page builder with GrapesJS.
---

# GrapesJS Skill

## Quick Start

Initialize the editor:

```js
import 'grapesjs/dist/css/grapes.min.css';
import grapesjs from 'grapesjs';

const editor = grapesjs.init({
  container: '#gjs',
  fromElement: true,
  storageManager: false,
  plugins: [],
});
```

## Core API Modules

Access modules from the editor instance:

```js
editor.DomComponents   // Component management (alias: editor.Components)
editor.BlockManager     // Block management (alias: editor.Blocks)
editor.StyleManager     // Style properties
editor.Panels           // UI panels
editor.Commands         // Command system
editor.Canvas           // Canvas control
editor.StorageManager   // Data persistence (alias: editor.Storage)
editor.DeviceManager    // Responsive devices
editor.SelectorManager  // CSS selectors/classes
editor.Modal            // Modal dialogs
editor.Pages            // Multi-page support
editor.I18n             // Internationalization
editor.AssetManager     // Asset/media management (alias: editor.Assets)
editor.LayerManager     // Layer tree (alias: editor.Layers)
editor.TraitManager     // Trait/settings (alias: editor.Traits)
editor.RichTextEditor   // RTE customization
editor.Keymaps          // Keyboard shortcuts
editor.UndoManager      // Undo/Redo
editor.Parser           // HTML/CSS parser
```

## Key Editor Methods

```js
editor.getHtml()              // Get HTML output
editor.getCss()               // Get CSS output
editor.getJs()                // Get JS output
editor.getProjectData()       // Get full project JSON
editor.loadProjectData(json)  // Load project from JSON
editor.getWrapper()           // Root component
editor.getSelected()          // Last selected component
editor.getSelectedAll()       // All selected components
editor.select(component)      // Select a component
editor.selectAdd(component)   // Add to selection
editor.selectRemove(component)// Remove from selection
editor.addComponents(html)    // Add components
editor.setComponents(html)    // Replace components
editor.addStyle(css)          // Add CSS rules
editor.setStyle(css)          // Replace all CSS rules
editor.on(event, callback)    // Listen to events
editor.off(event, callback)   // Remove listener
editor.once(event, callback)  // Listen once
editor.runCommand(id, opts)   // Run command
editor.stopCommand(id, opts)  // Stop command
editor.store()                // Save to storage
editor.load()                 // Load from storage
editor.getDevice()            // Get current device name
editor.setDevice('Mobile')    // Switch device
editor.getDirtyCount()        // Unsaved changes count
editor.clearDirtyCount()      // Reset changes counter
editor.getContainer()         // Editor container HTMLElement
editor.onReady(callback)      // Run when editor is fully loaded
editor.destroy()              // Destroy editor
editor.refresh()              // Update editor dimensions
editor.setCustomRte(obj)      // Replace built-in RTE
editor.setDragMode(mode)      // 'absolute' | 'translate'
```

## Plugins

Plugins are functions executed on editor init. **Always define custom component types inside plugins** (necessary so types exist before templates are loaded from storage).

```js
const myPlugin = (editor, options) => {
  editor.DomComponents.addType('my-type', { /* ... */ });
  editor.BlockManager.add('my-block', { /* ... */ });
};

grapesjs.init({
  plugins: [myPlugin],
  pluginsOpts: { [myPlugin]: { opt1: 'value' } },
});
```

### TypeScript Usage

```ts
import grapesjs, { usePlugin } from 'grapesjs';
import type { Plugin } from 'grapesjs';

interface MyPluginOptions { opt1: string; opt2?: number; }

const myPlugin: Plugin<MyPluginOptions> = (editor, options) => { /* ... */ };

grapesjs.init({
  plugins: [usePlugin(myPlugin, { opt1: 'A', opt2: 1 })],
});
```

## Default Commands

Built-in commands use `core:*` namespace:

| Command | Description |
|---|---|
| `core:canvas-clear` | Clear all content |
| `core:component-delete` | Delete selected |
| `core:component-enter` | Select first child |
| `core:component-exit` | Select parent |
| `core:component-next` | Select next sibling |
| `core:component-prev` | Select previous sibling |
| `core:component-outline` | Toggle outline borders |
| `core:component-offset` | Show margins/paddings |
| `core:component-select` | Enable canvas selection |
| `core:copy` / `core:paste` | Copy/paste component |
| `core:preview` | Preview mode |
| `core:fullscreen` | Fullscreen mode |
| `core:open-code` | Open code panel |
| `core:open-layers` | Open layers panel |
| `core:open-styles` | Open style manager |
| `core:open-traits` | Open trait manager |
| `core:open-blocks` | Open blocks panel |
| `core:open-assets` | Open asset manager |
| `core:undo` / `core:redo` | Undo/Redo |

## Editor Init Config Reference

Key `grapesjs.init({...})` options:

```js
grapesjs.init({
  container: '#gjs',           // Selector or HTMLElement
  fromElement: false,          // Load content from container
  height: '100%',              // Editor height
  width: 'auto',               // Editor width
  projectData: {},             // Initial project data (skips storage load)
  mediaCondition: 'max-width', // or 'min-width' for mobile-first
  plugins: [],
  pluginsOpts: {},
  panels: { defaults: [] },
  blockManager: { blocks: [], appendTo: '.container' },
  layerManager: { appendTo: '.container' },
  styleManager: { appendTo: '.container', sectors: [] },
  selectorManager: { appendTo: '.container', componentFirst: false },
  traitManager: { appendTo: '.container' },
  storageManager: { type: 'local', autosave: true, autoload: true },
  deviceManager: { devices: [] },
  assetManager: { assets: [], upload: false },
  canvas: {},
});
```

## Detailed References

- **API Reference** (components, blocks, commands, storage, events, traits, canvas, assets, layers, pages): See [references/api-reference.md](references/api-reference.md)
- **Style Customization** (Style Manager, theming, CSS variables, custom types): See [references/style-customization.md](references/style-customization.md)
- **Traits & Settings** (built-in types, custom types, custom UI, categories, i18n): See [references/traits-reference.md](references/traits-reference.md)
- **Canvas & Spots** (canvas spots, custom overlays, disabling built-in spots): See [references/canvas-reference.md](references/canvas-reference.md)
- **Storage Deep Dive** (local, remote, custom storage, inline, project data): See [references/storage-reference.md](references/storage-reference.md)
- **Commands Deep Dive** (stateful commands, extending, events, aborting): See [references/commands-reference.md](references/commands-reference.md)
- **UI Layout Patterns** (panels, layers, sidebar, switchers): See [references/ui-layout-patterns.md](references/ui-layout-patterns.md)
- **MJML Plugin** (email builder, MJML components, export MJML/HTML, custom components, preMjml/postMjml): See [references/mjml-plugin-reference.md](references/mjml-plugin-reference.md)
