# PIE Component Library - Improvement Recommendations

## Executive Summary

Based on a comprehensive analysis of the PIE (Principles for Interfaces and Experiences) component library monorepo, this document outlines specific, actionable improvements across architecture, testing, documentation, developer experience, and code quality.

The PIE library is a mature Web Component-based design system using Lit, with React wrappers, comprehensive testing via Playwright, and good monorepo tooling. However, there are several opportunities for enhancement.

---

## 1. Testing & Quality Assurance

### 1.1 Expand Unit Test Coverage
**Current State:**
- Only 1 unit test file found (`pie-cookie-banner/test/unit/localisation-utils.test.ts`)
- Heavy reliance on Playwright component and visual tests
- No apparent unit testing for utility functions, mixins, or core logic

**Recommendations:**
- **Add unit tests for shared utilities and mixins**
  - Test `pie-webc-core` mixins (FormControlMixin, DelegatesFocusMixin) in isolation
  - Test decorator functions (`@validPropertyValues`, `@requiredProperty`, etc.)
  - Test event dispatching utilities
  
- **Component-level unit testing**
  - Add Vitest unit tests for component logic that doesn't require browser APIs
  - Test state management, computed properties, and business logic
  - Example: Test button variant logic, modal state transitions, form validation logic
  
- **Create testing guidelines**
  - Document when to use unit tests vs. Playwright tests
  - Provide examples of good test patterns
  - Target: Aim for 70%+ coverage for utility functions and mixins

**Priority:** High
**Estimated Effort:** Medium
**Impact:** Reduced bugs, faster test execution, better code confidence

---

## 2. Code Quality & Technical Debt

### 2.1 Address TODO Comments
**Current State:**
- 19 TODO/FIXME comments found across components
- Recurring TODO: "Verify if ReactBaseType can be set as a more specific React interface" (appears in 8 components)
- Button component TODO: "if we need to repeat this logic elsewhere, then we should consider moving this code to a shared class or mixin"
- Radio-group contains comment: "This is quite hacky"

**Recommendations:**
- **Resolve React TypeScript TODOs**
  ```typescript
  // Current state in multiple files (defs-react.ts):
  /**
   * TODO: Verify if ReactBaseType can be set as a more specific React interface
   */
  type ReactBaseType = React.HTMLAttributes<HTMLElement>;
  ```
  - Create a centralized type mapping system for React wrappers
  - Document the React type strategy in pie-wrapper-react
  - Consider creating a utility type that maps Web Component tag names to appropriate React base types

- **Extract Form Simulation Mixin**
  - The `_simulateNativeButtonClick` logic in `pie-button` should be extracted into a reusable mixin
  - Other form-associated components might benefit from this pattern
  - Location: `@justeattakeaway/pie-webc-core/src/mixins/FormSimulationMixin.ts`

- **Refactor "Hacky" Code**
  - Review and improve the radio-group change event emission logic
  - Document why certain approaches are necessary (Shadow DOM limitations, etc.)

**Priority:** Medium
**Estimated Effort:** Small-Medium
**Impact:** Improved code maintainability, reduced duplication

### 2.2 Performance Optimization Opportunities
**Current State:**
- Cookie-banner has a performance TODO for caching in localisation utils

**Recommendations:**
- **Implement caching for localisation utilities**
  ```typescript
  // packages/components/pie-cookie-banner/src/localisation-utils.ts:146
  // TODO: This function performance can benefit from some form of caching
  ```
  - Add memoization for locale lookups
  - Consider using WeakMap for component instance-specific caching

- **Bundle Analysis Review**
  - The library generates `stats.html` for each component - create a dashboard or CI check to monitor bundle size trends
  - Set bundle size budgets using bundlewatch (already configured but could be enhanced)

**Priority:** Low
**Estimated Effort:** Small
**Impact:** Better runtime performance, prevention of bundle bloat

---

## 3. Developer Experience (DX)

### 3.1 Improve Component Generator
**Current State:**
- Good Yeoman-based generator exists
- Manual steps required for sub-components
- Percy setup requires manual configuration

**Recommendations:**
- **Enhance sub-component generation**
  - Add CLI flag to generate sub-components: `npx yo @justeattakeaway/pie-component --sub-component`
  - Auto-update vite.config.ts when sub-components are added
  - Auto-run `npx add-components` after generation

- **Automate Percy token setup**
  - Provide a CLI command: `npx setup-percy-token <component-name> <token>`
  - Create a guided wizard during component generation
  - Add validation to check if Percy token is set in CI

- **Generate test scaffolding**
  - Include example unit tests in the component template
  - Add common test scenarios (accessibility, component rendering, event handling)
  - Include page object pattern examples for Playwright tests

**Priority:** Medium
**Estimated Effort:** Medium
**Impact:** Faster component creation, reduced onboarding time

### 3.2 Enhanced Development Workflow
**Recommendations:**
- **Hot Module Replacement (HMR) improvements**
  - Ensure Storybook HMR works seamlessly with component watch mode
  - Document best practices for running multiple watch tasks

- **Better error messages**
  - Enhance `@validPropertyValues` decorator to provide clearer error messages
  - Add development-mode warnings for common mistakes (e.g., missing required props)

- **Component playground**
  - Create a lightweight dev environment separate from Storybook for rapid prototyping
  - Consider using Vite's built-in dev server with a simple HTML page

**Priority:** Low-Medium
**Estimated Effort:** Small-Medium
**Impact:** Faster development cycles, fewer errors

---

## 4. Documentation Improvements

### 4.1 Architecture & Patterns Documentation
**Current State:**
- Good README files for most packages
- Component-level documentation exists
- Missing architecture decision records (ADRs)

**Recommendations:**
- **Create Architecture Documentation**
  - Document the Shadow DOM event handling strategy (why custom events are needed)
  - Explain the form control association approach and its limitations
  - Document the React wrapper generation pipeline and type system
  - Create a "Component Patterns" guide covering:
    - When to use mixins vs. composition
    - State management patterns
    - Event handling best practices
    - Accessibility patterns

- **Add Decision Records**
  - Create `/docs/decisions/` directory with ADRs
  - Document key decisions like:
    - Why Lit over other Web Component frameworks
    - Why Playwright CT over other testing approaches
    - React wrapper strategy
    - Monorepo structure decisions

**Priority:** Medium
**Estimated Effort:** Medium
**Impact:** Better onboarding, clearer contribution guidelines

### 4.2 API Documentation
**Recommendations:**
- **Enhance Component API Documentation**
  - Generate API documentation from `custom-elements.json`
  - Create a dedicated API reference site (could use 11ty or similar)
  - Include usage examples for every prop, event, and slot

- **Migration Guides**
  - Document migration paths from "snacks" (referenced in button component metadata)
  - Create version upgrade guides for breaking changes
  - Provide codemods where possible

**Priority:** Medium
**Estimated Effort:** Medium-High
**Impact:** Reduced support requests, easier adoption

---

## 5. Accessibility (A11y)

### 5.1 A11y Testing Enhancement
**Current State:**
- Accessibility tests exist using `@axe-core/playwright`
- Good ARIA support in components (modal, buttons, etc.)

**Recommendations:**
- **Expand A11y Testing Coverage**
  - Ensure every component has accessibility tests
  - Add keyboard navigation tests for all interactive components
  - Test screen reader announcements using virtual screen readers

- **A11y Documentation**
  - Create an accessibility guide for the design system
  - Document keyboard shortcuts for complex components
  - Provide accessibility examples in Storybook

- **Automated A11y Checks in CI**
  - Add Pa11y or similar to CI pipeline
  - Fail builds on accessibility violations
  - Generate accessibility reports

**Priority:** High
**Estimated Effort:** Medium
**Impact:** Better inclusive design, legal compliance

---

## 6. Type Safety & Developer Ergonomics

### 6.1 TypeScript Improvements
**Recommendations:**
- **Stricter Type Checking**
  - Enable `strict: true` in root tsconfig if not already enabled
  - Use `exactOptionalPropertyTypes` for better prop safety
  - Enforce consistent use of `readonly` for immutable props

- **Improved React TypeScript Support**
  - Provide better type inference for React wrapper events
  - Create a utility type for Web Component refs in React
  - Document React + TypeScript usage patterns

- **Generate TypeScript Types for Themes/Tokens**
  - Auto-generate types from design tokens
  - Provide type-safe theme customization

**Priority:** Medium
**Estimated Effort:** Small-Medium
**Impact:** Fewer runtime errors, better DX

---

## 7. Build & Deployment

### 7.1 Build Performance
**Recommendations:**
- **Optimize Turbo Cache**
  - Review and optimize Turborepo cache configurations
  - Ensure remote caching is properly configured
  - Profile slow build tasks

- **Parallel Builds**
  - Audit dependencies between packages to maximize parallelization
  - Consider splitting large packages if they're bottlenecks

- **Incremental Type Checking**
  - Use TypeScript project references for faster type checking
  - Implement incremental type builds

**Priority:** Low-Medium
**Estimated Effort:** Small
**Impact:** Faster CI builds, better developer productivity

### 7.2 CDN Publishing
**Current State:**
- CDN publishing supported via `pieMetadata.cdnPublish`
- Good metadata configuration system

**Recommendations:**
- **CDN Bundle Optimization**
  - Create optimized CDN bundles with all dependencies inlined
  - Provide both ES module and UMD versions
  - Generate source maps for debugging

- **CDN Documentation**
  - Provide copy-paste CDN usage examples for each component
  - Document versioning strategy for CDN links
  - Create a CDN migration guide

**Priority:** Low
**Estimated Effort:** Small-Medium
**Impact:** Easier adoption without build tools

---

## 8. Monorepo Health

### 8.1 Dependency Management
**Recommendations:**
- **Dependency Audit**
  - Regular audits for outdated packages
  - Remove unused dependencies
  - Consolidate versions across workspace (already using resolutions, which is good)

- **Upgrade Strategy**
  - Document upgrade policy (when to upgrade major versions)
  - Create automated PRs for minor/patch updates
  - Test suite must pass before merging dependency updates

**Priority:** Medium
**Estimated Effort:** Ongoing
**Impact:** Security, performance, maintainability

### 8.2 Code Sharing & Reusability
**Recommendations:**
- **Extract More Shared Utilities**
  - Review components for duplicated logic
  - Create shared utilities in `pie-webc-core`
  - Examples: focus management, scroll locking, resize observers

- **Shared Test Utilities**
  - Create a comprehensive test utility package
  - Provide common test helpers (e.g., render with theme, mock user interactions)
  - Share page object patterns

**Priority:** Medium
**Estimated Effort:** Medium
**Impact:** Reduced duplication, easier testing

---

## 9. Component Completeness

### 9.1 Component Status Review
**Current State:**
- Good status tracking system (alpha, beta, stable)
- Some components are stable, others in various stages

**Recommendations:**
- **Component Audit**
  - Review all alpha/beta components for promotion opportunities
  - Identify missing functionality preventing stable release
  - Create roadmap for stabilization

- **Component Coverage Analysis**
  - Compare against common design system components
  - Identify gaps (e.g., Accordion, Dropdown menu, Tooltip if missing)
  - Prioritize based on consumer needs

**Priority:** Low-Medium
**Estimated Effort:** Ongoing
**Impact:** More complete design system

---

## 10. Community & Contribution

### 10.1 Contribution Experience
**Recommendations:**
- **Enhance Contribution Guide**
  - Add "Good First Issue" labels and documentation
  - Create video walkthroughs of common contribution tasks
  - Provide contributor recognition (contributors page, changelog credits)

- **Issue Templates**
  - Create templates for bug reports, feature requests, and component proposals
  - Include reproduction template with CodeSandbox/StackBlitz

- **PR Review Guidelines**
  - Document what reviewers look for
  - Provide PR checklist template
  - Set up automatic PR labeling

**Priority:** Low
**Estimated Effort:** Small
**Impact:** More external contributions, better community engagement

---

## Implementation Roadmap

### Phase 1 (Quick Wins - 1-2 sprints)
1. Resolve React TypeScript TODOs
2. Add more unit tests for core utilities
3. Extract form simulation mixin
4. Document architecture patterns

### Phase 2 (Core Improvements - 2-3 sprints)
1. Expand A11y testing coverage
2. Enhance component generator
3. Create API documentation system
4. Implement build performance optimizations

### Phase 3 (Polish & Scale - 3-4 sprints)
1. Component status audit and stabilization
2. Create migration guides
3. Enhance CDN publishing
4. Improve contribution experience

---

## Success Metrics

To measure the impact of these improvements:

1. **Code Quality**
   - Unit test coverage: Target 70%+
   - TODO count: Reduce by 80%
   - Build time: Reduce by 20%

2. **Developer Experience**
   - Time to create new component: Reduce by 30%
   - Onboarding time: Track time-to-first-PR for new contributors
   - Developer satisfaction surveys

3. **Library Health**
   - Number of stable components: Increase by X
   - Bundle sizes: Maintain or reduce
   - Accessibility violations: Zero in CI

4. **Adoption**
   - Documentation page views
   - NPM downloads
   - External contributions

---

## Conclusion

The PIE component library is well-architected with solid foundations. These recommendations focus on:
- **Reducing technical debt** (TODOs, test coverage)
- **Improving developer experience** (better tooling, documentation)
- **Ensuring quality** (testing, accessibility)
- **Scaling for the future** (performance, contribution process)

Implementing these improvements will make PIE more maintainable, easier to contribute to, and more valuable for consumers.
