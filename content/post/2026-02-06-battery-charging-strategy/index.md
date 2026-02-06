---
draft: false
title: "Why I Cap My Batteries at 80%"
description: "The science behind lithium-ion degradation and a practical charging strategy for MacBook and iPhone."
slug: "battery-charging-strategy"
date: 2026-02-06T00:00:00+08:00
categories:
    - Tech
tags:
    - Battery
    - Apple
    - Hardware
---

> Your battery degrades fastest when it's full — not when it's used.

## My Setup

| Device | Charge Limit | Daily Pattern |
|--------|-------------|---------------|
| MacBook Pro M3 Max | 80% | Plugged in most of the time, occasionally unplugged to move around |
| iPhone Air | 80% | Shallow 70–80% cycles, wireless charging at the office + cable at home + power bank on the go |

Both devices almost always have access to power. That's the key prerequisite — this strategy doesn't work if you regularly need a full charge to get through the day.

## Two Ways Batteries Die

### 1. Cycle Aging (from use)

Every charge-discharge cycle forces lithium ions between electrodes, causing the electrode material to expand and contract. Over time this leads to cracking and capacity loss.

Shallow cycles (70→80%) cause dramatically less stress than deep cycles (0→100%).

### 2. Calendar Aging (just from existing)

Even sitting idle, a lithium battery degrades. The electrolyte slowly decomposes at high voltage, thickening the SEI layer and increasing internal resistance. The cathode structure becomes unstable, leaching metal ions.

Two factors control the speed: **temperature** and **state of charge (voltage)**.

## Voltage Is Everything

| Charge | Voltage | Capacity After 1 Year |
|--------|---------|----------------------|
| 100% | ~4.2V | ~92–94% |
| 80% | ~3.9V | ~96–98% |
| 50% | ~3.6V | ~98–99% |

A mere 0.3V difference between 100% and 80% has a massive impact on chemical degradation rate.

## Why "Always Plugged In at 100%" Is the Worst

Here's what happens when your device sits on a charger at full:

1. Charges to 100% (4.2V)
2. Charger stops → battery naturally drops to 99%
3. Charger detects the drop → immediately tops back up
4. Repeat forever

The result: your battery sits at maximum voltage 24/7, maximizing calendar aging, plus accumulating countless 99→100% micro-cycles on top.

## What 80% Actually Solves

- Battery stabilizes around **3.9V** — the chemical degradation sweet spot
- The 70–80% voltage range (3.8–3.9V) is where lithium-ion cells experience **minimum stress**
- Shallow cycling at ~10% depth of discharge causes **near-zero electrode damage**

## "But I Unplug My Mac Sometimes"

A common worry: *"With an 80% limit, every time I unplug to move rooms I'm using battery cycles."*

This doesn't matter:

- A "cycle" = cumulative discharge equal to full capacity
- Losing 1–2% per unplug means you need **50–100 unplugs** to complete one cycle
- The calendar aging you **prevent** by staying at 3.9V instead of 4.2V far outweighs these trivial shallow cycles

## When NOT to Use 80%

- ❌ You're regularly away from power all day → consider 85–90%
- ❌ You're traveling and need maximum runtime → temporarily disable the limit
- ✅ You're mostly at a desk with power → **perfect scenario**
- ✅ You carry a power bank → battery anxiety eliminated

## Long-Term Storage

If you're shelving a device for weeks or months, charge to **50%** and power off. That's the most chemically stable state for lithium-ion.

## Requirements

- macOS Sequoia+ or iOS 18+ (native charge limit settings)
- Device usually near a power source
- A power bank or multiple chargers for peace of mind
