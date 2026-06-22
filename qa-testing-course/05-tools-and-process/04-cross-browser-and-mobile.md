# Cross-Browser and Mobile Testing

Your users don't all use the same setup. Some are on Chrome on a big monitor, some on
Safari on an iPhone, some on an old Android with a cracked screen and slow internet.
Software that works perfectly for *you* can be broken for *them*. **Cross-browser**
and **mobile/responsive** testing make sure the app works for *everyone* — and it's
very doable by hand.

## What You'll Learn

- Why the same app behaves differently across browsers and devices
- What to check in cross-browser testing
- What responsive and mobile testing involve
- Practical ways to test without a drawer full of devices

## Why Things Break Across Environments

Different browsers and devices interpret the same web page slightly differently — and
screens come in wildly different sizes. So you get bugs like:

- A button that looks fine in Chrome but overlaps text in Safari.
- A menu that works on desktop but is unreachable on a phone.
- A form that's perfect on a big screen but cut off on a small one.
- A feature that's fast on Wi-Fi but times out on mobile data.

These are **compatibility** issues (a non-functional type from
[Module 2](../02-types-of-testing/02-non-functional-testing.md)). The app's *logic*
is fine; it's the *environment* that exposes the problem.

## Cross-Browser Testing

**Goal:** the app works correctly across the browsers your users actually use.

The main browsers to consider:

- **Chrome** (by far the most common)
- **Safari** (all iPhones/iPads, Macs — and it behaves differently!)
- **Firefox**
- **Edge**

What to check in each:

- **Layout** — does everything line up, with nothing overlapping or cut off?
- **Functionality** — do all buttons, forms, and features work?
- **Appearance** — fonts, colors, images render correctly?
- **Interactions** — dropdowns, date pickers, pop-ups behave?

You don't test *every* browser equally — you focus on the ones your users have (check
analytics if available). Safari is a frequent troublemaker, so never skip it.

## Responsive and Mobile Testing

**Responsive design** means the layout *adapts* to the screen size. Your job is to
confirm it adapts *correctly* at different sizes.

```text
   Desktop (wide)        Tablet (medium)      Phone (narrow)
   ┌──────────────┐      ┌──────────┐         ┌─────┐
   │ ☰  logo  menu│      │ ☰  logo  │         │ ☰   │
   │              │      │          │         │logo │
   │  3 columns   │      │ 2 columns│         │  1  │
   │              │      │          │         │ col │
   └──────────────┘      └──────────┘         └─────┘
   Same page, layout reflows to fit each screen.
```

Things specific to **mobile** that desktop testing misses:

- **Touch** instead of mouse — are tap targets big enough? No tiny buttons.
- **Orientation** — does it work in both portrait and landscape?
- **On-screen keyboard** — does it cover the field you're typing in?
- **Gestures** — swiping, pinch-to-zoom where expected.
- **Slow networks** — does it cope with mobile data and spotty signal?
- **Notches and safe areas** — content not hidden behind the camera cutout.

## How to Test Without 50 Devices

You don't need a closet full of phones. Practical options:

1. **Browser DevTools device mode.** Chrome/Firefox have a built-in "responsive
   mode" (press F12 → toggle device toolbar) that simulates many screen sizes and
   devices. Great first pass — *free and instant*.
2. **Resize the browser window.** Drag it narrow and watch the layout reflow. Crude
   but catches a lot.
3. **Real devices.** Test on at least one real phone — emulators don't perfectly
   match real touch, performance, or Safari quirks.
4. **Cloud device farms.** Services like BrowserStack or Sauce Labs let you test on
   hundreds of real browser/device combos in the cloud. Common in real jobs.

A sensible habit: **rough pass in DevTools → confirm on a real phone → use a cloud
farm for the long tail** of browsers/devices you can't physically own.

## A Realistic Cross-Browser/Device Checklist

For an important page, run through:

- [ ] Chrome desktop — layout + core functions
- [ ] Safari desktop (or iPhone Safari) — same
- [ ] Firefox — same
- [ ] Phone portrait — layout reflows, tap targets work, no cut-off content
- [ ] Phone landscape — still usable
- [ ] Slow network — loads in reasonable time, no broken states

## Exercises

1. Why can an app work perfectly in Chrome but be broken in Safari?
2. List four things that are specific to *mobile* testing that you wouldn't check on
   desktop.
3. Open any website in your browser's DevTools device mode and test it at phone,
   tablet, and desktop widths. Note any layout problems.
4. Why test on at least one *real* phone instead of relying only on an emulator?

Next: [Intro to Test Automation](05-intro-to-automation.md)
