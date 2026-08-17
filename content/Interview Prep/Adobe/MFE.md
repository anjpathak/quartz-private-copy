---
{"publish":true,"created":"2026-05-31T11:13:21.150Z","modified":"2026-08-17T03:02:36.661Z"}
---

## 1) Shared dependencies

In Module Federation, a shared dependency means the host and remotes can reuse the same library instance at runtime instead of shipping separate copies. This is useful for things like React, React DOM, router libraries, or utility libraries such as Lodash, because loading two copies can cause bugs, waste bundle size, and break shared state or context.\[[blog.logrocket](https://blog.logrocket.com/solving-micro-frontend-challenges-module-federation/)]

If App1 and App2 both declare Lodash as shared, the federation runtime tries to use one compatible version. If one app wants a different version, you have a few options: make it non-singleton and allow each app to bundle its own copy, keep the version range compatible with `requiredVersion`, or decide that the library must be duplicated because the versions are incompatible. The interview-safe answer is: **shared dependencies are not magic; they work only when the versions are compatible enough for the runtime contract**.\[[angulararchitects](https://www.angulararchitects.io/en/aktuelles/getting-out-of-version-mismatch-hell-with-module-federation/)]

A good way to explain the version conflict process is:

- First, define the shared library as a shared dependency in the shell and remotes.

- If both apps can work with one version, use that shared singleton.

- If App2 truly needs a newer incompatible version, do not force a shared singleton blindly.

- Either isolate that dependency per app, or upgrade all consumers in a coordinated release.\[[blog.logrocket](https://blog.logrocket.com/solving-micro-frontend-challenges-module-federation/)]

Yes, individual apps can absolutely have their own independent CSS strategy. App1 can use Tailwind while App2 uses SCSS, because styling is just part of each app’s build pipeline. The real concern is not whether they can coexist, but whether their styles leak into each other; that is handled with CSS scoping, naming conventions, CSS Modules, Tailwind prefixes, or other isolation techniques.\[[stackoverflow](https://stackoverflow.com/questions/76967231/tailwind-not-working-when-components-are-shared-through-webpack-module-federatio)]

## 2) What microfrontends really mean

Your understanding is correct: microfrontends are independently owned, self-contained frontend apps that come together under one shell or host experience. Module Federation is one way to implement that by loading compiled modules from another app at runtime, which means the apps do not need to be built together into one bundle.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/ca3d8798-fb86-4e01-b360-f4738f057dea/Round-3-2.txt)]\[[docs.webpack.js](https://docs.webpack.js.org/guides/module-federation)]

But Module Federation is not the only way to do microfrontends. Other approaches include:

- iframe-based composition.

- Web Components.

- Server-side composition or edge composition.

- Build-time integration through shared packages or monorepo libraries.\[[linkedin](https://www.linkedin.com/posts/aravindkumarbysani_frontend-checklist-activity-7420087893253054465-3E25)]

Not all microfrontend approaches work the same way. Some are runtime-integrated, like Module Federation or iframes. Others are more build-time or server-composed. So it is not correct to assume that all microfrontend implementations communicate only at runtime in the same way; the integration point depends on the architecture.\[[docs.webpack.js](https://docs.webpack.js.org/guides/module-federation)]

## 3) Microfrontend vs monolith

Your assumption is correct: team structure is a major factor. If one team owns the whole product and the codebase is small or medium, a monolith is usually simpler and often faster. If multiple teams need to release independently, own separate feature areas, and move at different speeds, microfrontends become more attractive.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]\[[linkedin](https://www.linkedin.com/posts/aravindkumarbysani_frontend-checklist-activity-7420087893253054465-3E25)]

Other major factors are:

- Release frequency.

- Need for independent deployment.

- Performance budget.

- Degree of UI consistency required.

- Operational complexity your org can support.

- Need for tech-stack flexibility.

- Long-term scaling of the codebase and teams.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]\[[linkedin](https://www.linkedin.com/posts/aravindkumarbysani_frontend-checklist-activity-7420087893253054465-3E25)]

A good interview answer is: choose microfrontends when **organizational scale** is the main problem; choose monolith when **simplicity and performance** are the main problem. If you are starting from scratch but expect multiple teams later, you might still start with a modular monolith and move to microfrontends only when the ownership and release boundaries become real.\[[linkedin](https://www.linkedin.com/posts/aravindkumarbysani_frontend-checklist-activity-7420087893253054465-3E25)]\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## 4) Preventing one app from breaking another

This is one of the biggest reasons MFEs are hard. You prevent breakage by controlling the contract between host and remotes, pinning compatible shared dependency versions, testing integration points, and adding fallback behavior when a remote fails to load.\[[angulararchitects](https://www.angulararchitects.io/en/aktuelles/getting-out-of-version-mismatch-hell-with-module-federation/)]\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

The practical methods are:

- Keep APIs and props contracts versioned.

- Avoid breaking changes in shared libraries unless all consumers are updated together.

- Use `requiredVersion` and version ranges carefully.

- Treat shared dependencies like a compatibility contract, not just a package install detail.

- Add error boundaries, loading fallbacks, and remote health checks.\[[blog.logrocket](https://blog.logrocket.com/solving-micro-frontend-challenges-module-federation/)]

If a remote app changes its props contract and the host breaks, that means the runtime contract was not protected well enough. If two MFEs depend on different React versions, you either need to coordinate on one compatible version or avoid sharing React as a singleton if compatibility cannot be guaranteed. If a remote bundle fails due to CDN/network issues, the host should degrade gracefully rather than taking down the whole page.\[[angulararchitects](https://www.angulararchitects.io/en/aktuelles/getting-out-of-version-mismatch-hell-with-module-federation/)]

## 5) Ownership across teams

Ownership is usually divided by feature domain, not by technical layer. For example, one team might own navigation, another own analytics, another own editor tools, and another own shared platform services. The platform team typically owns the shell, shared infrastructure, design system, federation config, and deployment standards.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/ca3d8798-fb86-4e01-b360-f4738f057dea/Round-3-2.txt)]

This works best when each team owns:

- Its remote app code.

- Its tests and CI/CD pipeline.

- Its feature-specific API contracts.

- Its release cadence within agreed platform rules.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

The platform or core frontend team owns:

- The host shell.

- Shared libraries and design system.

- Federation version policies.

- Performance and security guardrails.

- Cross-app communication patterns.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## 6) Real-world scenarios

## Host loading remotes

The host typically loads a remote entry at runtime and then imports exposed modules from that remote. This gives independent deployment but means the host must handle loading states and remote failures.\[[docs.webpack.js](https://docs.webpack.js.org/guides/module-federation)]\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## Remote communication

Remotes can communicate through shared state, custom events, a global store, or a host-mediated API. For simple cases, event-based communication works; for stronger consistency, a shared store or platform event bus is better.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/ca3d8798-fb86-4e01-b360-f4738f057dea/Round-3-2.txt)]

## Avoid loading React twice

React should usually be shared as a singleton in federation so both host and remotes use the same instance. If React is loaded twice, hooks, context, and rendering assumptions can break.\[[docs.webpack.js](https://docs.webpack.js.org/guides/module-federation)]

## Remote failure

If a remote fails to load, the host should show a fallback UI, retry if appropriate, or hide that feature. The goal is that one broken feature should not crash the full app.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## Version mismatch

If a remote and host are out of sync, you need version policies, compatibility testing, and sometimes a gradual rollout strategy. In bad cases, the platform should refuse to mount an incompatible remote rather than failing silently.\[[blog.logrocket](https://blog.logrocket.com/solving-micro-frontend-challenges-module-federation/)]

## Performance control

To keep performance acceptable, use lazy loading, bundle splitting, code sharing, route-based loading, and caching. Avoid eagerly loading everything at startup; load only what the user needs.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/ca3d8798-fb86-4e01-b360-f4738f057dea/Round-3-2.txt)]

## Shared state

If the same auth, theme, or feature flag must be seen across MFEs, use a shared singleton store or platform-level state service. Browser storage can be used for persistence, but not as the only real-time synchronization mechanism.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## Gradual rollout

Use feature flags in the host or shared platform layer, then let MFEs read the flag state through a shared contract. That gives you controlled rollout without redeploying all teams at once.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/4649a831-32e7-443c-8c19-b2e5d0aeedea/Round-2.txt)]

## Faster independent deployment

Microfrontends help when one team needs to deploy faster than others. The shell stays stable while the remote updates independently, as long as the contract remains compatible.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## Shared UI library changes

If a shared UI library changes, treat it like an API change. Version it, test all consumers, and avoid breaking the world with one release.\[[angulararchitects](https://www.angulararchitects.io/en/aktuelles/getting-out-of-version-mismatch-hell-with-module-federation/)]

## 7) MFE vs plugin platform

Your final point is important: microfrontends and plugins are not the same trust model. In an MFE system, the modules usually belong to the same company or trusted product ecosystem, so they can share more infrastructure and have fewer security boundaries. In a plugin system, the plugin can be third-party or less trusted, so it needs sandboxing, permission control, and host-mediated APIs.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

So the real difference is:

- **Microfrontend**: mostly an organizational and delivery boundary.

- **Plugin**: a security and extensibility boundary.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

That is why MFEs can often use direct runtime composition and shared state, while plugins usually need iframe isolation, `postMessage`, signed manifests, and capability-based access.\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

## 8) When not to use microfrontends

Do not use microfrontends when the product is small, the team is small, performance is extremely sensitive, or the app needs very tight UI consistency. If the runtime complexity outweighs the organizational benefit, a monolith or modular monolith is usually the better choice.\[[docs.webpack.js](https://docs.webpack.js.org/guides/module-federation)]\[[ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/82070338/b13b6fc8-7a94-418d-820e-d2263d681bbd/Round-4-3.txt)]

A simple rule is: if your main problem is code scale, microfrontends may help; if your main problem is product simplicity and speed, microfrontends may hurt.\[[docs.webpack.js](https://docs.webpack.js.org/guides/module-federation)]

I can turn this into a **one-page interview answer sheet** next, with:

- “how to answer in 30 seconds,”

- “how to answer in 2 minutes,”

- and “follow-up traps the interviewer may ask.”
