---
name: motion-craft
description: >
  Professional web animation framework for 2026. Master GSAP's free plugins (ScrollTrigger,
  SplitText, Flip, DrawSVG, MorphSVG, MotionPath, Observer, ScrollSmoother), React/Next.js
  integration patterns, CSS animations with animation-timeline, Framer Motion alternatives,
  scroll-driven animations, text effects, page transitions, performance optimization, and
  production anti-patterns. Build enterprise-grade animations with confidence.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "GSAP 3.12+, React 18+, Next.js 14+, MDN, CanIUse"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## Motion Craft: Web Animation Mastery

Animation separates good web experiences from exceptional ones. This skill covers production-grade animation techniques for 2026, from GSAP fundamentals to advanced scroll-driven sequences, React integration patterns, and performance optimization.

---

## Part 1: GSAP Fundamentals & Free Plugins

### Why GSAP in 2026?

GSAP 3.12+ made all plugins free. ScrollTrigger, SplitText, Flip, DrawSVG, MorphSVG, MotionPath, Observer, and ScrollSmoother are all included. GSAP offers:

- Hardware-accelerated transforms and opacity
- Timeline composition and stagger patterns
- Precise timing and bezier easing
- Cross-browser reliability
- Superior performance vs. CSS for complex sequences

### Installation & Registration

```bash
npm install gsap
```

Register plugins once at app startup:

```javascript
import gsap from 'gsap';
import { ScrollTrigger, SplitText, Flip, DrawSVG, MorphSVG, MotionPath, Observer, ScrollSmoother } from 'gsap/all';

// Register all plugins globally
gsap.registerPlugin(ScrollTrigger, SplitText, Flip, DrawSVG, MorphSVG, MotionPath, Observer, ScrollSmoother);
```

### Basic Timeline Pattern

GSAP timelines are the foundation. They orchestrate sequences with precise timing:

```javascript
const tl = gsap.timeline();

tl.to('.hero', { duration: 0.8, opacity: 1, y: 0 }, 0)
  .to('.subtitle', { duration: 0.6, opacity: 1, y: 0 }, 0.2)
  .to('.cta-button', { duration: 0.5, opacity: 1, scale: 1 }, 0.4);
```

Key timeline concepts:
- `duration`: Animation length in seconds
- Position parameter (last arg): When in timeline (0 = start, 0.2 = 0.2s after start)
- Chain methods: `.to()`, `.from()`, `.fromTo()`, `.set()`
- `.play()`, `.pause()`, `.reverse()`, `.progress()`

### Stagger Patterns

Stagger staggers animations across multiple elements:

```javascript
// Stagger by 0.1s per element
gsap.to('.card', {
  duration: 0.6,
  opacity: 1,
  y: 0,
  stagger: 0.1, // Default: stagger by index
});

// Advanced stagger object
gsap.to('.item', {
  duration: 0.8,
  opacity: 1,
  y: 0,
  stagger: {
    amount: 0.6,        // Total stagger time across all elements
    from: 'center',     // 'start', 'center', 'edges', 'random'
    grid: [3, 4],       // For grid layouts: [cols, rows]
    ease: 'power2.inOut'
  },
});

// Reverse stagger
gsap.to('.list-item', {
  duration: 0.5,
  opacity: 1,
  stagger: {
    amount: 0.3,
    from: 'end'
  }
});
```

### ScrollTrigger: Scroll-Linked Animations

ScrollTrigger triggers animations when elements enter the viewport:

```javascript
gsap.to('.section-content', {
  scrollTrigger: {
    trigger: '.section-content',
    start: 'top 80%',          // When trigger top hits 80% from viewport top
    end: 'bottom 20%',         // When trigger bottom hits 20% from viewport top
    markers: true,             // Dev: visual markers on timeline
    scrub: 1,                  // Smooth scrub: animation linked to scrollbar (1 = 1s smoothing)
    pin: true,                 // Pin element while scrolling through
    pinSpacing: true,          // Add spacing for pinned content
    onEnter: () => console.log('entered'),
    onLeave: () => console.log('left'),
    onEnterBack: () => console.log('entered back'),
    onLeaveBack: () => console.log('left back'),
  },
  duration: 1,
  opacity: 1,
  y: 0,
});

// Batch trigger multiple elements efficiently
ScrollTrigger.batch('.card', {
  onEnter: (batch) => gsap.to(batch, { opacity: 1, y: 0, stagger: 0.1, overwrite: 'auto' }),
  onLeave: (batch) => gsap.set(batch, { opacity: 0, y: 100, overwrite: 'auto' }),
  onEnterBack: (batch) => gsap.to(batch, { opacity: 1, y: 0, stagger: 0.1, overwrite: 'auto' }),
  interval: 0.2,
});
```

Key ScrollTrigger properties:
- `trigger`: Element that triggers animation
- `start` / `end`: Viewport positions (e.g., 'top center', 'bottom 100px')
- `scrub`: Link to scrollbar (true/false or smoothing duration)
- `pin`: Fix element during animation
- `snap`: Snap to scroll points
- `markers`: Dev-only visual indicators

### SplitText: Character, Word, Line Animations

SplitText splits text into animatable units:

```javascript
// Split text into characters, words, and lines
const split = new SplitText('.hero-title', {
  type: 'chars,words,lines',
  linesClass: 'split-line'
});

// Animate characters
gsap.from(split.chars, {
  duration: 0.6,
  opacity: 0,
  y: 20,
  stagger: 0.05,
  ease: 'back.out'
});

// Word-by-word reveal
const wordSplit = new SplitText('.paragraph', { type: 'words' });
gsap.from(wordSplit.words, {
  duration: 0.3,
  opacity: 0,
  x: -10,
  stagger: 0.1
});

// Cleanup: SplitText adds DOM elements; revert when done
split.revert();
```

### DrawSVG: Animated SVG Strokes

DrawSVG animates SVG stroke drawing:

```html
<svg width="200" height="200" viewBox="0 0 200 200">
  <circle cx="100" cy="100" r="90" fill="none" stroke="black" stroke-width="2" />
</svg>
```

```javascript
// Draw stroke from start to end
gsap.from('circle', {
  duration: 1.5,
  drawSVG: '0%',  // Start with 0% drawn
  ease: 'power2.inOut'
});

// Animate to specific percentage
gsap.to('path', {
  duration: 2,
  drawSVG: '50% 100%',  // Draw from 50% to 100%
});

// Reverse animation
gsap.to('svg path', {
  scrollTrigger: {
    trigger: '.svg-container',
    scrub: 2,
  },
  drawSVG: '100%',
  duration: 1
});
```

### MorphSVG: Smooth Shape Transitions

MorphSVG smoothly morphs between SVG shapes:

```html
<svg>
  <path id="shape1" d="M100,50 Q200,100 100,150 Q0,100 100,50" />
  <path id="shape2" d="M100,100 Q200,50 200,150 Q100,200 0,150 Q0,100 100,100" />
</svg>
```

```javascript
// Morph from shape1 to shape2
gsap.to('#shape1', {
  duration: 1,
  attr: { d: '#shape2' },  // Use MorphSVG to morph
  ease: 'sine.inOut'
});

// Timeline morphing
const tl = gsap.timeline();
tl.to('#shape1', { duration: 0.6, attr: { d: '#shape2' } })
  .to('#shape1', { duration: 0.6, attr: { d: '#shape3' } })
  .to('#shape1', { duration: 0.6, attr: { d: '#shape1' } });
```

### MotionPath: Animate Along SVG Paths

MotionPath moves elements along custom paths:

```html
<svg>
  <path id="path" d="M100,100 Q200,50 300,100 T500,100" fill="none" />
</svg>
<div class="moving-element"></div>
```

```javascript
// Animate element along path
gsap.to('.moving-element', {
  duration: 3,
  motionPath: {
    path: '#path',
    align: '#path',
    autoRotate: true,  // Rotate to face direction
    alignOrigin: [0.5, 0.5]  // Center point
  },
  ease: 'power1.inOut'
});

// Array of points motion path
gsap.to('.ball', {
  duration: 4,
  motionPath: {
    path: [
      { x: 0, y: 0 },
      { x: 100, y: 50 },
      { x: 200, y: 0 },
      { x: 300, y: 100 }
    ],
    curviness: 1.5,
    autoRotate: true
  }
});
```

### Flip: Flip Between Two States

Flip animates elements between two DOM states:

```javascript
// Save initial state
Flip.getState('.card');

// Change DOM: reorder, resize, etc.
myList.appendChild(myList.querySelector('.card'));
document.querySelector('.card').style.width = '300px';

// Animate to new state
Flip.from('.card', {
  duration: 0.6,
  absolute: true,  // Use position: absolute for animation
  ease: 'back.out',
  stagger: 0.1
});
```

### Observer: Scroll & Pointer Events

Observer listens to scroll and pointer events with optimized performance:

```javascript
// Scroll-speed detection
let proxy = { skew: 0 },
    skewSetter = gsap.quickSetter('.element', 'skewY', 'deg'),
    clamp = gsap.utils.clamp(-20, 20);

Observer.create({
  type: 'scroll,touch',
  onChangeY: (self) => {
    let skew = clamp(self.getVelocity() / -300);
    if (Math.abs(skew) > Math.abs(proxy.skew)) {
      proxy.skew = skew;
      gsap.to(proxy, {
        skew: 0,
        duration: 0.8,
        ease: 'power3',
        overwrite: true,
        onUpdate: () => skewSetter(proxy.skew)
      });
    }
  }
});

// Pointer events
Observer.create({
  type: 'pointer',
  onMove: (self) => {
    console.log('X:', self.x, 'Y:', self.y);
  },
  onClick: (self) => {
    gsap.to(self.target, { duration: 0.3, scale: 1.1 });
  }
});
```

### ScrollSmoother: Smooth Scroll Effect

ScrollSmoother creates a smooth, momentum-based scrollbar:

```javascript
let smoother = ScrollSmoother.create({
  smooth: 1,           // Smoothing duration in seconds
  effects: true,       // Enable effects on scrolled elements
  normalizeScroll: true, // Ignore browser scroll differences
  ignoreMobileResize: true
});

// Disable on mobile if needed
if (window.innerWidth < 768) {
  smoother.kill();
}
```

---

## Part 2: GSAP + React/Next.js Integration

### useGSAP Hook Pattern

The `useGSAP` hook (part of GSAP 3.12+) handles animations in React with automatic cleanup:

```bash
npm install gsap
```

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

export default function HeroSection() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Register plugins inside the hook
    gsap.registerPlugin(ScrollTrigger);

    // Create timeline
    const tl = gsap.timeline();
    tl.to('.hero-text', { duration: 0.8, opacity: 1, y: 0 })
      .to('.hero-image', { duration: 0.8, opacity: 1, scale: 1 }, 0.2);
  }, { scope: containerRef });

  return (
    <div ref={containerRef} className="hero">
      <h1 className="hero-text">Welcome</h1>
      <img className="hero-image" src="hero.jpg" alt="Hero" />
    </div>
  );
}
```

Key useGSAP benefits:
- Automatic cleanup on unmount
- Automatic ScrollTrigger refresh
- Scoped animations to component
- Dependency tracking

### useRef Patterns for DOM Control

Always use refs to target elements in React:

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

export default function CardList() {
  const containerRef = useRef(null);
  const cardsRef = useRef([]);

  useGSAP(() => {
    // Animate all cards
    gsap.from(cardsRef.current, {
      duration: 0.6,
      opacity: 0,
      y: 30,
      stagger: 0.1,
      ease: 'back.out'
    });
  }, { scope: containerRef });

  return (
    <div ref={containerRef} className="card-grid">
      {items.map((item, i) => (
        <div key={i} ref={(el) => cardsRef.current[i] = el} className="card">
          {item.name}
        </div>
      ))}
    </div>
  );
}
```

### gsap.context() for Cleanup

`gsap.context()` groups animations and ensures cleanup:

```javascript
useGSAP(() => {
  let ctx = gsap.context(() => {
    // All animations created inside this function
    // are grouped and can be reverted together
    gsap.to('.box', { duration: 1, x: 100 });
    gsap.to('.circle', { duration: 1, rotation: 360 });

    // ScrollTriggers created here are also tracked
    gsap.to('.section', {
      scrollTrigger: {
        trigger: '.section',
        scrub: 1,
      },
      x: 200
    });
  }, containerRef); // Scope to container

  // Automatic cleanup on unmount
  return () => ctx.revert();
}, { scope: containerRef });
```

### ScrollTrigger in Next.js App Router

ScrollTrigger requires careful handling in Next.js to avoid hydration mismatches:

```javascript
'use client'; // Client component required

import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';
import { useRef, useEffect } from 'react';

gsap.registerPlugin(ScrollTrigger);

export default function ScrollSection() {
  const containerRef = useRef(null);
  const isClientRef = useRef(false);

  // Ensure animations only run on client
  useEffect(() => {
    isClientRef.current = true;
  }, []);

  useGSAP(() => {
    if (!isClientRef.current) return;

    gsap.to('.scroll-item', {
      scrollTrigger: {
        trigger: '.scroll-item',
        start: 'top 80%',
        end: 'top 20%',
        scrub: 1,
      },
      x: 100,
      opacity: 1,
      duration: 1
    });

    // Important: refresh after layout shift
    return () => ScrollTrigger.getAll().forEach(trigger => trigger.kill());
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <div className="scroll-item">Scroll item</div>
    </div>
  );
}
```

### Avoiding Hydration Issues

Next.js hydration requires server-rendered HTML to match client HTML. Avoid:

```javascript
// BAD: Will cause hydration mismatch
export default function Counter() {
  useGSAP(() => {
    // This runs only on client, but might be in initial render
    gsap.to('.count', { innerHTML: 42 }); // Changes DOM
  });

  return <div className="count">0</div>; // Server renders 0
}

// GOOD: Use useEffect to ensure client-side only
export default function Counter() {
  const containerRef = useRef(null);
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true);
  }, []);

  useGSAP(() => {
    if (!isClient) return;
    gsap.to('.count', { textContent: 42, duration: 1 });
  }, { scope: containerRef, dependencies: [isClient] });

  return <div ref={containerRef} className="count">0</div>;
}
```

### Dynamic Animation Triggers

Conditional animations based on props:

```javascript
export default function AnimatedCard({ isVisible }) {
  const cardRef = useRef(null);

  useGSAP(() => {
    if (isVisible) {
      gsap.to(cardRef.current, {
        duration: 0.6,
        opacity: 1,
        y: 0,
        ease: 'back.out'
      });
    } else {
      gsap.to(cardRef.current, {
        duration: 0.3,
        opacity: 0,
        y: -20
      });
    }
  }, { dependencies: [isVisible], scope: cardRef });

  return <div ref={cardRef} className="card">Content</div>;
}
```

---

## Part 3: CSS Animations & When to Use Them

### CSS vs. GSAP Decision Matrix

Use CSS when:
- Simple hover states (1-2 properties)
- Fixed transitions (fade, slide)
- Reducing JavaScript bundle size
- Native scroll-linked animations (animation-timeline)

Use GSAP when:
- Timeline composition (multiple sequences)
- Complex stagger patterns
- Scroll-triggered animations with ScrollTrigger
- SplitText or advanced SVG effects
- Cross-browser timing precision required

### @keyframes & animation Property

```css
/* Define keyframes */
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* Apply animation */
.hero {
  animation: slideIn 0.8s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
  animation-delay: 0.2s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}

/* Shorthand */
.element {
  animation: slideIn 0.8s ease-out 0.2s 1 forwards;
}
```

### animation-timeline: Scroll-Linked CSS

Native scroll-linked CSS animations (no JavaScript):

```css
@supports (animation-timeline: view()) {
  .card {
    animation: cardReveal 1s ease-out forwards;
    animation-timeline: view();
    animation-range: entry 0% cover 30%;
  }

  @keyframes cardReveal {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
}

/* Scroll progress animation */
.progress-bar {
  animation: fillProgress 1s linear;
  animation-timeline: scroll(root inline);
}

@keyframes fillProgress {
  from {
    width: 0;
  }
  to {
    width: 100%;
  }
}
```

### View Transitions API

Smooth transitions between page states (modern browsers):

```javascript
if (!document.startViewTransition) {
  // Fallback for unsupported browsers
  updateDOM();
  return;
}

document.startViewTransition(() => {
  updateDOM(); // Update DOM inside transition
});
```

```css
/* Style view transition */
::view-transition-old(root) {
  animation: fadeOut 0.3s ease-out forwards;
}

::view-transition-new(root) {
  animation: fadeIn 0.3s ease-out forwards;
}

@keyframes fadeOut {
  to { opacity: 0; }
}

@keyframes fadeIn {
  from { opacity: 0; }
}
```

---

## Part 4: Framer Motion & Motion Library

### When to Use Framer Motion

Framer Motion excels at:
- React component animations (AnimatePresence)
- Gesture-driven animations
- Layout shifts and shared layout animations
- Exit animations without DOM listeners

### AnimatePresence Pattern

```javascript
import { motion, AnimatePresence } from 'framer-motion';

export default function Modal({ isOpen, onClose }) {
  return (
    <AnimatePresence mode="wait">
      {isOpen && (
        <motion.div
          key="modal"
          initial={{ opacity: 0, scale: 0.95 }}
          animate={{ opacity: 1, scale: 1 }}
          exit={{ opacity: 0, scale: 0.95 }}
          transition={{ duration: 0.2 }}
          className="modal"
        >
          <button onClick={onClose}>Close</button>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

### Layout Animations

```javascript
<motion.div layout>
  {/* Smooth re-layout animation on children change */}
  {children.map((child) => (
    <motion.div key={child.id} layout>
      {child.name}
    </motion.div>
  ))}
</motion.div>
```

### Gesture Animations

```javascript
<motion.button
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
  transition={{ type: 'spring', stiffness: 400, damping: 17 }}
>
  Hover or tap me
</motion.button>
```

### Framer vs. GSAP

| Feature | Framer Motion | GSAP |
|---------|---------------|------|
| React Integration | Native | Hook via useGSAP |
| Timeline Composition | Limited | Excellent |
| ScrollTrigger | External | Built-in free plugin |
| SplitText | No | Yes |
| Complexity | Simple | Advanced |
| Bundle Size | 30KB | 40KB |

**Recommendation**: Use Framer Motion for component-level animations (present/exit). Use GSAP for page-level sequences and scroll effects.

---

## Part 5: Scroll-Driven Animations Deep Dive

### Pin & Parallax Pattern

```javascript
useGSAP(() => {
  gsap.to('.parallax-image', {
    scrollTrigger: {
      trigger: '.parallax-section',
      start: 'top top',
      end: 'bottom top',
      scrub: 1,
      pin: '.parallax-section',
      markers: true,
    },
    y: 100,
    opacity: 1,
    duration: 1
  });
}, { scope: containerRef });
```

### Horizontal Scroll Gallery

```javascript
useGSAP(() => {
  let proxy = { skew: 0 },
      skewSetter = gsap.quickSetter('.gallery-wrapper', 'skewY', 'deg'),
      clamp = gsap.utils.clamp(-20, 20);

  gsap.set('.gallery-item', { transformOrigin: 'center center', force3D: true });

  Observer.create({
    type: 'scroll,touch',
    onChangeY: (self) => {
      let skew = clamp(self.getVelocity() / -300);
      gsap.to(proxy, {
        skew: skew,
        duration: 0.8,
        ease: 'power3',
        overwrite: true,
        onUpdate: () => skewSetter(proxy.skew)
      });
    }
  });

  gsap.set('.gallery-wrapper', { transformOrigin: 'center center' });

  let anim = gsap.to('.gallery-wrapper', {
    scrollTrigger: {
      trigger: '.gallery-section',
      start: 'top top',
      end: 'right right',
      scrub: 1,
      pin: true,
      snap: {
        snapTo: 'labels',
        duration: 0.3,
        delay: 0.1,
      },
    },
    x: -500,
    duration: 1
  });
}, { scope: containerRef });
```

### Snap Points

```javascript
gsap.to('.section', {
  scrollTrigger: {
    trigger: '.section',
    scrub: 1,
    snap: {
      snapTo: [0, 0.5, 1],     // Snap to 0%, 50%, 100%
      duration: 0.5,           // Snap animation duration
      delay: 0.1,              // Delay before snapping
      ease: 'power2.inOut'
    },
  },
  y: 100,
  duration: 1
});
```

### Reveal Animations with Batch

```javascript
useGSAP(() => {
  ScrollTrigger.batch('.reveal-item', {
    interval: 0.1,
    batchSize: 4,
    onEnter: (batch) => {
      gsap.to(batch, {
        opacity: 1,
        y: 0,
        duration: 0.6,
        stagger: { amount: 0.15, from: 'edges' },
        overwrite: 'auto'
      });
    },
    onLeave: (batch) => gsap.set(batch, { opacity: 0, y: 100, overwrite: 'auto' }),
    onEnterBack: (batch) => {
      gsap.to(batch, {
        opacity: 1,
        y: 0,
        duration: 0.6,
        stagger: { amount: 0.15, from: 'edges' },
        overwrite: 'auto'
      });
    },
    onLeaveBack: (batch) => gsap.set(batch, { opacity: 0, y: 100, overwrite: 'auto' }),
  });

  return () => ScrollTrigger.getAll().forEach(trigger => trigger.kill());
}, { scope: containerRef });
```

---

## Part 6: Advanced Text Animations

### Character-by-Character Reveal

```javascript
useGSAP(() => {
  const split = new SplitText('.heading', { type: 'chars,words,lines' });

  gsap.from(split.chars, {
    duration: 0.8,
    opacity: 0,
    y: 80,
    rotationZ: -10,
    transformOrigin: '0% 50%',
    stagger: {
      amount: 0.6,
      from: 'start'
    },
    ease: 'back.out',
    onComplete: () => split.revert()
  });
}, { scope: containerRef });
```

### Typewriter Effect

```javascript
useGSAP(() => {
  const text = '.typewriter-text';
  const split = new SplitText(text, { type: 'chars' });

  gsap.to(split.chars, {
    duration: 0.05,
    opacity: 1,
    stagger: 0.05,
    ease: 'none'
  });

  return () => split.revert();
}, { scope: containerRef });
```

### Counter Animation

```javascript
export default function Counter({ end = 100 }) {
  const numberRef = useRef(null);

  useGSAP(() => {
    const counter = { value: 0 };
    gsap.to(counter, {
      value: end,
      duration: 2,
      ease: 'power2.out',
      snap: { value: 1 },
      onUpdate: () => {
        if (numberRef.current) {
          numberRef.current.textContent = Math.floor(counter.value);
        }
      }
    });
  }, { scope: numberRef });

  return <div ref={numberRef} className="counter">0</div>;
}
```

### Blur-In Text

```javascript
useGSAP(() => {
  const split = new SplitText('.blur-text', { type: 'words' });

  gsap.from(split.words, {
    duration: 0.6,
    opacity: 0,
    filter: 'blur(10px)',
    stagger: 0.1,
    ease: 'circ.out',
    onComplete: () => split.revert()
  });
}, { scope: containerRef });
```

---

## Part 7: Page Transitions & Route Animations

### Next.js Page Transition with useGSAP

```javascript
'use client';

import { usePathname } from 'next/navigation';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';
import gsap from 'gsap';

export default function PageTransition({ children }) {
  const containerRef = useRef(null);
  const pathname = usePathname();

  useGSAP(() => {
    // Animate in on page change
    gsap.from(containerRef.current, {
      opacity: 0,
      y: 20,
      duration: 0.5,
      ease: 'power2.out'
    });
  }, { dependencies: [pathname], scope: containerRef });

  return <div ref={containerRef}>{children}</div>;
}
```

### Layout-Based Page Transitions

```javascript
// app/layout.js
import PageTransition from '@/components/PageTransition';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <PageTransition>{children}</PageTransition>
      </body>
    </html>
  );
}
```

### Exit Animation Pattern

```javascript
'use client';

import { useRouter } from 'next/navigation';
import gsap from 'gsap';

export default function Page() {
  const router = useRouter();

  const handleNavigate = async (href) => {
    // Fade out
    await gsap.to('main', {
      opacity: 0,
      duration: 0.3
    });
    // Navigate
    router.push(href);
  };

  return (
    <main>
      <button onClick={() => handleNavigate('/about')}>Go to About</button>
    </main>
  );
}
```

---

## Part 8: Performance Optimization Rules

### Rule 1: Only Animate Transform & Opacity

Performance-critical: animate only GPU-accelerated properties.

```javascript
// GOOD: Fast (GPU accelerated)
gsap.to('.box', {
  duration: 0.6,
  x: 100,           // transform: translateX()
  rotation: 360,    // transform: rotate()
  opacity: 0.5      // opacity
});

// BAD: Slow (triggers layout recalculation)
gsap.to('.box', {
  duration: 0.6,
  left: '100px',    // Position change = reflow
  width: '200px',   // Width change = repaint
  margin: '10px'    // Margin change = reflow
});
```

### Rule 2: will-change CSS Property

Signal browser to optimize before animation:

```css
.hero-element {
  will-change: transform, opacity;
}

.parallax {
  will-change: transform;
}

/* Remove after animation */
.animated-out {
  will-change: auto;
}
```

```javascript
useGSAP(() => {
  // Add will-change before animation
  gsap.set('.element', { willChange: 'transform, opacity' });

  gsap.to('.element', {
    duration: 1,
    x: 100,
    opacity: 0.5,
    onComplete: () => {
      gsap.set('.element', { willChange: 'auto' });
    }
  });
}, { scope: containerRef });
```

### Rule 3: GPU Compositing

Force GPU compositing with `transform3d`:

```javascript
gsap.set('.layer', {
  force3D: true,  // Creates GPU layer
  perspective: 1000
});

// Or via CSS
.gpu-accelerated {
  transform: translate3d(0, 0, 0);
  perspective: 1000px;
}
```

### Rule 4: Batch Updates

Use `gsap.context()` to group DOM reads/writes:

```javascript
useGSAP(() => {
  let ctx = gsap.context(() => {
    // All DOM reads/writes grouped efficiently
    gsap.to('.box', { x: 100 });
    gsap.to('.circle', { rotation: 360 });
    gsap.to('.rect', { y: 50 });
  }, containerRef);

  return () => ctx.revert();
}, { scope: containerRef });
```

### Rule 5: requestAnimationFrame Instead of setInterval

```javascript
// BAD: Misses frames
setInterval(() => {
  element.style.transform = `translateX(${x}px)`;
}, 16);

// GOOD: Syncs with browser refresh rate
let animationFrameId;
const animate = () => {
  element.style.transform = `translateX(${x}px)`;
  animationFrameId = requestAnimationFrame(animate);
};
animationFrameId = requestAnimationFrame(animate);
```

GSAP uses `requestAnimationFrame` internally, so use GSAP for timing-critical animations.

### Rule 6: Debounce Resize Events

```javascript
let resizeTimer;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => {
    ScrollTrigger.refresh();
  }, 100);
});
```

---

## Part 9: Anti-Patterns & What NOT to Do

### Anti-Pattern 1: Animating Everything

Not every element needs animation. Over-animation causes:
- Cognitive overload
- Slower perceived performance
- Eye fatigue
- Accessibility violations

```javascript
// BAD: Too much animation
gsap.to('.icon', { rotation: 360, duration: 1, repeat: -1 });
gsap.to('.text', { opacity: [1, 0, 1], duration: 0.5, repeat: -1 });
gsap.to('.border', { borderColor: ['red', 'blue', 'green'], duration: 0.3, repeat: -1 });

// GOOD: Strategic animation
gsap.to('.cta-button', {
  duration: 0.6,
  scale: 1.05,
  boxShadow: '0 10px 30px rgba(0,0,0,0.3)'
}, 'on-scroll');
```

### Anti-Pattern 2: Blocking Scroll

Never prevent natural scroll behavior:

```javascript
// BAD: Blocks scroll
Observer.create({
  type: 'scroll',
  onChangeY: (self) => {
    event.preventDefault(); // DON'T DO THIS
  }
});

// GOOD: Work with scroll
Observer.create({
  type: 'scroll',
  onChangeY: (self) => {
    // Update animation based on scroll, don't block it
    gsap.to('.element', { y: self.y / 10 });
  }
});
```

### Anti-Pattern 3: Ignoring prefers-reduced-motion

Always respect user accessibility preferences:

```javascript
// BAD: Ignores accessibility
gsap.to('.element', {
  duration: 1,
  x: 100
});

// GOOD: Check prefers-reduced-motion
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

const duration = prefersReducedMotion ? 0 : 1;
gsap.to('.element', {
  duration: duration,
  x: 100
});

// Or in React
const useAnimationDuration = () => {
  const [prefersReduced, setPrefersReduced] = useState(false);

  useEffect(() => {
    const mq = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReduced(mq.matches);
    mq.addListener(() => setPrefersReduced(mq.matches));
  }, []);

  return prefersReduced ? 0 : 1;
};
```

### Anti-Pattern 4: Using GSAP for Simple Hover States

CSS is better for simple interactions:

```javascript
// BAD: GSAP for simple hover
gsap.to('.button', {
  duration: 0.3,
  scale: 1.05,
  backgroundColor: '#FF0000'
});

// GOOD: Use CSS
.button {
  transition: all 0.3s ease-out;
}

.button:hover {
  transform: scale(1.05);
  background-color: #FF0000;
}
```

### Anti-Pattern 5: Not Cleaning Up Timelines

Always kill timelines and ScrollTriggers on unmount:

```javascript
// BAD: Memory leak
useGSAP(() => {
  gsap.to('.element', {
    scrollTrigger: {
      trigger: '.section',
      scrub: 1,
    },
    x: 100
  });
  // No cleanup!
});

// GOOD: Kill on unmount
useGSAP(() => {
  const tl = gsap.timeline();
  tl.to('.element', { x: 100, duration: 1 });

  return () => {
    tl.kill();
    ScrollTrigger.getAll().forEach(trigger => trigger.kill());
  };
}, { scope: containerRef });
```

---

## Part 10: Production Checklist

Before deploying animations:

- [ ] All animations test on Chrome, Firefox, Safari, Edge (desktop)
- [ ] Mobile: test on iOS Safari, Chrome Android
- [ ] ScrollTrigger refreshes on window resize
- [ ] Timelines killed on component unmount
- [ ] Only transform and opacity animated
- [ ] will-change removed after animation
- [ ] prefers-reduced-motion respected
- [ ] No layout shifts during animations
- [ ] Lighthouse performance > 90
- [ ] Core Web Vitals: LCP, FID, CLS green
- [ ] Accessibility: tab navigation works, animations don't trap focus
- [ ] No console errors or warnings
- [ ] Images lazy-loaded before animation
- [ ] RequestAnimationFrame used for timing-critical updates

---

## Conclusion

Master motion in 2026 by understanding when to use GSAP, CSS, or Framer Motion. Build performant, accessible animations with ScrollTrigger, SplitText, and timeline composition. Integrate seamlessly with React/Next.js using useGSAP. Respect user preferences and browsers. Animate strategically, not constantly.

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

<sub>Open RX by CutTheChexx — The Prescription.</sub>
