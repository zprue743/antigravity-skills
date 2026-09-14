# Vue and TypeScript integration

Use for Vue applications. Confirm Vue, Bryntum core, wrapper, and declaration versions.

For Vue 3 full bundles, the wrapper is `@bryntum/schedulerpro-vue-3`; native classes come from `@bryntum/schedulerpro`. Follow the installed distribution's packaging for thin/trial variants.

The wrapper and native SchedulerPro instance differ. Access the native instance after mount using the installed wrapper's declared reference shape. Ref unwrapping varies; do not add `.value` blindly.

Keep native widgets, projects, stores, and models out of deep Vue proxying. Reactive configuration/DTOs and live Bryntum instances have different ownership. Hold native instances in non-reactive storage or appropriate shallow/raw boundaries, preserving identity.

Update through supported props or native APIs. Avoid deep watchers that echo changes into full data reloads. Check for duplicate relayed store notifications before wiring persistence.

Use shipped configuration types and typed custom fields. Do not hide API mismatches with broad `any` casts. Remove application-owned listeners; let the wrapper manage its widget. Verify shared-project ownership before destruction.

Source: [official Vue integration guide](https://bryntum.com/products/schedulerpro/docs/guide/SchedulerPro/integration/vue/guide).
