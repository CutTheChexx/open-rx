# Component Recipes — ui-arsenal Reference

Production-ready component code snippets for immediate use.

---

## shadcn/ui Theme Setup: CSS Variables & Dark Mode

### Base CSS Variables Configuration

```css
/* globals.css */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.6%;
    --card: 0 0% 100%;
    --card-foreground: 0 0% 3.6%;
    --popover: 0 0% 100%;
    --popover-foreground: 0 0% 3.6%;
    --muted: 0 0% 96.1%;
    --muted-foreground: 0 0% 45.1%;
    --accent: 0 0% 9%;
    --accent-foreground: 0 0% 100%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 100%;
    --border: 0 0% 89.8%;
    --input: 0 0% 89.8%;
    --ring: 0 0% 3.6%;
    --radius: 0.5rem;
    --primary: 0 0% 9%;
    --primary-foreground: 0 0% 100%;
    --secondary: 0 0% 96.1%;
    --secondary-foreground: 0 0% 9%;
  }

  .dark {
    --background: 0 0% 3.6%;
    --foreground: 0 0% 98%;
    --card: 0 0% 3.6%;
    --card-foreground: 0 0% 98%;
    --popover: 0 0% 3.6%;
    --popover-foreground: 0 0% 98%;
    --muted: 0 0% 14.9%;
    --muted-foreground: 0 0% 63.9%;
    --accent: 0 0% 98%;
    --accent-foreground: 0 0% 9%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 9%;
    --border: 0 0% 14.9%;
    --input: 0 0% 14.9%;
    --ring: 0 0% 83.1%;
    --primary: 0 0% 98%;
    --primary-foreground: 0 0% 9%;
    --secondary: 0 0% 14.9%;
    --secondary-foreground: 0 0% 98%;
  }

  * {
    @apply border-border;
  }

  body {
    @apply bg-background text-foreground;
  }
}
```

### Dark Mode Toggle Component

```typescript
// components/theme-toggle.tsx
'use client'

import { useTheme } from 'next-themes'
import { Moon, Sun } from 'lucide-react'
import { Button } from '@/components/ui/button'

export function ThemeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      <Sun className="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
      <span className="sr-only">Toggle theme</span>
    </Button>
  )
}
```

### Custom Color Theme Override

```css
/* For custom branding */
:root {
  --primary: 262 80% 50%; /* Purple brand */
  --secondary: 198 93% 60%; /* Cyan accent */
  --accent: 40 96% 40%; /* Orange pop */
  --destructive: 0 84% 60%; /* Red for errors */
}

/* Or dynamic via CSS properties */
[data-theme='acme'] {
  --primary: 210 100% 50%;
  --secondary: 30 100% 50%;
  --accent: 260 100% 50%;
}
```

---

## Aceternity UI Integration Setup for Next.js

### Installation & Configuration

```bash
npm install framer-motion clsx tailwind-merge
```

```typescript
// tsconfig.json or jsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["components/*"],
      "@/lib/*": ["lib/*"],
      "@/ui/*": ["components/ui/*"]
    }
  }
}
```

### Minimal Aceternity Setup Component

```typescript
// components/ui/aceternity-wrapper.tsx
'use client'

import { ReactNode } from 'react'

export function AceternitySafeWrapper({ children }: { children: ReactNode }) {
  return <div className="w-full">{children}</div>
}
```

### 3D Card Component Setup

```typescript
// components/ui/3d-card.tsx
'use client'

import React, { useState, useRef, useEffect } from 'react'
import { motion } from 'framer-motion'

interface CardContainerProps {
  children: React.ReactNode
  className?: string
}

export const CardContainer: React.FC<CardContainerProps> = ({ children, className = '' }) => {
  const [mousePosition, setMousePosition] = useState({ x: 0, y: 0 })
  const [isHovering, setIsHovering] = useState(false)
  const ref = useRef<HTMLDivElement>(null)

  const handleMouseMove = (e: React.MouseEvent<HTMLDivElement>) => {
    if (!ref.current) return

    const rect = ref.current.getBoundingClientRect()
    setMousePosition({
      x: e.clientX - rect.left,
      y: e.clientY - rect.top,
    })
  }

  const rotateX = isHovering ? (mousePosition.y - ref.current?.clientHeight! / 2) / 10 : 0
  const rotateY = isHovering ? (mousePosition.x - ref.current?.clientWidth! / 2) / 10 : 0

  return (
    <motion.div
      ref={ref}
      onMouseMove={handleMouseMove}
      onMouseEnter={() => setIsHovering(true)}
      onMouseLeave={() => setIsHovering(false)}
      animate={{ rotateX, rotateY }}
      className={`perspective ${className}`}
      style={{
        transformStyle: 'preserve-3d',
      }}
    >
      {children}
    </motion.div>
  )
}

interface CardBodyProps {
  children: React.ReactNode
  className?: string
}

export const CardBody: React.FC<CardBodyProps> = ({ children, className = '' }) => {
  return (
    <motion.div
      className={className}
      style={{
        transformStyle: 'preserve-3d',
      }}
    >
      {children}
    </motion.div>
  )
}

interface CardItemProps {
  as?: keyof JSX.IntrinsicElements
  children: React.ReactNode
  className?: string
  translateZ?: number
}

export const CardItem: React.FC<CardItemProps> = ({
  as: Component = 'div',
  children,
  className = '',
  translateZ = 0,
}) => {
  return (
    <motion.div
      as={Component as any}
      className={className}
      style={{
        transformStyle: 'preserve-3d',
        transform: `translateZ(${translateZ}px)`,
      }}
    >
      {children}
    </motion.div>
  )
}
```

---

## Class-Variance-Authority (CVA) Patterns

### Basic CVA Setup

```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### Button Component with CVA

```typescript
// components/ui/button.tsx
import * as React from 'react'
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      ref={ref}
      {...props}
    />
  )
)

Button.displayName = 'Button'

export { Button, buttonVariants }
```

### Advanced CVA with Compound Variants

```typescript
// components/ui/alert.tsx
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const alertVariants = cva(
  'relative w-full rounded-lg border p-4 [&>svg+div]:translate-y-[-3px] [&>svg]:absolute [&>svg]:left-4 [&>svg]:top-4 [&>svg]:text-foreground [&>svg~*]:pl-7',
  {
    variants: {
      variant: {
        default: 'bg-background text-foreground',
        destructive: 'border-destructive/50 text-destructive dark:border-destructive [&>svg]:text-destructive',
        success: 'border-green-500/50 text-green-700 dark:text-green-400 [&>svg]:text-green-600 dark:border-green-500',
        warning: 'border-yellow-500/50 text-yellow-700 dark:text-yellow-400 [&>svg]:text-yellow-600 dark:border-yellow-500',
      },
      density: {
        compact: 'py-2 px-3',
        default: 'p-4',
        spacious: 'p-6',
      },
    },
    compoundVariants: [
      {
        variant: 'destructive',
        density: 'compact',
        className: 'py-2 px-3 text-sm',
      },
      {
        variant: 'success',
        density: 'spacious',
        className: 'py-6 px-6 bg-green-50 dark:bg-green-950',
      },
    ],
    defaultVariants: {
      variant: 'default',
      density: 'default',
    },
  }
)

export interface AlertProps
  extends React.HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof alertVariants> {}

const Alert = React.forwardRef<HTMLDivElement, AlertProps>(
  ({ className, variant, density, ...props }, ref) => (
    <div
      ref={ref}
      role="alert"
      className={cn(alertVariants({ variant, density }), className)}
      {...props}
    />
  )
)

Alert.displayName = 'Alert'

export { Alert, alertVariants }
```

### cn() Utility Usage Patterns

```typescript
// Pattern 1: Conditional classes
const isActive = true
const isDisabled = false

const buttonClasses = cn(
  'px-4 py-2 rounded font-medium',
  isActive && 'bg-blue-600 text-white',
  isDisabled && 'opacity-50 cursor-not-allowed'
)

// Pattern 2: Variant selection
const variant = 'outline'

const cardClasses = cn(
  'rounded-lg p-4 border',
  variant === 'outline' && 'border-gray-200 bg-white',
  variant === 'filled' && 'border-none bg-gray-100',
  variant === 'glass' && 'border-white/20 bg-white/10 backdrop-blur'
)

// Pattern 3: Responsive combinations
const gridClasses = cn(
  'grid gap-4',
  'grid-cols-1',
  'md:grid-cols-2',
  'lg:grid-cols-3',
  'xl:grid-cols-4'
)

// Pattern 4: Component composition
function Card({ variant, children, className }: any) {
  return (
    <div
      className={cn(
        'rounded-lg border p-6',
        variant === 'primary' && 'bg-blue-50 border-blue-200',
        variant === 'secondary' && 'bg-gray-50 border-gray-200',
        className // Allow overrides
      )}
    >
      {children}
    </div>
  )
}
```

---

## Production Component Recipes

### Recipe 1: Hero Section with Gradient & CTA

```typescript
// components/sections/hero.tsx
'use client'

import { Button } from '@/components/ui/button'
import { ArrowRight } from 'lucide-react'
import { motion } from 'framer-motion'

export function HeroSection() {
  return (
    <section className="relative overflow-hidden bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 py-20 sm:py-32">
      {/* Animated background blobs */}
      <div className="absolute top-0 left-1/4 w-96 h-96 bg-blue-500/20 rounded-full mix-blend-multiply filter blur-3xl animate-blob opacity-70" />
      <div className="absolute top-0 right-1/4 w-96 h-96 bg-purple-500/20 rounded-full mix-blend-multiply filter blur-3xl animate-blob animation-delay-2000 opacity-70" />
      <div className="absolute -bottom-8 left-1/2 w-96 h-96 bg-pink-500/20 rounded-full mix-blend-multiply filter blur-3xl animate-blob animation-delay-4000 opacity-70" />

      {/* Content */}
      <div className="relative z-10 mx-auto max-w-7xl px-6 sm:px-8 text-center">
        <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.8 }}>
          <div className="inline-flex items-center rounded-full border border-white/20 bg-white/10 px-4 py-2 mb-8 backdrop-blur-sm">
            <span className="text-sm font-medium text-white/80">Introducing v2.0</span>
          </div>

          <h1 className="text-6xl sm:text-7xl font-bold text-white leading-tight mb-6">
            Build faster with <span className="text-transparent bg-clip-text bg-gradient-to-r from-blue-400 via-purple-400 to-pink-400">modern components</span>
          </h1>

          <p className="text-xl text-gray-400 max-w-2xl mx-auto mb-12 leading-relaxed">
            Production-ready UI components built with React, TypeScript, and Tailwind CSS. Copy, customize, and ship.
          </p>

          <div className="flex gap-4 justify-center flex-wrap">
            <Button size="lg" className="gap-2">
              Get Started
              <ArrowRight className="w-4 h-4" />
            </Button>
            <Button size="lg" variant="outline">
              View Docs
            </Button>
          </div>
        </motion.div>
      </div>
    </section>
  )
}
```

### Recipe 2: Feature Grid with Icons

```typescript
// components/sections/features.tsx
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Zap, Shield, Rocket, Palette } from 'lucide-react'

const features = [
  {
    icon: Zap,
    title: 'Lightning Fast',
    description: 'Optimized components built for performance. No bloat, just speed.',
  },
  {
    icon: Shield,
    title: 'Accessible',
    description: 'WCAG 2.1 AA compliant. Works for everyone, everywhere.',
  },
  {
    icon: Rocket,
    title: 'Developer First',
    description: 'Copy-paste components. Customize. Ship. No lock-in.',
  },
  {
    icon: Palette,
    title: 'Themeable',
    description: 'CSS variable-based theming. Change colors in milliseconds.',
  },
]

export function FeaturesSection() {
  return (
    <section className="py-20 px-4 sm:px-6 lg:px-8 max-w-6xl mx-auto">
      <div className="text-center mb-16">
        <h2 className="text-4xl font-bold mb-4">Why choose us</h2>
        <p className="text-lg text-gray-600 dark:text-gray-400 max-w-2xl mx-auto">
          Everything you need to build modern web applications, without compromise.
        </p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
        {features.map((feature, idx) => {
          const Icon = feature.icon
          return (
            <Card key={idx} className="border border-gray-200 dark:border-gray-700">
              <CardHeader>
                <div className="w-12 h-12 rounded-lg bg-blue-100 dark:bg-blue-900 flex items-center justify-center mb-4">
                  <Icon className="w-6 h-6 text-blue-600 dark:text-blue-400" />
                </div>
                <CardTitle>{feature.title}</CardTitle>
              </CardHeader>
              <CardContent>
                <CardDescription>{feature.description}</CardDescription>
              </CardContent>
            </Card>
          )
        })}
      </div>
    </section>
  )
}
```

### Recipe 3: Pricing Card with Badge

```typescript
// components/sections/pricing-card.tsx
'use client'

import { motion } from 'framer-motion'
import { Button } from '@/components/ui/button'
import { Badge } from '@/components/ui/badge'
import { Check } from 'lucide-react'

interface PricingTierProps {
  name: string
  description: string
  price: number
  period: string
  features: string[]
  popular?: boolean
  cta: string
}

export function PricingTier({
  name,
  description,
  price,
  period,
  features,
  popular = false,
  cta,
}: PricingTierProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      whileInView={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.5 }}
      viewport={{ once: true }}
      className={`relative rounded-2xl border p-8 transition-all ${
        popular
          ? 'border-primary/50 bg-gradient-to-b from-primary/10 to-transparent shadow-2xl scale-105'
          : 'border-gray-200 dark:border-gray-700 bg-white/50 dark:bg-gray-900/50'
      }`}
    >
      {popular && (
        <Badge className="absolute -top-3 left-1/2 transform -translate-x-1/2">
          Most Popular
        </Badge>
      )}

      <h3 className="text-2xl font-bold">{name}</h3>
      <p className="text-sm text-gray-600 dark:text-gray-400 mt-2">{description}</p>

      <div className="mt-6 mb-8">
        <span className="text-5xl font-bold">${price}</span>
        <span className="text-gray-600 dark:text-gray-400">/{period}</span>
      </div>

      <Button className="w-full" variant={popular ? 'default' : 'outline'}>
        {cta}
      </Button>

      <div className="mt-8 space-y-4">
        {features.map((feature, idx) => (
          <div key={idx} className="flex gap-3">
            <Check className="w-5 h-5 text-green-500 flex-shrink-0 mt-0.5" />
            <span className="text-gray-700 dark:text-gray-300">{feature}</span>
          </div>
        ))}
      </div>
    </motion.div>
  )
}

export function PricingSection() {
  const tiers = [
    {
      name: 'Starter',
      description: 'For individuals',
      price: 29,
      period: 'month',
      features: ['Up to 10 projects', 'Basic analytics', 'Community support', '1GB storage'],
      cta: 'Get Started',
    },
    {
      name: 'Professional',
      description: 'For growing teams',
      price: 99,
      period: 'month',
      features: [
        'Unlimited projects',
        'Advanced analytics',
        'Priority support',
        '100GB storage',
        'Team collaboration',
        'Custom domain',
      ],
      popular: true,
      cta: 'Start Free Trial',
    },
    {
      name: 'Enterprise',
      description: 'For large organizations',
      price: 299,
      period: 'month',
      features: [
        'Everything in Pro',
        'Unlimited storage',
        'Dedicated account manager',
        'SLA guarantee',
        'Custom integrations',
        'Advanced security',
      ],
      cta: 'Contact Sales',
    },
  ]

  return (
    <section className="py-20 px-4 sm:px-6 lg:px-8">
      <div className="max-w-7xl mx-auto">
        <div className="text-center mb-16">
          <h2 className="text-4xl font-bold mb-4">Simple Pricing</h2>
          <p className="text-lg text-gray-600 dark:text-gray-400">
            Choose the plan that best fits your needs. Always flexible.
          </p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-8 lg:gap-6">
          {tiers.map(tier => (
            <PricingTier key={tier.name} {...tier} />
          ))}
        </div>
      </div>
    </section>
  )
}
```

### Recipe 4: Testimonial Carousel

```typescript
// components/sections/testimonials.tsx
'use client'

import { useState, useEffect } from 'react'
import { motion, AnimatePresence } from 'framer-motion'
import { ChevronLeft, ChevronRight, Star } from 'lucide-react'
import { Button } from '@/components/ui/button'

interface Testimonial {
  name: string
  role: string
  company: string
  content: string
  rating: number
  avatar: string
}

const testimonials: Testimonial[] = [
  {
    name: 'Sarah Chen',
    role: 'Product Designer',
    company: 'Acme Inc',
    content:
      'The components are beautifully designed and incredibly easy to customize. Saved us weeks of development time.',
    rating: 5,
    avatar: '👩‍💻',
  },
  {
    name: 'James Rodriguez',
    role: 'Lead Developer',
    company: 'TechStart',
    content:
      'Finally found a component library that takes accessibility seriously. WCAG compliant out of the box.',
    rating: 5,
    avatar: '👨‍💼',
  },
  {
    name: 'Emma Watson',
    role: 'Startup Founder',
    company: 'BuildFast',
    content: 'Copy-paste components mean I can focus on business logic instead of CSS. Highly recommended.',
    rating: 5,
    avatar: '👩‍🔬',
  },
]

export function TestimonialCarousel() {
  const [current, setCurrent] = useState(0)

  useEffect(() => {
    const timer = setInterval(() => {
      setCurrent(prev => (prev + 1) % testimonials.length)
    }, 6000)
    return () => clearInterval(timer)
  }, [])

  const next = () => setCurrent((prev) => (prev + 1) % testimonials.length)
  const prev = () => setCurrent((prev) => (prev - 1 + testimonials.length) % testimonials.length)

  return (
    <section className="py-20 px-4 sm:px-6 lg:px-8 bg-gradient-to-br from-slate-50 to-slate-100 dark:from-slate-900 dark:to-slate-800">
      <div className="max-w-4xl mx-auto">
        <h2 className="text-4xl font-bold text-center mb-16">Loved by developers</h2>

        <div className="relative">
          <AnimatePresence mode="wait">
            <motion.div
              key={current}
              initial={{ opacity: 0, x: 100 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: -100 }}
              transition={{ duration: 0.5 }}
              className="rounded-2xl bg-white dark:bg-slate-800 p-8 sm:p-12 shadow-lg"
            >
              <div className="flex gap-1 mb-4">
                {[...Array(testimonials[current].rating)].map((_, i) => (
                  <Star key={i} className="w-5 h-5 fill-yellow-400 text-yellow-400" />
                ))}
              </div>

              <p className="text-lg text-gray-700 dark:text-gray-300 mb-6 leading-relaxed">
                "{testimonials[current].content}"
              </p>

              <div className="flex items-center gap-4">
                <div className="w-12 h-12 rounded-full bg-gradient-to-br from-blue-400 to-purple-400 flex items-center justify-center text-2xl">
                  {testimonials[current].avatar}
                </div>
                <div>
                  <p className="font-bold text-gray-900 dark:text-white">{testimonials[current].name}</p>
                  <p className="text-sm text-gray-600 dark:text-gray-400">
                    {testimonials[current].role} at {testimonials[current].company}
                  </p>
                </div>
              </div>
            </motion.div>
          </AnimatePresence>

          <div className="flex gap-2 justify-center mt-8">
            <Button variant="outline" size="icon" onClick={prev}>
              <ChevronLeft className="w-4 h-4" />
            </Button>
            <div className="flex gap-2 items-center">
              {testimonials.map((_, idx) => (
                <button
                  key={idx}
                  onClick={() => setCurrent(idx)}
                  className={`w-2 h-2 rounded-full transition-all ${
                    idx === current ? 'bg-gray-900 dark:bg-white w-8' : 'bg-gray-300 dark:bg-gray-600'
                  }`}
                />
              ))}
            </div>
            <Button variant="outline" size="icon" onClick={next}>
              <ChevronRight className="w-4 h-4" />
            </Button>
          </div>
        </div>
      </div>
    </section>
  )
}
```

### Recipe 5: Stats Section with Counters

```typescript
// components/sections/stats.tsx
'use client'

import { useEffect, useState } from 'react'
import { motion, useMotionValue, useTransform } from 'framer-motion'

interface StatProps {
  value: number
  label: string
  suffix?: string
}

function AnimatedStat({ value, label, suffix = '' }: StatProps) {
  const [displayValue, setDisplayValue] = useState(0)

  useEffect(() => {
    const duration = 2
    const increment = value / (duration * 100)
    const interval = setInterval(() => {
      setDisplayValue(prev => {
        const next = prev + increment
        return next >= value ? value : next
      })
    }, 10)

    return () => clearInterval(interval)
  }, [value])

  return (
    <div className="text-center">
      <p className="text-5xl sm:text-6xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-purple-400 mb-2">
        {Math.round(displayValue).toLocaleString()}
        {suffix}
      </p>
      <p className="text-gray-600 dark:text-gray-400 font-medium">{label}</p>
    </div>
  )
}

export function StatsSection() {
  const stats = [
    { value: 50000, label: 'Active Developers', suffix: '+' },
    { value: 1000000, label: 'Components Used', suffix: '+' },
    { value: 99.9, label: 'Uptime', suffix: '%' },
    { value: 24, label: 'Hour Support', suffix: '/7' },
  ]

  return (
    <section className="py-20 px-4 sm:px-6 lg:px-8 bg-gradient-to-r from-slate-900 to-slate-800">
      <div className="max-w-6xl mx-auto">
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-12">
          {stats.map((stat, idx) => (
            <motion.div
              key={idx}
              initial={{ opacity: 0, y: 20 }}
              whileInView={{ opacity: 1, y: 0 }}
              transition={{ duration: 0.5, delay: idx * 0.1 }}
              viewport={{ once: true }}
            >
              <AnimatedStat {...stat} />
            </motion.div>
          ))}
        </div>
      </div>
    </section>
  )
}
```

### Recipe 6: CTA Banner with Gradient

```typescript
// components/sections/cta-banner.tsx
'use client'

import { motion } from 'framer-motion'
import { Button } from '@/components/ui/button'
import { ArrowRight } from 'lucide-react'

export function CTABanner() {
  return (
    <section className="relative py-20 px-4 sm:px-6 lg:px-8 overflow-hidden">
      {/* Gradient background */}
      <div className="absolute inset-0 bg-gradient-to-r from-blue-600 via-purple-600 to-pink-600 opacity-90" />

      {/* Animated blobs */}
      <div className="absolute top-0 right-0 w-96 h-96 bg-white/10 rounded-full mix-blend-multiply filter blur-3xl animate-blob opacity-70" />
      <div className="absolute -bottom-20 left-1/3 w-96 h-96 bg-white/10 rounded-full mix-blend-multiply filter blur-3xl animate-blob animation-delay-2000 opacity-70" />

      {/* Content */}
      <div className="relative z-10 max-w-4xl mx-auto text-center">
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          whileInView={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.8 }}
          viewport={{ once: true }}
        >
          <h2 className="text-4xl sm:text-5xl font-bold text-white mb-6">
            Ready to ship faster?
          </h2>

          <p className="text-xl text-white/90 mb-8 leading-relaxed">
            Join thousands of developers building beautiful web applications with our component library.
            Start building today, no credit card required.
          </p>

          <div className="flex gap-4 justify-center flex-wrap">
            <Button size="lg" className="bg-white text-blue-600 hover:bg-gray-100 gap-2">
              Get Started Free
              <ArrowRight className="w-4 h-4" />
            </Button>
            <Button size="lg" variant="outline" className="border-white text-white hover:bg-white/10">
              Schedule Demo
            </Button>
          </div>
        </motion.div>
      </div>
    </section>
  )
}
```

---

## Global Animation Utilities

Add these to your `globals.css` for animation support across all recipes:

```css
@keyframes blob {
  0%, 100% {
    transform: translate(0, 0) scale(1);
  }
  33% {
    transform: translate(30px, -50px) scale(1.1);
  }
  66% {
    transform: translate(-20px, 20px) scale(0.9);
  }
}

.animate-blob {
  animation: blob 7s infinite;
}

.animation-delay-2000 {
  animation-delay: 2s;
}

.animation-delay-4000 {
  animation-delay: 4s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.animate-fade-in {
  animation: fadeIn 0.5s ease-in;
}
```

---

These recipes are production-ready. Copy, paste, customize. Ship.
