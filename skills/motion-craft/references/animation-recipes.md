# Motion Craft: Animation Recipes

Production-ready animation patterns. Copy, paste, customize.

---

## Recipe 1: Hero Section Page Load Sequence

Full-screen hero with staggered text reveal and image fade.

```javascript
'use client';

import gsap from 'gsap';
import { SplitText } from 'gsap/SplitText';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(SplitText);

export default function HeroSection() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Split hero title into characters
    const titleSplit = new SplitText('.hero-title', {
      type: 'chars,words'
    });

    // Main timeline
    const tl = gsap.timeline();

    // Character reveal (staggered)
    tl.from(titleSplit.chars, {
      duration: 0.6,
      opacity: 0,
      y: 30,
      rotationX: -90,
      transformOrigin: '0% 50%',
      stagger: {
        amount: 0.4,
        from: 'start',
        ease: 'power2.out'
      }
    }, 0);

    // Subtitle fade in (parallel)
    tl.from('.hero-subtitle', {
      duration: 0.8,
      opacity: 0,
      y: 20
    }, 0.2);

    // Image scale and fade
    tl.from('.hero-image', {
      duration: 1,
      opacity: 0,
      scale: 0.9,
      ease: 'back.out'
    }, 0.3);

    // CTA button
    tl.from('.cta-button', {
      duration: 0.6,
      opacity: 0,
      y: 20,
      ease: 'back.out'
    }, 0.6);

    // Cleanup
    return () => titleSplit.revert();
  }, { scope: containerRef });

  return (
    <div ref={containerRef} className="hero">
      <div className="hero-content">
        <h1 className="hero-title">Design. Animate. Ship.</h1>
        <p className="hero-subtitle">Build motion like never before</p>
        <button className="cta-button">Get Started</button>
      </div>
      <img className="hero-image" src="/hero.jpg" alt="Hero" />
    </div>
  );
}
```

**CSS:**
```css
.hero {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  min-height: 100vh;
  padding: 4rem;
  gap: 3rem;
}

.hero-title {
  font-size: 4rem;
  font-weight: 800;
  line-height: 1.1;
  margin: 0;
}

.hero-subtitle {
  font-size: 1.25rem;
  color: #666;
  margin-top: 1rem;
}

.hero-image {
  width: 100%;
  border-radius: 12px;
  object-fit: cover;
}

.cta-button {
  margin-top: 2rem;
  padding: 1rem 2rem;
  font-size: 1rem;
  font-weight: 600;
  background: #000;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
```

---

## Recipe 2: Scroll-Triggered Section Reveals

Staggered element reveal on scroll.

```javascript
'use client';

import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(ScrollTrigger);

export default function RevealSection() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Batch reveal pattern
    ScrollTrigger.batch('.reveal-card', {
      interval: 0.1,
      batchSize: 3,
      onEnter: (batch) => {
        gsap.to(batch, {
          opacity: 1,
          y: 0,
          duration: 0.6,
          stagger: {
            amount: 0.15,
            from: 'edges'
          },
          overwrite: 'auto'
        });
      },
      onLeave: (batch) => {
        gsap.set(batch, {
          opacity: 0,
          y: 50,
          overwrite: 'auto'
        });
      },
      onEnterBack: (batch) => {
        gsap.to(batch, {
          opacity: 1,
          y: 0,
          duration: 0.6,
          stagger: {
            amount: 0.15,
            from: 'edges'
          },
          overwrite: 'auto'
        });
      }
    });

    return () => ScrollTrigger.getAll().forEach(trigger => trigger.kill());
  }, { scope: containerRef });

  return (
    <section ref={containerRef} className="reveal-section">
      <h2>Our Features</h2>
      <div className="card-grid">
        {[1, 2, 3, 4, 5, 6].map((i) => (
          <div key={i} className="reveal-card">
            <h3>Feature {i}</h3>
            <p>Description of feature</p>
          </div>
        ))}
      </div>
    </section>
  );
}
```

**CSS:**
```css
.reveal-section {
  padding: 6rem 2rem;
}

.reveal-section h2 {
  font-size: 2.5rem;
  margin-bottom: 3rem;
  text-align: center;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.reveal-card {
  opacity: 0;
  transform: translateY(50px);
  padding: 2rem;
  background: #f5f5f5;
  border-radius: 12px;
  transition: all 0.3s ease;
}

.reveal-card:hover {
  transform: translateY(0) translateY(-8px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
}
```

---

## Recipe 3: Horizontal Scroll Gallery

Full-screen carousel with smooth scroll animation.

```javascript
'use client';

import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(ScrollTrigger);

export default function HorizontalGallery() {
  const containerRef = useRef(null);
  const wrapperRef = useRef(null);

  useGSAP(() => {
    // Get total scrollable width
    const gallery = wrapperRef.current;
    const scrollWidth = gallery.scrollWidth;

    // Animate horizontal scroll
    gsap.to(gallery, {
      scrollTrigger: {
        trigger: containerRef.current,
        start: 'top top',
        end: 'bottom bottom',
        scrub: 1,
        pin: true,
        markers: false,
      },
      x: -(scrollWidth - window.innerWidth),
      duration: 4,
      ease: 'none'
    });

    return () => ScrollTrigger.getAll().forEach(t => t.kill());
  }, { scope: containerRef });

  return (
    <section ref={containerRef} className="h-scroll-section">
      <div ref={wrapperRef} className="h-scroll-wrapper">
        {[1, 2, 3, 4, 5].map((i) => (
          <div key={i} className="h-scroll-item">
            <img src={`/gallery-${i}.jpg`} alt={`Gallery ${i}`} />
            <h3>Project {i}</h3>
          </div>
        ))}
      </div>
    </section>
  );
}
```

**CSS:**
```css
.h-scroll-section {
  width: 100%;
  overflow: hidden;
}

.h-scroll-wrapper {
  display: flex;
  gap: 2rem;
  width: fit-content;
}

.h-scroll-item {
  flex-shrink: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.h-scroll-item img {
  width: 80%;
  height: 80%;
  object-fit: cover;
  border-radius: 12px;
}

.h-scroll-item h3 {
  margin-top: 2rem;
  font-size: 1.5rem;
}
```

---

## Recipe 4: Number Counter Animation

Animated counter from 0 to target number.

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

export default function CounterAnimation({ end = 1000, suffix = '', duration = 2 }) {
  const numberRef = useRef(null);

  useGSAP(() => {
    const counter = { value: 0 };

    gsap.to(counter, {
      value: end,
      duration: duration,
      ease: 'power2.out',
      snap: { value: 1 },
      onUpdate: () => {
        if (numberRef.current) {
          numberRef.current.textContent = Math.floor(counter.value).toLocaleString() + suffix;
        }
      }
    });
  }, { scope: numberRef });

  return <div ref={numberRef} className="counter">0{suffix}</div>;
}
```

**Usage:**
```javascript
<CounterAnimation end={42} suffix="%" />
<CounterAnimation end={500000} suffix=" users" />
<CounterAnimation end={99.9} suffix="px" duration={3} />
```

**CSS:**
```css
.counter {
  font-size: 3rem;
  font-weight: 800;
  font-variant-numeric: tabular-nums;
}
```

---

## Recipe 5: Magnetic Button Effect

Button that follows cursor within a radius.

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef, useState } from 'react';

export default function MagneticButton({ children, onClick }) {
  const buttonRef = useRef(null);
  const [isHovering, setIsHovering] = useState(false);

  useGSAP(() => {
    const handleMouseMove = (e) => {
      if (!isHovering) return;

      const button = buttonRef.current;
      const rect = button.getBoundingClientRect();
      const centerX = rect.left + rect.width / 2;
      const centerY = rect.top + rect.height / 2;

      const distance = 80; // Magnetic radius
      const x = e.clientX - centerX;
      const y = e.clientY - centerY;
      const dist = Math.sqrt(x * x + y * y);

      if (dist < distance) {
        const strength = (distance - dist) / distance;
        gsap.to(button, {
          x: x * strength * 0.5,
          y: y * strength * 0.5,
          duration: 0.3,
          overwrite: 'auto'
        });
      }
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, { dependencies: [isHovering] });

  const handleMouseEnter = () => {
    setIsHovering(true);
  };

  const handleMouseLeave = () => {
    setIsHovering(false);
    gsap.to(buttonRef.current, {
      x: 0,
      y: 0,
      duration: 0.3
    });
  };

  return (
    <button
      ref={buttonRef}
      onMouseEnter={handleMouseEnter}
      onMouseLeave={handleMouseLeave}
      onClick={onClick}
      className="magnetic-button"
    >
      {children}
    </button>
  );
}
```

**CSS:**
```css
.magnetic-button {
  padding: 1rem 2rem;
  font-size: 1rem;
  font-weight: 600;
  background: #000;
  color: #fff;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s ease;
  will-change: transform;
}

.magnetic-button:active {
  transform: scale(0.95);
}
```

---

## Recipe 6: Parallax Hero Section

Background moves slower than foreground for depth effect.

```javascript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(ScrollTrigger);

export default function ParallaxHero() {
  const containerRef = useRef(null);
  const bgRef = useRef(null);

  useGSAP(() => {
    gsap.to(bgRef.current, {
      scrollTrigger: {
        trigger: containerRef.current,
        start: 'top top',
        end: 'bottom top',
        scrub: 1,
      },
      y: 100,
      ease: 'none',
      duration: 1
    });

    return () => ScrollTrigger.getAll().forEach(t => t.kill());
  }, { scope: containerRef });

  return (
    <div ref={containerRef} className="parallax-hero">
      <div ref={bgRef} className="parallax-bg">
        <img src="/bg-image.jpg" alt="Background" />
      </div>
      <div className="parallax-content">
        <h1>Scroll Down</h1>
        <p>Watch the parallax effect</p>
      </div>
    </div>
  );
}
```

**CSS:**
```css
.parallax-hero {
  position: relative;
  height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.parallax-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 120%;
  will-change: transform;
}

.parallax-bg img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.parallax-content {
  position: relative;
  z-index: 10;
  text-align: center;
  color: white;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.parallax-content h1 {
  font-size: 4rem;
  margin: 0;
}
```

---

## Recipe 7: 3D Card Tilt on Hover

Card tilts toward cursor for interactive depth.

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef, useState } from 'react';

export default function Tilt3DCard() {
  const cardRef = useRef(null);
  const [isHovering, setIsHovering] = useState(false);

  useGSAP(() => {
    const handleMouseMove = (e) => {
      if (!isHovering || !cardRef.current) return;

      const rect = cardRef.current.getBoundingClientRect();
      const centerX = rect.left + rect.width / 2;
      const centerY = rect.top + rect.height / 2;

      const x = e.clientX - centerX;
      const y = e.clientY - centerY;

      const rotateX = (y / (rect.height / 2)) * -15;
      const rotateY = (x / (rect.width / 2)) * 15;

      gsap.to(cardRef.current, {
        rotationX: rotateX,
        rotationY: rotateY,
        duration: 0.3,
        overwrite: 'auto'
      });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, { dependencies: [isHovering] });

  return (
    <div
      ref={cardRef}
      onMouseEnter={() => setIsHovering(true)}
      onMouseLeave={() => {
        setIsHovering(false);
        gsap.to(cardRef.current, {
          rotationX: 0,
          rotationY: 0,
          duration: 0.6
        });
      }}
      className="tilt-card"
    >
      <div className="tilt-card-content">
        <h2>3D Tilt Card</h2>
        <p>Hover to interact</p>
      </div>
    </div>
  );
}
```

**CSS:**
```css
.tilt-card {
  width: 300px;
  height: 400px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  padding: 2rem;
  color: white;
  will-change: transform;
  transform-style: preserve-3d;
  perspective: 1000px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  transition: box-shadow 0.3s ease;
}

.tilt-card:hover {
  box-shadow: 0 40px 80px rgba(0, 0, 0, 0.3);
}

.tilt-card-content {
  transform-style: preserve-3d;
}

.tilt-card h2 {
  transform: translateZ(50px);
}
```

---

## Recipe 8: Staggered Grid Reveal with Blur

Grid items reveal with blur-in effect.

```javascript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(ScrollTrigger);

export default function StaggeredGrid() {
  const containerRef = useRef(null);

  useGSAP(() => {
    gsap.from('.grid-item', {
      scrollTrigger: {
        trigger: containerRef.current,
        start: 'top 70%',
      },
      opacity: 0,
      filter: 'blur(10px)',
      scale: 0.95,
      duration: 0.8,
      stagger: {
        amount: 0.6,
        from: 'edges',
        grid: [4, 2]
      },
      ease: 'back.out'
    });

    return () => ScrollTrigger.getAll().forEach(t => t.kill());
  }, { scope: containerRef });

  return (
    <div ref={containerRef} className="grid-container">
      <h2>Gallery</h2>
      <div className="grid">
        {[1, 2, 3, 4, 5, 6, 7, 8].map((i) => (
          <div key={i} className="grid-item">
            <img src={`/item-${i}.jpg`} alt={`Item ${i}`} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

**CSS:**
```css
.grid-container {
  padding: 4rem 2rem;
}

.grid-container h2 {
  font-size: 2.5rem;
  margin-bottom: 3rem;
  text-align: center;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}

.grid-item {
  aspect-ratio: 1;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.3s ease;
}

.grid-item:hover {
  transform: scale(1.05);
}

.grid-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

---

## Recipe 9: Typewriter Text Effect

Character-by-character text reveal.

```javascript
import gsap from 'gsap';
import { SplitText } from 'gsap/SplitText';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(SplitText);

export default function TypewriterText({ text }) {
  const textRef = useRef(null);

  useGSAP(() => {
    const split = new SplitText(textRef.current, {
      type: 'chars'
    });

    gsap.from(split.chars, {
      duration: 0.05,
      opacity: 0,
      y: 10,
      stagger: 0.05,
      ease: 'back.out'
    });

    return () => split.revert();
  }, { scope: textRef });

  return (
    <p ref={textRef} className="typewriter-text">
      {text}
    </p>
  );
}
```

**Usage:**
```javascript
<TypewriterText text="This text types out character by character." />
```

---

## Recipe 10: Scroll Progress Indicator

Bar that fills as page scrolls.

```javascript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(ScrollTrigger);

export default function ScrollProgress() {
  const barRef = useRef(null);

  useGSAP(() => {
    gsap.to(barRef.current, {
      scrollTrigger: {
        trigger: 'body',
        start: 'top top',
        end: 'bottom bottom',
        scrub: true,
      },
      width: '100%',
      duration: 1,
      ease: 'none'
    });

    return () => ScrollTrigger.getAll().forEach(t => t.kill());
  }, { scope: barRef });

  return <div ref={barRef} className="scroll-progress" />;
}
```

**CSS:**
```css
.scroll-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 4px;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  width: 0;
  z-index: 1000;
  will-change: width;
}
```

---

## Recipe 11: Tooltip with Animation

Animated tooltip on hover.

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef, useState } from 'react';

export default function AnimatedTooltip({ text, children }) {
  const tooltipRef = useRef(null);
  const [isVisible, setIsVisible] = useState(false);

  useGSAP(() => {
    if (isVisible) {
      gsap.to(tooltipRef.current, {
        opacity: 1,
        y: -10,
        duration: 0.3,
        ease: 'back.out',
        pointerEvents: 'auto'
      });
    } else {
      gsap.to(tooltipRef.current, {
        opacity: 0,
        y: 0,
        duration: 0.2,
        pointerEvents: 'none'
      });
    }
  }, { dependencies: [isVisible] });

  return (
    <div className="tooltip-wrapper">
      <div
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
      >
        {children}
      </div>
      <div ref={tooltipRef} className="tooltip">
        {text}
      </div>
    </div>
  );
}
```

**CSS:**
```css
.tooltip-wrapper {
  position: relative;
  display: inline-block;
}

.tooltip {
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: #000;
  color: #fff;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.875rem;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  margin-bottom: 0.5rem;
}

.tooltip::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 5px solid transparent;
  border-top-color: #000;
}
```

---

## Recipe 12: Text Split with Line Animation

Animate text by lines with reveal effect.

```javascript
import gsap from 'gsap';
import { SplitText } from 'gsap/SplitText';
import { useGSAP } from '@gsap/react';
import { useRef } from 'react';

gsap.registerPlugin(SplitText);

export default function LineSplitAnimation() {
  const textRef = useRef(null);

  useGSAP(() => {
    const split = new SplitText(textRef.current, {
      type: 'lines',
      linesClass: 'split-line'
    });

    gsap.from(split.lines, {
      duration: 0.8,
      opacity: 0,
      y: 30,
      stagger: 0.15,
      ease: 'power2.out'
    });

    return () => split.revert();
  }, { scope: textRef });

  return (
    <div ref={textRef} className="split-text">
      <p>
        This is a paragraph split into lines.
        Each line animates in sequence.
        Create engaging text reveals.
      </p>
    </div>
  );
}
```

---

## Recipe 13: Animated Burger Menu

Hamburger icon with animated lines.

```javascript
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { useRef, useState } from 'react';

export default function BurgerMenu() {
  const topRef = useRef(null);
  const middleRef = useRef(null);
  const bottomRef = useRef(null);
  const [isOpen, setIsOpen] = useState(false);

  useGSAP(() => {
    if (isOpen) {
      gsap.to(topRef.current, {
        rotation: 45,
        y: 12,
        duration: 0.3
      });
      gsap.to(middleRef.current, {
        opacity: 0,
        duration: 0.3
      });
      gsap.to(bottomRef.current, {
        rotation: -45,
        y: -12,
        duration: 0.3
      });
    } else {
      gsap.to(topRef.current, {
        rotation: 0,
        y: 0,
        duration: 0.3
      });
      gsap.to(middleRef.current, {
        opacity: 1,
        duration: 0.3
      });
      gsap.to(bottomRef.current, {
        rotation: 0,
        y: 0,
        duration: 0.3
      });
    }
  }, { dependencies: [isOpen] });

  return (
    <button
      className="burger-menu"
      onClick={() => setIsOpen(!isOpen)}
      aria-label="Toggle menu"
    >
      <span ref={topRef} className="burger-line" />
      <span ref={middleRef} className="burger-line" />
      <span ref={bottomRef} className="burger-line" />
    </button>
  );
}
```

**CSS:**
```css
.burger-menu {
  width: 50px;
  height: 50px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 6px;
  background: transparent;
  border: none;
  cursor: pointer;
}

.burger-line {
  display: block;
  width: 30px;
  height: 3px;
  background: #000;
  border-radius: 2px;
  will-change: transform;
}
```

---

## Tips & Tricks

1. **Always cleanup**: Use useGSAP return function to kill timelines
2. **Batch operations**: Use ScrollTrigger.batch() for efficiency
3. **Prefers reduced motion**: Check user preferences before animating
4. **GPU acceleration**: Force3D: true for transform animations
5. **Easing**: Use 'back.out', 'elastic.out', 'power2.out' for smooth feels
6. **Stagger from edges**: for grid reveals, stagger: { from: 'edges' }
7. **Mobile first**: Disable heavy animations on mobile devices
8. **Test performance**: Use Chrome DevTools performance tab

---

End of Animation Recipes. Happy animating.
