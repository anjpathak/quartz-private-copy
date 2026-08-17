---
{"publish":true,"created":"2025-12-23T09:03:03.000Z","modified":"2026-08-17T03:04:13.087Z"}
---

## Preparation Tasks

Run linting and formatting to enforce code standards (e.g., ESLint, Prettier).[](https://dev.to/flippedcoding/the-developer-s-deployment-checklist-3p5p)​\
Update dependencies and clear caches to avoid inconsistencies.[](https://syncromsp.com/blog/software-deployment-checklist/)​\
Verify environment variables for production (e.g., API URLs, keys in .env.prod).[](https://www.spaceotechnologies.com/blog/frontend-development-process/)​

## Build Process

Execute the build command (e.g., `npm run build` for Vite/Webpack) to bundle, transpile (TypeScript/JSX), minify, and tree-shake code.[](https://blog.logrocket.com/best-practices-ci-cd-pipeline-frontend/)​\
Process assets like images/CSS for optimization and generate source maps if needed.[](https://buddy.works/guides/how-build-and-deploy-frontend-applications)​\
For frameworks like React/Angular, ensure code splitting and lazy loading are configured.[](https://deploybot.com/blog/front-end-deployment-of-a-react-app)​

## Testing Phase

Run unit/integration tests (e.g., Jest, Cypress) on the built artifacts.[](https://dev.to/flippedcoding/the-developer-s-deployment-checklist-3p5p)​\
Perform end-to-end tests, accessibility checks (e.g., Lighthouse), and cross-browser validation.[](https://vercel.com/docs/production-checklist)​\
Conduct manual QA on staging to catch UI regressions or performance issues.[](https://syncromsp.com/blog/software-deployment-checklist/)​

## Security & Optimization

Scan for vulnerabilities (e.g., npm audit, Snyk) and implement CSP headers.[](https://docs.netlify.com/resources/checklists/production-checklist/)​\
Optimize performance: Compress assets, enable caching, and set up HTTPS.[](https://hungvu.tech/considerations-for-a-production-ready-website)​\
Review bundle size with tools like Webpack Bundle Analyzer.[](https://blog.pixelfreestudio.com/how-to-optimize-frontend-builds-with-devops-practices/)​

## Deployment Execution

Push artifacts to remote server/CDN (e.g., Vercel/Netlify CLI: `vercel deploy --prod`).[](https://create-react-app.dev/docs/deployment/)​\
Clear remote caches and restart services if using custom servers (e.g., Nginx).[](https://dev.to/caroso1222/deploying-frontend-applications-the-fun-way-1fpf)​\
Tag the release in Git and notify team via Slack/email.[](https://syncromsp.com/blog/software-deployment-checklist/)​

Frontend build tools (e.g., Webpack, Vite, Parcel) primarily handle transformation and optimization of source code into deployable assets during the build phase, but their scope is limited to local/automated compilation tasks. They automate bundling and processing but do not cover testing, security scanning, or server deployment. Boundaries are defined by their focus on code/assets rather than runtime operations or infrastructure.

## Tasks Handled by Build Tools

Build tools execute core compilation via commands like `npm run build`.[](https://blog.logrocket.com/best-practices-ci-cd-pipeline-frontend/)​

- Transpile modern JS/TS (Babel/esbuild), process CSS/Sass, and handle JSX/Vue templates.[](https://pieces.app/blog/vite-vs-webpack-which-build-tool-is-right-for-your-project)​

- Bundle modules, resolve dependencies, tree-shake unused code, and enable code splitting.[](https://www.uplers.com/blog/5-best-task-runner-module-bundler-front-end-development-tools/)​

- Minify/uglify code, optimize images/fonts, and generate source maps for debugging.[](https://blog.pixelfreestudio.com/how-to-optimize-frontend-builds-with-devops-practices/)​

- Support dev server features like HMR and asset hashing for caching.[](https://jsdevspace.substack.com/p/vite-vs-webpack-a-guide-to-choosing)​

## Tasks Outside Build Tools

These require separate tools, scripts, or manual processes integrated via CI/CD (e.g., GitHub Actions, Jenkins).[](https://www.linkedin.com/pulse/cicd-front-end-developers-automating-deployment-pipelines-potta-7ngyc)​

- Linting/formatting (ESLint/Prettier), unit/integration tests (Jest/Cypress), and E2E/accessibility checks (Lighthouse).[](https://zeet.co/blog/deployment-pipeline)​

- Dependency audits (npm audit/Snyk), environment config validation, and bundle analysis (Webpack Bundle Analyzer).[](https://blog.pixelfreestudio.com/how-to-optimize-frontend-builds-with-devops-practices/)​

- Actual deployment (e.g., `vercel deploy`, rsync to server/CDN), cache invalidation, HTTPS setup, and monitoring (New Relic).[](https://57blocks.com/blog/our-guide-to-modern-front-end-build-pipelines)​

- Manual QA on staging, Git tagging, team notifications, and post-deploy rollbacks.[](https://blog.logrocket.com/best-practices-ci-cd-pipeline-frontend/)​

## Integration in Pipeline

Build tools output artifacts (e.g., `/dist` folder) consumed by CI/CD for subsequent steps.[](https://57blocks.com/blog/our-guide-to-modern-front-end-build-pipelines)​

Build tools like Webpack and Vite do not perform lint analysis by default; linting is handled by dedicated tools like ESLint or Prettier as a separate step. They can integrate linting via plugins (e.g., `eslint-webpack-plugin` for Webpack, `vite-plugin-eslint` for Vite) to run it during dev/build processes.[](https://www.robinwieruch.de/vite-eslint/)​

## Default Behavior

Core build tools focus on bundling, transpiling (Babel/esbuild), minification, and asset optimization without built-in linting.[](https://www.meerako.com/blogs/frontend-build-tools-vite-vs-webpack-turbopack-comparison)​\
Linting requires explicit setup, often via npm scripts like `npm run lint` run before `npm run build`.[](https://www.dhiwise.com/post/the-ultimate-guide-to-integrating-eslint-with-vite)​

## Integration Options

- **Webpack**: Add `eslint-webpack-plugin` to run ESLint during bundling, catching issues early.[](https://dev.to/shivampawar/optimizing-your-react-app-a-guide-to-production-ready-setup-with-webpack-typescript-eslint-and-prettier-2024-4lcl)​

- **Vite**: Use `vite-plugin-eslint()` in `vite.config.js` for on-save linting in dev server or build-time checks.[](https://stackoverflow.com/questions/69842785/how-can-i-integrate-eslint-in-a-vite-react-project)​\
  Plugins make linting part of the workflow but keep it configurable and non-core.[](https://mill-build.org/blog/8-what-is-a-build-tool.html)​

## Best Practices

Run linting independently in CI/CD pipelines (e.g., pre-build stage) for fast failure and parallelization with tests.[](https://www.reddit.com/r/devops/comments/wlpfmm/what_comes_first_in_ci_workflow_linters_or/)​\
Separate linting ensures build tools remain lightweight for compilation
