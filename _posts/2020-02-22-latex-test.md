---
layout: post
title: 'LaTeX in Jekyll'
sub_title: 'Setting up LaTeX in Jekyll'
excerpt_separator: <!-- more -->
categories:
    - Code
tags:
    - LaTeX
    - accessibility
---

I would like to reference equations like so `\eqref{mylabel}`.

<!-- more -->

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
\begin {equation} \label{diffint}
\frac{\mathrm{d}}{\mathrm{d} x} \int_{a}^{x} f(s)ds = f(x)
\end{equation}
$$

😍

## Sources

- https://www.iangoodfellow.com/blog/jekyll/markdown/tex/2016/11/07/latex-in-markdown.html
- http://flennerhag.com/2017-01-14-latex
