# Architecture & Language Matrix

> **A note on the future:** The long-term vision is a single, unified language capable of handling every layer of the stack — from bare-metal GPU compute to reactive UI. That language does not yet exist. Until it does, the following polyglot architecture is our standard.

## The Core Philosophy

Carbon OS and its core applications (like AI Studio) are built to run seamlessly across macOS, Windows, Linux, and natively on Carbon OS.

To achieve this level of cross-platform stability without sacrificing developer velocity or bare-metal performance, we reject the idea of a "single language" codebase. Instead, we use a strict polyglot architecture. Every service, application, and tool added to the OS must be written in the language specifically designed for its domain.

When designing a new feature, defer to the following language matrix.

---

## 1. C++: The Engine Room
**Domain:** Core AI inference, rendering engines, local LLM execution, and bare-metal hardware access.

When absolute performance is required, or when we need direct access to GPU APIs (CUDA, Metal) and system-level rendering contexts, we use C++. It is the muscle of the OS.

C++ is also the only language in our stack that can natively interface with the macOS Objective-C++ runtime and Windows C++ OS layers, libraries, and services. This deep, native interoperability with the host OS — combined with the fact that C++ compiles directly to native machine code with no runtime or VM in between — makes it the only viable choice for this layer.

* **When to use it:** Hosting WebViews, running local AI models, complex math, or real-time rendering.
* **When NOT to use it:** High-level orchestration, basic file moving, or general application logic. Do not write management layers in C++ just to keep the codebase unified; it introduces unnecessary complexity and brittleness.
* **Dependency Management:** C++ dependencies can become a nest of conflicts. We strictly recommend using **`vcpkg`** to organize, fetch, and build C++ libraries across our multi-platform targets.

## 2. Go (Golang): The Orchestrator
**Domain:** Services, application logic, management tools, network I/O, and file system operations.

Go is our middle-tier manager. It is explicitly designed for concurrency, cross-platform compilation, and system-level orchestration without the memory-management overhead of C++. Because Go compiles to a static binary, it is perfect for utilities that need to drop into any OS and just work.

* **When to use it:** CLI tools (like our `env` package manager), background daemons, filesystem watchers, downloading/unpacking archives, and bridging communication between the UI and the C++ core.
* **When NOT to use it:** Pixel-pushing UI components or heavy, real-time GPU processing.

## 3. TypeScript & TSX: The Presentation Layer
**Domain:** All User Interface (UI) components and reactive visual state.

TypeScript is the undisputed standard for building fluid, reactive, and maintainable user interfaces. By decoupling the UI from the underlying logic, we can iterate rapidly on the user experience without recompiling the core engines.

* **When to use it:** Anything the user clicks, sees, or interacts with. If it involves the DOM, styling, or UI component architecture, it is built in TSX.
* **When NOT to use it:** Heavy data processing, local file system manipulation, or system-level process management.

---

## The Golden Rule of Boundaries

The success of a Carbon OS application relies on the clean separation of these three layers.

1. **TSX** asks for data.
2. **Go** fetches, orchestrates, or manages that data.
3. **C++** performs the heavy compute when Go tells it to.