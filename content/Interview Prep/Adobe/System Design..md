---
{"publish":true,"created":"2026-04-28T19:50:00.923Z","modified":"2026-08-17T03:02:33.327Z"}
---

Ways to sandbox:

- **Iframes**:

- **Web Workers**: good for compute isolation, but they do not render UI directly.

- **Process isolation**: strongest boundary, better crash/security isolation, but higher overhead and more complex IPC.[](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-label)

- **JS VM / sandboxed runtime**: code runs in a controlled execution environment with limited globals; useful when you want to restrict capabilities without full browser-frame embedding.[](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/82070338/0c11fcfd-78f7-44b2-a459-adbe91db6d1a/image-4.jpg)

- **Webview / embedded web container**: similar family to iframe but can be host-managed and more controlled depending on platform.[](https://eye-able.com/sv/blog/aria-label)

- **Shadow DOM**: good for style encapsulation, but **not** a real security isolation boundary.
