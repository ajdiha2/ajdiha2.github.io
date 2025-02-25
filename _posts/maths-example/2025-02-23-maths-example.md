---
layout: post
title:  Maths Example with Mathjax
date:   2025-02-23 06:45:44 +0300
categories: jekyll update
usemathjax: true
---

Below is an example of maths using mathjax. 

Any page needing maths should start with the frontmatter:
{% highlight markdown %}
usemathjax: true
{% endhighlight %}

$$ 
\begin{align*}
y = y(x,t) &= A e^{i\theta} \\
&= A (\cos \theta + i \sin \theta) \\
&= A (\cos(kx - \omega t) + i \sin(kx - \omega t)) \\
&= A\cos(kx - \omega t) + i A\sin(kx - \omega t)  \\
&= A\cos \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big) + i A\sin \Big(\frac{2\pi}{\lambda}x - \frac{2\pi v}{\lambda} t \Big)  \\
&= A\cos \frac{2\pi}{\lambda} (x - v t) + i A\sin \frac{2\pi}{\lambda} (x - v t)
\end{align*}
$$

Inline maths can be written with the `\\(` and `\\)` characters, producing inline maths
such as \\(\delta(t) \xrightarrow{\mathscr{F}} 1\\).

The above is accomplished with thanks to [Alan Duan](https://alanduan.me/random/mathjax/) and [Zichen Vincent Zhang](https://webdocs.cs.ualberta.ca/~zichen2/blog/coding/setup/2019/02/17/how-to-add-mathjax-support-to-jekyll.html.).



<div dir="rtl">
<h2>تعریف</h2>
مشتق: شیب خط مماس بر تابع f در نقطه a
</div>

$$
\large {
\lim_{h \to 0} {f(a+h) - f(a) \over h} 
}
$$

$$
\large {
\lim_{x \to a} {f(x) - f(a) \over x - a} 
}
$$

<div dir="rtl">
مثال: رابطه‌ای برای مشتق تابع \(y = x^2 + 4\) بیابید
</div>

$$
\begin{align*}
\lim_{x \to a} {f(x)-f(a) \over x-a} =
\lim_{x \to a} {(x^2+4)-(a^2+4) \over x-a} = 
\lim_{x \to a} {x^2-a^2 \over x-a} = \\
\lim_{x \to a} {(x-a)(x+a) \over x-a} =
\lim_{x \to a} (x+a) = 2a
\end{align*}
$$

