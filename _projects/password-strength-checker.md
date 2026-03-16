---
title: "Password Strength Checker (JS)"
description: Minimal dark-themed password strength checker with live meter + checklist.
date: 2026-03-02
permalink: /projects/password-strength-checker/
layout: post
---

## Overview
I've built a beginner-friendly **HTML/CSS/JavaScript** password strength checker that updates in real time while you type.

_Note: Since I CAN code HTML and CSS but not Javascript in depth I used different sources to build a unique js-file which I checked with an AI tool._

My goal was to keep the code explainable while still making the UI look **dope** (dark & minimal).

## Features
- Live strength rating (Weak / Okay / Strong / Very strong)
- Checklist rules (length, lower/upper/number/symbol)
- Show/Hide toggle
- Custom dark theme

## Screenshots

![Empty state](/assets/img/projects/password-strength-checker/01.png)
_Empty state_

![Weak password example](/assets/img/projects/password-strength-checker/02.png)
_Weak password example_

![Strong password example](/assets/img/projects/password-strength-checker/03.png)
_Strong password example_

## How it works
1. On every keystroke (`input` event), the password is evaluated.
2. Regex checks determine which rules are met (lower/upper/number/symbol).
3. A small score is calculated and mapped to strength label and meter percentage.
4. UI updates: checklist + meter + strength text.

## Snippets from the key code
#### Password evaluation (rules + score)
{% highlight js %}
function evaluatePassword(pw) {
  const checks = {
    length: pw.length >= 12,
    lower: /[a-z]/.test(pw),
    upper: /[A-Z]/.test(pw),
    number: /[0-9]/.test(pw),
    symbol: /[^A-Za-z0-9]/.test(pw),
  };

  // Score: 0..5 (one point per check)
  let score = 0;
  for (const key in checks) {
    if (checks[key]) score++;
  }

  // Convert score -> label + bar %
  let label = "—";
  let percent = 0;

  if (pw.length === 0) {
    label = "—";
    percent = 0;
  } else if (score <= 2) {
    label = "Weak";
    percent = 25;
  } else if (score === 3) {
    label = "Okay";
    percent = 55;
  } else if (score === 4) {
    label = "Strong";
    percent = 80;
  } else {
    label = "Very strong";
    percent = 100;
  }

  return { checks, score, label, percent };
}
{% endhighlight %}

## What I learned

- Implementing rule-based password evaluation using regex patterns
- Recognizing the limitations of client-side validation from a security perspective
- How small design decisions (e.g., real-time feedback) influence user behavior
- Why length often matters more than complexity rules in practical security contexts

## Links

🔗 [View Source Code on Codeberg and try it yourself!](https://codeberg.org/securityfox/password-strength-checker)
