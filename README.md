# Dependency Confusion Example with `electron-panel-window`

## Introduction

While auditing an Electron app for vulnerabilities, I discovered an interesting case of dependency confusion. The package `electron-panel-window` was susceptible to this type of attack, and I successfully claimed it. The npm security team has since removed this package due to the associated risks.

This post will explain what dependency confusion is, how it can be reproduced, and provide a sample code to demonstrate taking over an npm package.

## What is Dependency Confusion?

Dependency confusion, also known as a supply chain attack, occurs when a malicious actor publishes a package to a public registry (like npm) with the same name as an internal package used by an organization. When the package manager resolves dependencies, it may mistakenly fetch the public, potentially malicious package instead of the intended internal one. This can lead to significant security breaches.

## My Discovery

While looking for vulnerabilities in an Electron app, I stumbled upon a confusion between two packages: [electron-panel-window](https://www.npmjs.com/package/electron-panel-window) and [@time-loop/electron-panel-window](https://www.npmjs.com/package/@time-loop/electron-panel-window). The developers had changed their organization from `@time-loop` to `@goabstract`, leaving the original package name unclaimed. This confusion allowed me to claim the [@time-loop/electron-panel-window](https://www.npmjs.com/package/@time-loop/electron-panel-window) package and demonstrate the risks involved.

## How Dependency Confusion Can Be Reproduced

### Steps to Reproduce

1. **Identify Internal Packages**: First, identify the names of packages used internally by a target organization. In my case the package was `@time-loop/electron-panel-window`

2. **Publish a Malicious Package**: Create a malicious package with the same name as the internal package and publish it to the public registry. (target organization name sholud be available in npm. ex: time-loop)

3. **Trigger Dependency Resolution**: When the target organization or its users install dependencies, the package manager may mistakenly fetch the malicious package instead of the internal one.

### Sample Code: Taking Over a Package


```json
// package.json
{
  "name": "electron-panel-window",
  "version": "1.0.0",
  "description": "Malicious package mimicking internal electron-panel-window",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "Malicious Actor",
  "license": "ISC"
}
```
```js

// index.js
console.log("Malicious code executed!");
// Add any malicious payload here   
```

#### 2. Publish the Package to npm

```
npm publish
```

#### 3. Dependency Confusion in Action
When someone runs npm install in a project that lists electron-panel-window as a dependency, npm will fetch the publicly available malicious package instead of the intended internal one.

## Preventing Dependency Confusion
To mitigate the risk of dependency confusion, consider the following best practices:

1) **Namespace Your Packages**: Use scoped packages (e.g., @myorg/package) to reduce the likelihood of naming conflicts.
2) **Verify Sources**: Configure your package manager to only fetch packages from trusted sources or private registries.
3) **Use Lockfiles**: Always use lockfiles (e.g., package-lock.json) to ensure consistent dependency resolution.
4) **Monitor Dependencies**: Regularly audit your dependencies for unexpected changes or updates.
   
## Conclusion
After I took over the electron-panel-window package, the next day, the npm security team discovered that the package contained malicious code. My intention was never to hack anyone; I wanted to highlight the vulnerability and report it. However, before I could report it, the npm security team had already identified the issue and removed the malicious package from the registry.

![image](https://github.com/rakesh0x7/electron-panel-window/assets/60481830/351758d3-032f-4ac6-a3a1-cdd2983d883f)

## References
- https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610
