# Zync plugin UI (beta)

Framework-neutral, dependency-free UI primitives for isolated Zync plugin panes. Works with vanilla JavaScript or framework wrappers. It does not import Zync internals, fetch remote assets, inject untrusted HTML, grant permissions, or run SSH commands.

## Install and bundle

```sh
npm install @zync-sh/plugin-ui@beta
npm install --save-dev @zync-sh/plugin-sdk@beta
```

Before publication, install the local packages by path instead. The UI package and SDK have separate versions and release lifecycles. This initial UI beta provides theme tokens, buttons, labelled inputs, badges, empty states, dropdowns and tooltips; it is not a full React component library.

```js
import { installThemeBridge, enhanceSelects, installTooltips, createButton } from '@zync-sh/plugin-ui';
import '@zync-sh/plugin-ui/styles.css';

document.body.classList.add('zui-root');
const disposeTheme = installThemeBridge();
const disposeSelects = enhanceSelects();
const disposeTooltips = installTooltips();
document.body.append(createButton({ text: 'Refresh', onClick: () => refreshYourPlugin() }));
// On component teardown: disposeTheme(); disposeSelects(); disposeTooltips();
```

Bundle the JavaScript and CSS into your package. Zync 2.33.8 and newer support
package-relative pane assets such as `./pane.js` and `./pane.css`; use Vite
`base: './'` or the equivalent in your bundler. Inline bundles remain an option
within the pane HTML size limit. Do not load dependencies from CDNs or leave
root-relative asset paths. The basic SDK starter copies files and needs an added
bundling step before it can process npm or CSS imports.

## Theme contract

`installThemeBridge` accepts `zync:theme:update` messages only from the parent window by default. Unknown keys and invalid CSS colors are ignored. Tokens are `--zui-background`, `--zui-surface`, `--zui-border`, `--zui-text`, `--zui-muted`, `--zui-primary`, plus `--zui-positive`, `--zui-negative`, `--zui-warning`, `--zui-info`. Light/dark mode updates semantic colors and native control color scheme. Dark fallback tokens work before the first host message.

Put `zui-root` on the pane body or application container for base controls and slim scrollbars. Built primitives use `zui-*` classes; popup classes are styled independently so they work outside a scrolling container. CSS is scoped to these classes, apart from token defaults on `:root`.

## Controls

- `createButton({text, variant, ariaLabel, onClick})`: `default`, `primary`, `danger`, `ghost` variants. Use `ariaLabel` for icon-only buttons.
- `createField({label, type, value, placeholder})`: returns `{field, input}` with a visible label. Append `field`; read `input.value`.
- `createBadge(text, tone)`: `muted`, `positive`, `negative`, `warning`, `info` tones.
- `createEmptyState({title, description, actions})`: labelled section; actions are button elements.
- `enhanceSelects(root, {closeEvent})`: native single-select stays the source of truth. Supports arrows, Home/End, typeahead, Escape and Tab; skips disabled/hidden options and optgroups. Dispatch `change` after setting `.value` programmatically. Multiple selects remain native. Repeated installation skips existing instances; call the returned disposer to restore native controls and disconnect observers/listeners.
- `installTooltips(root)`: delegated `data-tooltip` hover/focus tooltips. Preserves existing `aria-describedby`, hides detached targets and supports Escape. Call its disposer before reinstalling on the same root.

These controls do not create privileged confirmation dialogs. Use the SDK's host-provided API for permissions and dangerous actions. Theme integration is visual, not a security boundary. Test your actual host build and light/dark/custom themes before distributing a plugin.

From this package's own checkout, run `npm run type-check`, `npm test`, and
`npm pack --dry-run` before releasing it. Test consuming plugins in the installed
app as well. Publish prereleases under `beta`, not stable. Beta checks are not an
independent security audit. Plugin authors should follow the community
[best practices](https://zync.thesudoer.in/docs/plugin-best-practices/) and
[publishing guide](https://zync.thesudoer.in/docs/plugin-publishing/).
