# QR Forge - Dependency Audit Report

**Date:** 2026-01-14
**Project:** QR Forge (qr-forge2)
**Type:** Single-file HTML application

---

## Executive Summary

This audit analyzed the dependencies used in QR Forge, a client-side QR code generator. The application is a single-file HTML application with minimal external dependencies loaded via CDN. While the architecture is simple and lightweight, there are **critical security concerns** and **outdated packages** that require immediate attention.

### Key Findings:
- ✅ **No npm vulnerabilities** (no package.json exists)
- ⚠️ **Outdated library version** (qr-code-styling 1.5.0 → 1.9.2)
- 🔴 **Critical security issue**: Missing Subresource Integrity (SRI) on CDN resources
- 🔴 **Security risk**: Unpinned Google Fonts CDN
- ✅ **Minimal bloat**: Lean architecture with only essential dependencies

---

## Dependency Analysis

### 1. External Dependencies

The application uses the following external dependencies via CDN:

#### A. **qr-code-styling** (JavaScript Library)
- **Current Version:** 1.5.0
- **Latest Version:** 1.9.2 (as of 2026-01)
- **Source:** unpkg.com
- **Status:** ⚠️ **OUTDATED** (7 minor versions behind)
- **Security Status:** ✅ No known CVEs
- **Loaded from:** `https://unpkg.com/qr-code-styling@1.5.0/lib/qr-code-styling.js`

**Issues:**
- Version is approximately 2+ years old
- Missing Subresource Integrity (SRI) hash
- Potential bug fixes and features in newer versions

#### B. **Google Fonts** (DM Sans, Bebas Neue)
- **Source:** fonts.googleapis.com / fonts.gstatic.com
- **Status:** ⚠️ **NO VERSION PINNING**
- **Security Status:** ⚠️ Missing SRI (not recommended for fonts, but still a consideration)

**Fonts loaded:**
- DM Sans (weights: 300, 400, 500)
- Bebas Neue

**Issues:**
- Google can change font files at any time
- Preconnect hints are good for performance
- No privacy concerns due to crossorigin attribute

---

## Security Vulnerabilities

### 🔴 CRITICAL: Missing Subresource Integrity (SRI)

**Severity:** HIGH
**Location:** Line 1006 (qr-code-styling CDN)

**Issue:**
The application loads JavaScript from unpkg CDN without SRI hashes. This exposes users to supply chain attacks if:
- The unpkg CDN is compromised
- DNS hijacking occurs
- Man-in-the-middle attacks intercept the request

**Impact:**
An attacker who compromises the CDN could inject malicious code into all QR Forge users' browsers, potentially:
- Stealing user input data (URLs, WiFi passwords, email addresses)
- Redirecting QR code generation to malicious servers
- Injecting crypto miners or other malware

**References:**
- [Public CDNs Are Useless and Dangerous](https://httptoolkit.com/blog/public-cdn-risks/)
- [Managing Third-Party Assets Security Risks](https://auth0.com/blog/third-party-assets-security-risks-management/)
- [CDN Security Best Practices](https://www.sitelock.com/blog/cdn-security-best-practices/)

### 🟡 MEDIUM: Outdated Library Version

**Severity:** MEDIUM
**Library:** qr-code-styling 1.5.0 → 1.9.2

**Issue:**
Using an outdated version (1.5.0) instead of the latest stable release (1.9.2) published 9 months ago.

**Potential Risks:**
- Missing security patches (if any were released)
- Missing bug fixes that could affect QR code generation
- Missing performance improvements
- Potential compatibility issues with modern browsers

**Note:** No specific CVEs have been found for qr-code-styling, but staying current is best practice.

**References:**
- [qr-code-styling on npm](https://www.npmjs.com/package/qr-code-styling)
- [Security Analysis - Socket.dev](https://socket.dev/npm/package/@p107/qr-code-styling)

### 🟡 LOW-MEDIUM: CDN Dependency Risk

**Severity:** LOW-MEDIUM
**Issue:** Relying on third-party CDNs (unpkg, Google Fonts)

**Concerns:**
- Single point of failure if CDN goes down
- Privacy implications (Google Fonts can track users)
- No control over CDN availability or performance
- Potential for future compromises

**Best Practice Recommendation:**
Self-host dependencies for maximum security and reliability.

---

## Code Bloat Analysis

### ✅ Excellent: Minimal Bloat

The application demonstrates **excellent architecture** with minimal unnecessary code:

**Strengths:**
1. **Single-file application** - No build process complexity
2. **No framework overhead** - Pure vanilla JavaScript
3. **Inline CSS** - No external stylesheets to load
4. **Single external dependency** - Only qr-code-styling library
5. **Client-side only** - No server dependencies
6. **Efficient DOM manipulation** - Direct manipulation without abstractions

**Code Quality:**
- Clean, readable JavaScript
- Well-organized CSS with CSS custom properties
- Responsive design without heavy framework
- Minimal JavaScript payload (~1.5KB minified, excluding library)

**Library Size:**
- qr-code-styling: ~50KB minified (reasonable for functionality provided)
- Google Fonts: ~20-30KB per font (could be optimized)

### Potential Optimizations

#### 1. Font Loading Optimization (LOW PRIORITY)
**Current:** Loading 3 font weights of DM Sans
**Recommendation:** Consider if all weights (300, 400, 500) are necessary
**Savings:** ~10-15KB per unused weight

#### 2. Font Format Optimization (LOW PRIORITY)
**Current:** Loading full font files
**Recommendation:** Use `font-display: swap` (already done via `&display=swap`)
**Additional:** Could subset fonts to only include used characters

#### 3. CSS Custom Property Usage (ALREADY OPTIMIZED) ✅
The code excellently uses CSS custom properties for theming, avoiding repetition.

#### 4. Animation Complexity (VERY LOW PRIORITY)
**Current:** Multiple CSS animations (grid movement, floating orbs, pulse effects)
**Impact:** Minimal (~2KB of CSS)
**Recommendation:** Keep as-is - adds visual polish without significant bloat

---

## Recommendations

### 🔴 CRITICAL PRIORITY

#### 1. Add Subresource Integrity (SRI) to qr-code-styling

**Action:** Replace line 1006 with:
```html
<script
    src="https://unpkg.com/qr-code-styling@1.5.0/lib/qr-code-styling.js"
    integrity="sha384-[HASH_HERE]"
    crossorigin="anonymous">
</script>
```

**How to get SRI hash:**
```bash
curl https://unpkg.com/qr-code-styling@1.5.0/lib/qr-code-styling.js | \
openssl dgst -sha384 -binary | openssl base64 -A
```

Or use online tool: https://www.srihash.org/

#### 2. Upgrade qr-code-styling to Latest Version

**Current:** 1.5.0
**Target:** 1.9.2
**Action:**
```html
<script src="https://unpkg.com/qr-code-styling@1.9.2/lib/qr-code-styling.js"></script>
```

**Important:** Must regenerate SRI hash after version change

**Testing Required:**
- Verify all QR generation features still work
- Test logo upload functionality
- Test all QR styles (dots, markers, corners)
- Test all export formats (PNG, SVG)
- Cross-browser testing (Chrome, Firefox, Safari, Edge)

### 🟡 HIGH PRIORITY

#### 3. Consider Self-Hosting Dependencies

**Rationale:** Eliminates CDN security risks and improves reliability

**Implementation:**
1. Download qr-code-styling library locally
2. Download and self-host Google Fonts
3. Update references in HTML
4. Use your own CDN or serve from same origin

**Benefits:**
- Full control over dependencies
- No third-party privacy concerns
- Better performance (no external DNS lookups)
- Offline functionality
- No CDN availability dependencies

**Trade-offs:**
- Loses CDN caching benefits across sites (minimal in practice)
- Must manage updates manually
- Slightly larger initial page size

#### 4. Add Content Security Policy (CSP)

**Action:** Add meta tag to prevent XSS attacks
```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self' 'unsafe-inline' https://unpkg.com;
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
               font-src 'self' https://fonts.gstatic.com;
               img-src 'self' data: blob:;">
```

Adjust based on whether you self-host or continue using CDNs.

### 🟢 MEDIUM PRIORITY

#### 5. Create package.json for Dependency Management

**Benefits:**
- Track dependencies formally
- Enable security scanning (npm audit, Snyk)
- Facilitate future development workflow
- Document project dependencies

**Example package.json:**
```json
{
  "name": "qr-forge",
  "version": "1.0.0",
  "description": "Free QR Code Generator with Logo",
  "private": true,
  "dependencies": {
    "qr-code-styling": "^1.9.2"
  },
  "scripts": {
    "audit": "npm audit",
    "serve": "python3 -m http.server 8000"
  }
}
```

#### 6. Implement Automated Security Scanning

**Tools to consider:**
- Snyk (free for open source)
- Dependabot (GitHub native)
- npm audit (if package.json is added)

### 🟢 LOW PRIORITY (NICE TO HAVE)

#### 7. Font Subsetting
Reduce font file sizes by including only used characters.

#### 8. Add Fallback for CDN Failure
Implement fallback to local copy if CDN fails to load.

#### 9. Consider Progressive Web App (PWA)
Add service worker for offline functionality.

---

## Alternative Libraries Considered

### Actively Maintained Forks:

1. **@liquid-js/qr-code-styling**
   - Version: 4.1.0
   - Last updated: 3 months ago
   - More actively maintained than original
   - **Recommendation:** Consider migration for better long-term support

2. **linkbreakers-qr-code-styling**
   - Version: 1.1.2
   - Last updated: 23 days ago (very recent)
   - Active maintenance
   - **Recommendation:** Investigate for potential migration

### Why Consider Alternatives?

The original `qr-code-styling` (kozakdenys) was last published 9 months ago, while forks show more recent activity. More active maintenance means:
- Faster security patches
- Better browser compatibility
- Bug fixes and improvements
- Community support

---

## Testing Checklist for Updates

Before deploying any dependency updates:

- [ ] QR code generation works for all types (URL, Text, Email, WiFi)
- [ ] Logo upload and display works correctly
- [ ] All style options function properly (dots, markers, corners)
- [ ] Color pickers update QR code correctly
- [ ] Size slider adjusts QR code size
- [ ] PNG download produces high-resolution output
- [ ] SVG download produces valid SVG file
- [ ] Test on Chrome, Firefox, Safari, Edge
- [ ] Test on mobile devices (iOS Safari, Chrome Android)
- [ ] Verify no console errors
- [ ] Check performance (should remain fast)

---

## Conclusion

QR Forge demonstrates excellent architecture with minimal bloat and unnecessary dependencies. However, **critical security improvements** are needed:

### Immediate Actions Required:
1. ✅ Add SRI hashes to all external scripts
2. ✅ Update qr-code-styling to version 1.9.2
3. ✅ Add Content Security Policy

### Medium-Term Improvements:
1. Consider self-hosting dependencies
2. Evaluate migrating to more actively maintained fork
3. Implement automated security scanning

### Long-Term Considerations:
1. Convert to PWA for offline functionality
2. Add build process for optimization (optional)
3. Implement automated dependency updates

The application is well-architected and lean. With the recommended security improvements, it will be both secure and maintainable.

---

## References & Sources

### Security Research:
- [Public CDNs Are Useless and Dangerous](https://httptoolkit.com/blog/public-cdn-risks/)
- [Managing Third-Party Assets Security Risks](https://auth0.com/blog/third-party-assets-security-risks-management/)
- [CDN Security Best Practices - SiteLock](https://www.sitelock.com/blog/cdn-security-best-practices/)
- [Compromising Websites Through a CDN](https://justi.cz/security/2018/05/23/cdn-tar-oops.html)
- [CDN Flaw Left Thousands of Sites Open to Abuse](https://portswigger.net/daily-swig/cdn-flaw-left-thousands-of-websites-open-to-abuse)

### Package Information:
- [qr-code-styling on npm](https://www.npmjs.com/package/qr-code-styling)
- [qr-code-styling Security Analysis - Socket.dev](https://socket.dev/npm/package/@p107/qr-code-styling)
- [qr-code-styling Security Analysis - Snyk](https://snyk.io/advisor/npm-package/qr-code-styling)

### CDN Comparison:
- [jsDelivr vs unpkg vs cdnjs](https://blog.blazingcdn.com/en-us/jsdelivr-vs-unpkg-vs-cdnjs-best-free-cdn-for-open-source-projects)
- [Choose the Best JS CDN](https://blog.blazingcdn.com/en-us/choose-best-js-cdn-compare-npm-alternatives-skypack-jsdelivr-unpkg)

---

**Audit Performed By:** Claude (AI Assistant)
**Date:** January 14, 2026
**Next Review:** Recommended in 6 months or after any dependency changes
