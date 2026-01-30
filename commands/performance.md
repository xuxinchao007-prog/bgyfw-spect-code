---
description: Run frontend performance audit using performance-auditor agent (Use PROACTIVELY)
---

# Frontend Performance Audit

Use the **performance-auditor** agent to analyze and optimize frontend performance. Use PROACTIVELY when developing features, before releases, or when performance issues are reported.

The agent will:
1. Analyze Core Web Vitals (LCP, FID, CLS)
2. Review bundle size and code splitting
3. Check for performance anti-patterns
4. Identify runtime bottlenecks
5. Suggest optimization strategies
6. Generate performance metrics report

**When to use:**
- Before releasing new features
- When performance issues are reported
- During code review for frontend changes
- For regular performance audits (weekly/monthly)
- When integrating new libraries

**Key metrics:**
- **Lighthouse Performance score**: Target 90+
- **LCP** (Largest Contentful Paint): < 2.5s
- **FID** (First Input Delay): < 100ms
- **CLS** (Cumulative Layout Shift): < 0.1
- **Bundle size**: < 200KB gzipped
- **Time to Interactive**: < 3s

**Common optimizations:**
- Code splitting and lazy loading
- Image optimization (WebP, AVIF, responsive images)
- Bundle size reduction (tree shaking, dead code elimination)
- Caching strategies (service worker, CDN)
- Rendering performance (reduce main thread blocking)
- Critical resource preloading
