---
layout: post
title: 'What is information entropy?'
sub_title: 'A simple explanation for information entropy'
excerpt_separator: <!-- more -->
categories:
  - Math
tags:
  - Information Entropy
  - Theoretical Informatics
  - Shannon Entropy
---

Recently I was thinking about a simple explanation for information entropy. So here you have the fruits of my labor.

<!-- more -->

Information entropy is best explained with *information transmission* in mind. Say one wants to transport as little bits as possible from a sender to a recipient to inform the recipient about a certain state the sender wants to communicate.

Traditionally we communicate in bits in technical fields, that is, two-states in one *something*. One could also communicate using more than two states, but as the world developed in a way to favor two-state bits, we stick to this:

$$
1 \; \mathrm{bit} =
  \begin{cases}
    \quad 1 \\
    \quad 0
  \end{cases} \quad \text{or} \quad 1 \; bit \in \{0, 1\}
$$

Now let's imagine the sender is a kid who wants to inform his friend using a tin can telephone about the state of his parents' presence (them being in bed or not, so the friend (recpient) can come over).

What is the overall goal of transporting this information? Basically the sender wants to reduce the recpients uncertainty about when he come over without or with little risk of being detected. So, basically the uncertainty reduction is the inverse of an events/state's probability.

We can easily see, that we only need to transport one bit here:

$$
1 \; \mathrm{bit} =
  \begin{cases}
    \quad \text{parents in bed} \\
    \quad \text{parents awake}
  \end{cases}
$$

So there are two possible states here:

$$
2 = \text{possible-bit-states}^{no-of-bits} = 2^1
$$

The uncertainty of the friend of being detected or knowing about the kid's house state is reduced by a factor of two, as there are two states, if we communicate one lonely bit. Remember the uncertainty reduction being the inverse of the probability?
With the communication of one bit, the kid can inform his friend about the state of the house and reduce his uncertainty by two, which - assuming the friend knows about the two possible states of the house - would bring him down to certainty.

Now, let's add the state of the kid's sister, too:

$$
? \; \mathrm{bits} =
  \begin{cases}
    \quad \text{parents in bed} \\
    \quad \text{parents awake} \\
    \quad \text{sister in bed} \\
    \quad \text{sister awake}
  \end{cases}
$$

How would we calculate the lowest number of bits to communicate this state? We have four states which are a combination of the list above and this is also our uncertainty reduction factor.

$$
4 = 2^n
$$

$$
n = log_2(4)
$$

$$
n = 2
$$

We would need two bits to transport the state completely.

By transporting these bits we can reduce the senders uncertainty. [Shannon](https://en.wikipedia.org/wiki/Claude_Shannon) defined this uncertainty reduction as the uncertainty being halved per bit transported, completely independent of what the uncertainty was in the first place. And we just had the same line of thought while thinking this through.

This is best understood with two equally-likely states being available only. As in the example above:

$$
\mathrm{state} =
  \begin{cases}
    \quad \text{parents in bed} \\
    \quad \text{parents awake}
  \end{cases}
$$

What if the probability of each state is now skewed because the mother usually goes to bed way later than the sender's father?

$$
\mathrm{state} =
  \begin{cases}
    \quad \text{20 % probability of parents being in bed} \\
    \quad \text{80 % probability of parents being awake}
  \end{cases}
$$

And the recpient knows about this bias.

How much bits do we transport now to inform the recpient about the state of the house?

$$
n = log_2(p^{-1})
$$

With $p$ being the probabilty of a state. Let's start with the parents being in bed $p_{bed} = 0.2$.

$$
n_{bed} = log_2(\frac{5}{1}) \approxeq 2.32
$$

and the parents being awake $p_{awake} = 0.8$:

$$
n_{awake} = log_2(\frac{5}{4}) \approxeq 0.32
$$

Naturally one sees, that the uncertainty reduction is way higher with the information of the parents being in bed communicated compared to the information being awake being communicated as the parents being awake was quite likely in the first place.

Therefore, the float-point bits here can be interpreted as information gain on the recipients' side instead of a real bit, which would be impossible to communicate.

So, how much information (gain) can the kid transport on average to his/her friend if they want to meet every evening?

$$
information_{avg} = 0.2 \times 2.32 + 0.8 \times 0.32 = 0.72 \; \text{bit}
$$

What we just computed is information entropy. ;-)

Let's formalize this a bit:

$$
Entropy(\bold{p}) = H(\bold{p}) = - \sum_i p_i log_2(p_i)
$$

Here we used that $log_2(p^{-1}) = - log_2(p)$. Entropy is the information gain the recipients gets when he learns about the kid's house state each day. Or more generally the average amount of information that one gets when drawing a random sample from a probability distrubtion $\bold{p}$.

*Hint:* The information entropy is sometimes written more generally with $log(x)$ and not $log_2(x)$. Why can we do this? Because:

$$
log_2(x) = \frac{log(x)}{log(2)}
$$

as we usually are interested in entropy differences and this just divides the right side of the equation by a constant and it looks the same as the mathematical expression for ["physical" entropy](https://en.wikipedia.org/wiki/Entropy) in statistical mechanics.