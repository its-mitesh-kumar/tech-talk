# Scaling React with Plugins: Build a Backstage-Style Architecture That Powers Open Source Platforms

## Agenda

1.  **The Problem:** Scaling React Applications
2.  **Overview of Backstage.io: A Plugin-Powered Platform**
3.  **Architectural Choices:** Monolith vs. Micro-frontends vs. Plugins
4.  **Why Plugin-Based Architectures was the right choice for Backstage**
5.  **Core Concepts of a Plugin System**
    * Host Application (Core Shell)
    * Plugin Interface & Contracts
    * Plugin Discovery & Loading
    * Shared Services & Context
6.  **Building Your Own Backstage-Style System in React (Inspired by Backstage.io)**
    * Defining Plugin APIs & Manifests
    * Dynamic Plugin Loading
    * Routing Strategies
    * UI/Component System for Plugins
    * State Management & Data Fetching
7.  **Conclusion & Key Takeaways**
8.  **Q&A**

---

## 1. The Problem: Scaling React Applications

* **React gives flexibility—but no structure at scale.**
    ```
    +----------------------------------------+
    |     Scaling React: The Challenges      |
    |----------------------------------------|
    | +---------------------+  +---------------------+ |
    | | Monolithic Frontend |  |   Slow Build Times  | |
    | +---------------------+  +---------------------+ |
    | +---------------------+  +---------------------+ |
    | |  Team Bottlenecks   |  |Difficulty in Isolation| |
    | +---------------------+  +---------------------+ |
    | +---------------------+  +---------------------+ |
    | | Technology Lock-in  |  |Onboarding Challenges| |
    | +---------------------+  +---------------------+ |
    +----------------------------------------+
    ```
* **Monolithic Frontend:** As features grow, the codebase becomes large, complex, and hard to manage.
* **Slow Build Times:** Large applications take longer to build and deploy.
* **Team Bottlenecks:** Multiple teams working on the same codebase can lead to conflicts and slower development cycles.
* **Difficulty in Isolation:** Bugs in one part of the application can affect others.
* **Technology Lock-in:** Hard to adopt new technologies or approaches for different parts of the application.
* **Onboarding Challenges:** New developers face a steep learning curve.

---

## 2. Overview of Backstage.io: A Plugin-Powered Platform

* **What is Backstage?**
    * An open platform (CNCF Graduated Project from Spotify) for building developer portals.
    * Aims to unify tools, services, docs, and software components into a single pane of glass.

* **Core Strength: The Plugin Architecture**
    * Backstage is a lightweight core shell; its extensive functionality comes from plugins.
    * **Vast Ecosystem:**
        * Key official plugins: Service Catalog, Scaffolder (for new projects), TechDocs, Kubernetes integration, etc.
        * **Over 100+ community plugins** (and growing!), showcasing massive extensibility (e.g., integrations with GitHub, GitLab, Jenkins, Grafana).
        
    


https://github.com/user-attachments/assets/8c7506b5-4078-4972-90c5-a4f4ba8cef6f

<img width="1710" alt="Screenshot 2025-05-17 at 12 16 38 PM" src="https://github.com/user-attachments/assets/2012f28e-7755-4dcc-9b20-628744cb19d8" />

* **Scale & Complexity Managed:**
        * The main `backstage/backstage` monorepo has millions of lines of code and hundreds of contributors.
        * This scale is manageable *because* of the plugin architecture, which allows for modularity and distributed ownership.

---

## 3. Architectural Choices: Monolith vs. Micro-frontends vs. Plugins

Let's compare how plugin architectures stack up against other common approaches.

* **Monolithic Architecture:**
    * **What:** Entire application is a single, large codebase and deployment unit.
        ```
        +------------------------------------------------------+
        |                   Monolithic App                     |
        | +-----------+ +-----------+ +-----------+            |
        | | Feature A | | Feature B | | Feature C | (All in one) |
        | +-----------+ +-----------+ +-----------+            |
        |------------------------------------------------------|
        | Pros:                                                |
        |   +--------------------------+ +---------------------+ |
        |   | Simpler Initial Dev/Deploy | |Straightforward Test | |
        |   +--------------------------+ +---------------------+ |
        | Cons:                                                |
        |   +------------------+ +----------------+ +------------+ |
        |   | Becomes Unwieldy | | Tight Coupling | | Slow Builds| |
        |   +------------------+ +----------------+ +------------+ |
        |   +------------------+ +-----------------+ +-----------+ |
        |   | Team Bottlenecks | | Tech. Lock-in   | | Isolation | |
        |   +------------------+ +-----------------+ +-----------+ |
        +------------------------------------------------------+
        ```

* **Micro-frontend (MFE) Architecture:**
    * **What:** Application is composed of several smaller, independently deployable frontend applications.
        ```
        +------------------------------------------------------------+
        |                 Micro-frontend App                         |
        | +-------+ +-------+ +-------+ (Separate Deployments)       |
        | | MFE A | | MFE B | | MFE C |                              |
        | +-------+ +-------+ +-------+                              |
        |       \      |      /                                      |
        |     +---------------------+                                |
        |     | App Shell/Container | (Composition Layer)            |
        |     +---------------------+                                |
        |------------------------------------------------------------|
        | Pros:                                                      |
        |   +-------------------+ +---------------------+ +----------+ |
        |   | Max Team Autonomy | | Independent Deploy  | | Tech Diversity |
        |   +-------------------+ +---------------------+ +----------+ |
        | Cons:                                                      |
        |   +-------------------+ +--------------------+ +-----------+ |
        |   | Higher Op. Complex| | UX Inconsistencies | |Bundle Size| |
        |   +-------------------+ +--------------------+ +-----------+ |
        +------------------------------------------------------------+
        ```

* **Plugin-Based Architecture (like Backstage):**
    * **What:** A core "host" application (Backstage app shell) provides a framework, shared services, and extension points. Functionality is added via "plugins" that integrate into the host.
        ```
        +-----------------------------------------------------------------+
        |                   Plugin-Based App (e.g., Backstage)            |
        | +-------------------------------------------------------------+ |
        | |                 Host Application (Core Shell)               | |
        | | (Manages Layout, Core APIs, UI Kit, Plugin Lifecycle)       | |
        | +-------------------------^-----------------------------------+ |
        |                           | (Plugin API / Contract)             |
        |    +-----------------+    |    +-----------------+              |
        |    |    Plugin A     |<---+--->|    Plugin B     | (Integrated) |
        |    +-----------------+         +-----------------+              |
        |-----------------------------------------------------------------|
        | Pros (vs. Monolith):                                            |
        |   +------------+ +-------------+ +---------------------------+ |
        |   | Modularity | | Scalability | | Independent Plugin Dev    | |
        |   +------------+ +-------------+ +---------------------------+ |
        | Pros (vs. MFE):                                                 |
        |   +-----------------+ +---------------------+ +---------------+ |
        |   | Integrated UX   | | Stronger Governance | | Simpler Ops   | |
        |   +-----------------+ +---------------------+ +---------------+ |
        |   +---------------------+                                       |
        |   | Shared Core Logic   |                                       |
        |   +---------------------+                                       |
        +-----------------------------------------------------------------+
        ```
    * **When is a Plugin Architecture a good fit?**
        * Balance between monolith simplicity and MFE flexibility.
        * Building platforms/ecosystems (like Backstage).
        * Central control with distributed feature development.
        * Modular development is key; plugins generally use the host's tech stack (React for Backstage).



---

## 4. Why Plugin-Based Architectures was the right choice for Backstage

Given the scale and goals of Backstage (unifying a diverse set of developer tools and information):

+-----------------------------+
| Easy to Add New Features    |
+-----------------------------+
* Backstage needed to integrate with countless existing tools and allow new ones to be added easily. Plugins provide this extensibility without modifying the core.
    * *Analogy:* Think of Backstage as a smartphone. The core OS is stable (Backstage core), and "apps" (plugins) add specific functionalities (CI/CD views, Kubernetes dashboards, documentation).

+-----------------------------+
| Supports Community Growth   |
+-----------------------------+
* A plugin model encourages community contributions and allows different teams (internal or external) to build and maintain integrations independently. This is key to Backstage's growth with 100+ community plugins.

+---------------------------------------+
| Keeps a Consistent User Experience    |
+---------------------------------------+
* Unlike pure micro-frontends which can sometimes lead to a fragmented UX, Backstage's plugin architecture, coupled with its Core APIs and shared UI toolkit (Material UI & `@backstage/core-components`), ensures a consistent look and feel. Plugins operate within the same host environment.

+---------------------------------------------+
| Balances Team Freedom & Overall Control     |
+---------------------------------------------+
* Teams can own specific plugins (e.g., a team for the Kubernetes plugin, another for TechDocs).
* The Backstage core team provides the framework, APIs, and guidelines, ensuring quality and interoperability.

+---------------------------------------+
| Helps Many Teams Work Together        |
+---------------------------------------+
* As seen, Backstage is a large project. A plugin architecture allows parallel development on different features (plugins) without constant conflicts in a single monolithic codebase.

For Backstage, a monolithic approach wouldn't scale for the number of integrations. A pure micro-frontend approach might sacrifice the desired level of UX consistency and shared core services. The plugin architecture offered the best balance.

---

## 5. Core Concepts of a Plugin System

* **Host Application (Core Shell):**
    * The main React application. In Backstage, this is your `packages/app` directory, which sets up the overall layout, navigation (sidebar), and core services.
    * Manages plugin lifecycle: discovering, loading, and integrating plugins.
    * Provides "extension points": predefined places where plugins can add UI or functionality (e.g., a new page, a sidebar item, a dashboard widget).
    ```
    +----------------------------------------+
    |     BACKSTAGE HOST APPLICATION (app)   |
    |----------------------------------------|
    | Sidebar    |      Main Content Area    |
    | (Plugin    |                           |
    |  Links     |  (Plugin Page Rendered Here)|
    |  Added Here)|                           |
    |            |                           |
    |----------------------------------------|
    | Shared APIs (Identity, Config, Fetch)  |
    | Shared UI (Material UI, Core Components)|
    +----------------------------------------+
    ```

* **Plugin Interface & Contracts (API):**
    * Defines how plugins interact with the Backstage host and other plugins.
    * Backstage uses `createPlugin` to define a plugin and `ApiRef` / `createApiFactory` to define and provide APIs.
    * Specifies what a plugin *must* provide (e.g., its ID, routes, components for extension points) and what it *can* consume from the host (e.g., `IdentityApi`, `ConfigApi`).
    ```
                  +-----------------------------+
                  | Backstage Host Application  |
                  | (Exposes Core APIs like     |
                  |  IdentityApi, FetchApi)     |
                  +-------------^---------------+
                                |
                 (Plugin Contract: createPlugin,
                  Extension Points, ApiRef usage)
                                |
                  +-------------v---------------+
                  |      Backstage Plugin       |
                  | (e.g., @backstage/plugin-catalog)|
                  | (Implements extensions, uses APIs)|
                  +-----------------------------+
    ```
    * Example: A Backstage plugin might export a component for a specific route and use the `IdentityApi` to get the current user.

* **Plugin Discovery & Loading:**
    * **Discovery:** In Backstage, plugins are typically installed as NPM packages and then explicitly imported and registered in the host application's `plugins.ts` file. This tells the host which plugins are available.
    * **Loading:** Plugin code is usually bundled with the host application. `React.lazy` can be used within plugins or by the host for code-splitting parts of a plugin or the plugin itself if it's very large and not always needed. Backstage's build system handles bundling these effectively.
    ```
    Backstage Host App (packages/app/src/plugins.ts):
      import { catalogPlugin } from '@backstage/plugin-catalog';
      // ... other plugin imports

      // Registering plugins tells the host they exist
      // The host then uses their exported extension points.

    Loading a Plugin's Page (Conceptual):
      // Host Router might use React.lazy for plugin pages
      const CatalogPage = React.lazy(() =>
         import('@backstage/plugin-catalog').then(m => ({ default: m.CatalogIndexPage }))
      );
    ```

* **Shared Services & Context (Backstage APIs):**
    * The Backstage host application provides a rich set of core APIs that plugins can use.
    * Examples: `IdentityApi` (user authentication), `ConfigApi` (application configuration), `FetchApi` (making authenticated HTTP requests), `ErrorApi` (error reporting).
    * These are typically accessed within plugins using `useApi(someApiRef)`.
    * This ensures plugins operate consistently and securely within the Backstage environment.
    ```
    +---------------------------------+
    | Backstage Host Application      |
    |---------------------------------|
    | - IdentityApi                   |
    | - ConfigApi                     |-----> (Consumed by Plugins via useApi())
    | - FetchApi                      |
    | - StorageApi                    |
    +------------------^--------------+
                       |
    +------------------|--------------+
    |   Backstage Plugin (e.g., TechDocs) |
    | (Uses FetchApi to get docs,     |
    |  IdentityApi for user context)  |
    +---------------------------------+
    ```

---

## 6. Building Your Own Backstage-Style System in React (Inspired by Backstage.io)

*(This section explains the "how-to" at a high level, using Backstage as our model.)*

Let's look at how Backstage achieves its plugin architecture, and how you could apply similar principles.

* **a) Defining Plugin APIs & Manifests: The "Rules of Engagement" for Plugins**
    * **What is it?** This is about setting clear contracts for how plugins integrate and what services they can use.
        ```
        +--------------------------------+       +----------------------------+
        |      Host Application          |       |         Plugin             |
        | (e.g., Backstage App Shell)    |       | (e.g., Service Catalog)    |
        |--------------------------------|       |----------------------------|
        | - Provides Core APIs (ApiRef)  |<----->| - Defines itself (createPlugin)|
        | - Defines Extension Points     |       | - Consumes Core APIs (useApi)|
        |   (e.g., Sidebar, Routes)      |------>| - Provides Components for  |
        |                                |       |   Extension Points         |
        +--------------------------------+       +----------------------------+
                      ^
                      |
            +---------------------+
            |   Plugin Contract   |
            | (Manifests, createPlugin args, ApiRefs) |
            +---------------------+
        ```
    * **Plugin Definition (like Backstage's `createPlugin`):**
        * Each plugin in Backstage is defined using `createPlugin`. This function takes configuration like:
            * `id`: A unique string for the plugin (e.g., `catalog`, `techdocs`).
            * `apis`: A list of APIs the plugin offers to other parts of the system (if any).
            * `routes`: How the plugin maps URL paths to its components (e.g., `/catalog` maps to the `CatalogIndexPage`).
            * `externalRoutes`: Routes to external services the plugin might link to.
        * This acts like a manifest, telling Backstage core what the plugin is and how it contributes to the overall application.
    * **Core APIs (like Backstage's `ApiRef` and `ApiFactory`):**
        * Backstage provides a system for defining and consuming shared services (APIs). An `ApiRef` is like an interface or a key for an API (e.g., `identityApiRef`).
        * The host application (or other plugins) provides implementations for these APIs using an `ApiFactory`.
        * **Example: `IdentityApi` in Backstage:**
            * `identityApiRef` defines the contract (e.g., `getUserId()`, `getProfileInfo()`).
            * The host app provides an implementation (e.g., using OAuth, SAML).
            * Plugins can then request this API: `const identityApi = useApi(identityApiRef);`
        * This ensures plugins get a consistent way to access core functionalities like user identity, configuration, or making authenticated API calls.
        ```typescript
        // Simplified Backstage-style API definition
        import { createApiRef } from '@backstage/core-plugin-api';

        export const myCustomUtilityApiRef = createApiRef<MyCustomUtility>({
          id: 'my-app.my-utility', // Unique ID for the API
        });

        export interface MyCustomUtility {
          doSomethingCool(): string;
          fetchSomeData(entityRef: string): Promise<any>;
        }

        // In your plugin, you'd use it like:
        // const utilityApi = useApi(myCustomUtilityApiRef);
        // const result = utilityApi.doSomethingCool();
        ```
        * By defining these clearly, you ensure all plugins integrate smoothly and use services in a standardized way, just like in Backstage.

* **b) Dynamic Plugin Loading: Bringing Plugins to Life Efficiently**
    * **What is it?** Backstage aims to be efficient. While plugins are often bundled with the main app, their components (especially pages) are typically lazy-loaded.
        ```
        Initial App Load:
        +-----------------------------------+
        | Host App Shell (Core UI, Router)  | -- Quick to load
        +-----------------------------------+
        | (Plugin code NOT YET loaded)      |
        +-----------------------------------+

        User navigates to /catalog:
        +-----------------------------------+
        | Host App Shell                    |
        +-----------------------------------+
        | React.lazy(() => import(Catalog)) | --- Fetches Catalog Plugin Code ---+
        +-----------------------------------+                                    |
                                                                                 V
                                                                      +-----------------+
                                                                      | Catalog Plugin  |
                                                                      | Code Bundle     |
                                                                      +-----------------+
        ```
    * **`React.lazy` for Page Components:**
        * Backstage commonly uses `React.lazy` for the main components associated with plugin routes.
        * When you navigate to `/catalog`, the code for the Catalog plugin's main page component is loaded on demand.
        * `const CatalogIndexPage = React.lazy(() => import('@backstage/plugin-catalog').then(m => ({ default: m.CatalogIndexPage })));`
        * This means the initial bundle for the Backstage app remains smaller, and users only download the code for the parts of Backstage they actually visit.

* **c) Routing Strategies: Letting Plugins Define Their Own Space**
    * **What is it?** How do plugins like "Service Catalog" or "TechDocs" get their own pages (e.g., `/catalog`, `/docs`) within the Backstage app?
        ```
        +-----------------------------+
        | Host Application Router     |
        | (e.g., using <FlatRoutes>)  |
        |-----------------------------|
        | - Core Route: /             |
        | - Core Route: /settings     |
        |                             |
        |   (Dynamically Adds Routes  |
        |    from Plugins at Startup) |
        |           ^                 |
        |           |                 |
        +-----------|-----------------+
                    |
        +-----------------------+      +-----------------------+
        | Plugin A (Catalog)    |      | Plugin B (TechDocs)   |
        | - Route: /catalog/* |----->| - Route: /docs/* |-----> (to Host Router)
        | - Component: CatalogPage|      | - Component: DocsPage |
        +-----------------------+      +-----------------------+
        ```
    * The **Backstage App Shell** (`packages/app`) sets up the main router using `react-router-dom`.
    * Each plugin, through its `createPlugin` definition, can specify its routes.
    * The Backstage app collects these routes from all registered plugins and integrates them into the main routing configuration.
    * Plugins can also define "mount points" for where their main page component should be rendered.
    * Example (Simplified from how Backstage's `createPlugin` and `FlatRoutes` work):
        ```tsx
        // In Backstage's app setup (conceptually)
        <FlatRoutes> {/* Backstage's wrapper around React Router */}
          <Route path="/" element={<HomepageCompositionRoot />} />
          {/* Routes from catalogPlugin */}
          <Route path="/catalog/*" element={<CatalogIndexPage />} /> {/* Provided by catalog plugin */}
          {/* Routes from techdocsPlugin */}
          <Route path="/docs/*" element={<TechDocsIndexPage />} /> {/* Provided by docs plugin */}
          {/* ... and so on for other plugins */}
        </FlatRoutes>
        ```
        * This allows each plugin to own its URL namespace and contribute its UI to the correct part of the application, appearing as a seamless part of Backstage.

* **d) UI/Component System for Plugins: Ensuring a Cohesive Backstage Experience**
    * **What is it?** Backstage plugins need to look and feel like they belong together.
        ```
        +-----------------------------------+
        | Host Application (Backstage Core) |
        |-----------------------------------|
        | - Material UI (Base)              |
        | - @backstage/core-components      | -- Shared UI Kit
        |   (Header, Page, InfoCard, etc.)  |
        | - Theming System (Colors, Fonts)  |
        +-----------------^-----------------+
                          | (Used By)
        +-----------------|-----------------+
        |    Plugin A     |    Plugin B     |
        | (Uses <Page>,   | (Uses <Header>, |
        |  <InfoCard>)    |  <Table_>)      |
        +-----------------+-----------------+
        Result: Consistent Look & Feel
        ```
    * Backstage provides a **Shared UI Foundation**:
        * It uses **Material UI** as its base component library.
        * It also offers its own library of common components (`@backstage/core-components`) like `Header`, `Page`, `Table`, `InfoCard`. These are built on Material UI and tailored for Backstage's needs.
    * **Theming:** Backstage has a theming system, allowing customization of colors, fonts, etc., which applies globally to the host and all plugins.
    * Plugins are strongly encouraged (and often required) to use these shared components and adhere to the theme. This ensures a consistent user experience across different plugins developed by different teams.
    * This is like Backstage providing a standard set of visual building blocks and a style guide that all plugin developers must use.

* **e) State Management & Data Fetching: How Plugins Handle Information**
    * **What is it?** How do plugins manage their internal data, and how do they get information from the Backstage backend or other sources?
        ```
        +-----------------------------------+
        | Host Application (Backstage Core) |
        |-----------------------------------|
        | - ConfigApi (for app config)      |
        | - IdentityApi (for user info)     | -- Core APIs
        | - FetchApi (for HTTP requests)    |
        | - ...other Core APIs              |
        +-----------------^-----------------+
                          | (Consumed via useApi())
        +-----------------|-----------------+
        | Plugin A (Frontend)             |
        |---------------------------------|
        | - Local State (useState)        |
        | - Uses ConfigApi for its settings |
        | - Uses FetchApi to call its own | -----> +---------------------+
        |   backend or external services  |        | Plugin A (Backend)  |
        +---------------------------------+        | (Optional, separate)|
                                                 +---------------------+
        ```
    * **Local Plugin State:** Plugins typically manage their own UI state using React's built-in hooks (`useState`, `useReducer`).
    * **Accessing Shared Data/Services (via Backstage APIs):**
        * Plugins don't usually share complex state directly with each other. Instead, they rely on Backstage Core APIs.
        * **Configuration:** `useApi(configApiRef)` to get configuration values.
        * **User Identity:** `useApi(identityApiRef)` to get user details.
        * **Data Fetching:** Plugins often define their own backend (`@backstage/plugin-<name>-backend`) and a corresponding client (`@backstage/plugin-<name>-common` or client within the frontend plugin). The frontend plugin uses the `FetchApi` (or a specialized client built upon it) to communicate with its backend or other services.
        ```typescript
        // Inside a Backstage plugin component
        import { useApi, configApiRef } from '@backstage/core-plugin-api';
        import { catalogApiRef } from '@backstage/plugin-catalog-react'; // API from another plugin

        function MyPluginComponent() {
          const config = useApi(configApiRef);
          const catalogApi = useApi(catalogApiRef); // Using an API provided by the catalog plugin
          const backendUrl = config.getString('backend.baseUrl');

          // Fetch data using catalogApi or FetchApi
          // Example: const { entities } = await catalogApi.getEntities(...);

          return <div>My Plugin using {backendUrl}</div>;
        }
        ```
    * This approach keeps plugins decoupled while allowing them to access necessary information and services in a standardized way.

By understanding these principles from Backstage, you can design a React application that is similarly modular, extensible, and scalable.

---

## 7. Conclusion & Key Takeaways

* Scaling React often requires moving beyond monoliths.
* Plugin architectures, as exemplified by Backstage.io, offer a powerful and structured solution.
* Balance modularity with central governance and a cohesive user experience.
* **Key Elements (inspired by Backstage):** Core app shell, well-defined plugin contracts (`createPlugin`), shared Core APIs (`ApiRef`), dynamic loading of plugin UIs, and a common UI toolkit.
* **Choose wisely:** Monoliths vs. MFEs vs. Plugins.

**Start small, define clear contracts (APIs), and iterate!**

---
## 🌍 Open Source Contribution Opportunities

If you're excited about plugin-based architecture and want to contribute to real-world projects, here are some open source opportunities you can explore:

### 🧩 [Backstage](https://github.com/backstage/backstage)
The core platform built by Spotify and now a CNCF project. Everything in Backstage is a plugin—from CI/CD dashboards to Kubernetes monitoring and documentation systems.

### 🌱 [Backstage Community Plugins](https://github.com/backstage/community-plugins)
A collection of open source plugins maintained by the community. These plugins extend Backstage with integrations for GitHub, Jenkins, Datadog, and more.




**Thank You!**
 ### Opensource contribution opportunities 

* Mitesh Kumar [Linkedin](https://www.linkedin.com/in/mitesh-kumar-b0818b143/)
* Subscribe to my newsletter [Engineering the Frontend](https://www.linkedin.com/newsletters/engineering-the-frontend-7283499721103417348/)


---
## 8. Q&A
