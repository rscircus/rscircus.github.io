---
categories:
- Code
date: "2020-02-21T00:00:00Z"
math: true
excerpt_separator: <!-- more -->
sub_title: Setting up LaTeX in Jekyll
tags:
- LaTeX
- accessibility
title: LaTeX in Jekyll
---

Hint: Please follow [this](https://www.simonspavound.com/posts/2020/09/equations-with-katex-in-hugo/) for the Hugo integration as of 2023.

Reference equations like so `\eqref{mylabel}`. And define them like so:

```latex
$$
  \label{diffint}
    \frac{\mathrm{d}}{\mathrm{d} x} \int e^{x}\,dx = e^{x}
$$
```

<!--more-->

To get this to work, we need to add two pieces to `_layouts/post.html`, which is used for displaying posts, where I usually write my articles.

First:

```html
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS_CHTML"></script>
```

which enables MathJax in all posts (rendered with this template).

And to get

- accessibility features (aural rendering, tactile rendering, collapsing)
- equation numbering
- AMSMath
- AMSSymbols
- inlineMath with `$`
- displayMath with `$$`

we configure it as follows:

```html
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
      tex2jax: {
          inlineMath: [
              ['$', '$'],
              ['\\(', '\\)'],
          ],
          processEscapes: true,
      },
      jax: ['input/TeX', 'input/MathML', 'input/AsciiMath', 'output/CommonHTML'],
      extensions: [
          'tex2jax.js',
          'mml2jax.js',
          'asciimath2jax.js',
          'AssistiveMML.js',
          '[Contrib]/a11y/accessibility-menu.js',
      ],
      TeX: {
          extensions: ['AMSmath.js', 'AMSsymbols.js', 'noErrors.js', 'noUndefined.js'],
          equationNumbers: {
              autoNumber: 'AMS',
          },
      },
  })
</script>
```

Here an example in Eq. \eqref{diffint}.

$$
 \frac{\mathrm{d}}{\mathrm{d} x} \int e^{x}\,dx = e^{x}
$$

which actually works with any 'healthy' function $f(x)$ (KaTeX: \\(f(x)\\)). However, \eqref{diffint} is particularly interesting, as $ f(x) = e^x $ (KaTeX: \\(f(x) = e^x\\)) is a unique function because is its own derivative and antiderivative. ☺️

## Sources

- https://www.iangoodfellow.com/blog/jekyll/markdown/tex/2016/11/07/latex-in-markdown.html
- http://flennerhag.com/2017-01-14-latex
