Week 3 Lecture
========================================================

Papers to read this week:

* [Kuhnert et al. 2010](https://github.com/hlynch/Bayesian2020/tree/master/_data/KuhnertEtAl2010.pdf)
* [Lambert et al. 2005](https://github.com/hlynch/Bayesian2020/tree/master/_data/LambertEtAl2005.pdf): More advanced material
* [Senn 2007](https://github.com/hlynch/Bayesian2020/tree/master/_data/Senn2007.pdf): More advanced material
* [Lambert et al. 2008](https://github.com/hlynch/Bayesian2020/tree/master/_data/LambertEtAl2008.pdf): A response to Senn 2007


The use of prior information is one of the major distinguishing features of Bayesian analysis, and also the issue that generates the most controversy about Bayesian methods.

**How do we think about the relationship between the prior distribution, the data, and the posterior?** 

We took an aside last week to discuss the ways in which Bayesian statistics is used for criminal justice proceedings, so here we will continue using criminal justice (a system we have some intuition about) to think about the role of priors. Our criminal justice system assumes all defendants are “innocent until proven guilty”. This is our prior expectation. The evidence has to overwhelmingly suggest guilt for a jury to reach a guilty verdict. (“Innocent until proven guilty” refers to the prior distribution, “proof beyond a reasonable doubt” refers to what the posterior distribution would have to look like to determine that a defendant is actually guilty.) 

**Question**: What does the prior distribution actually look like in a criminal case? (Hint: Can’t be a delta function at “innocent” – why not?)

How do we obtain priors?
------------------------

The vast majority of ecological analyses use non-informative priors bounded only by biological feasibility. (In other words, you might constrain lifespan to the maximum known lifespan for that species, but use a uniform distribution over that range as the prior.) In these cases, Bayesian statistics are being employed because they are simply more flexible than frequentist methods; where both analyses are possible, they will yield similar results.

Technical Note: It is possible to use a prior distribution that is “improper”, i.e. doesn’t integrate to 1, and still obtain a proper posterior distribution. However, JAGS generally forces the use of proper prior distributions.

Conjugacy
-------------

When the prior and posterior distributions have the same distributional form, the prior is said to be “conjugate” to the distribution for the data. For example, if you have data that are binomially distributed, and you use a beta distribution to represent your prior knowledge of the binomial probability $p$, then you will get a posterior distribution for $p$ that is also described by a beta distribution. We say in this case that the beta and binomial form a “conjugate pair”.

NB: Note that the beta distribution is a prior distribution on the continuous (but bounded) parameter $p$, and the binomial distribution is a discrete distribution representing the discrete outcome of the binomial process. In other words, the distribution used to model the data may be discrete, and the distribution to describe the parameter may be continuous. There is no conflict combining these two different types of distribution in this context.

Why do we care? In the old days, before MCMC methods were made feasible by fast computers, the only way to do Bayesian analyses was to exploit the algebraic simplicity of conjugacy, because these were the only cases in which you could calculate, using pencil and paper, the posterior distribution. Now with the advent of MCMC and fast computers, we don’t *need* to limit ourselves to prior distributions that are conjugate to the model for the data. Even now, however, WinBUGS and JAGS will run much faster if you do use a conjugate prior, and due to the historical legacy, conjugate pairs are often the default choice even if they are no longer required. (In fact, you should use a prior distribution that most honestly reflects your prior knowledge, whatever form that might be, unless computation efficiency because a major concern for your analysis.)

Sometimes, however, informative priors are used, either because they reflect genuine prior knowledge of the system that should be reflected in the analysis, or because there is so little data to work with that you need some kind of informative prior to get reasonable, useful posterior distributions. 

Suppose we are told that we have a fair coin, and it has come up heads $y=10$ times. We do not know how many times the coin was flipped.

The likelihood for the data ($y=10$) can be written as:

$$
p(y|N) = \mbox{Binomial}(0.5,N) = \frac{N!}{(N-y)!y!}0.5^{N-y}0.5^{y} \propto \frac{N!}{(N-y)!y!}0.5^{N}
$$

We need a prior for N, and since N is discrete we need a discrete distribution for this prior. Jeffrey’s prior in this case would be $p(N) \propto 1/N$. (See text box for more information about Jeffrey's prior; we will not go into much detail and for the example here, you just need to know that the prior is $\propto 1/N$.)

The posterior distribution is then given by:

$$
p(N) \propto \frac{N!}{(N-y)!y!}0.5^{N} \times \frac{1}{N} = \frac{(N-1)!}{(N-y)!}0.5^{N}
$$
This is in fact (modulo constants) the pdf of the negative binomial (you can convince yourself this is true at home), which is what you would expect **if** the experimental design had been “keep flipping until you get y heads”. That is precisely the scenario that generates a negative binomial distribution. **But**, in this case, we do not need to assume anything about the manner in which the data were collected, and in fact we do not know whether the 10th head was the last coin flip which ended the “experiment”. In could have been 10 heads followed by 10 tails – in this case the experimental design does not influence the posterior distribution for N. 

I assigned a re-reading of Berger and Berry (1988) for lecture today because this example gets to the heart of that earlier discussion from Biometry. Remember from Berger and Berry that a frequentist approach to this same problem would require some \emph{a priori} assumption about the nature of the experiment. This comes about because frequentist p values depend not only on the likelihood of the outcome, but the likelihood of all outcomes *more extreme* than the one obtained. This dependence on *more extreme* outcomes that were not observed require that you know (or assume) something about the range of outcomes possible under the experimental design. Bayesian statistics do not hinge on unobserved outcomes, which is why it does not require any assumptions about the manner in which the data were obtained.

Sensitivity analysis
------------------------

At the end of every Bayesian analysis, you should do a sensitivity analysis to see how sensitive your posterior distributions are to priors. In some cases, some parameters may be highly sensitive to the choice of prior, but other parameters of more interest are not. This would be perfectly fine if you are not going to be making any biological inference on the parameters that are highly sensitive to the priors. They may be tangential to the parameters you are really focused on, which themselves may be fairly robust to various prior assumptions. However, you may have cases where the prior makes a big difference. While many data analysts fear this possibility, this is in fact, a confirmation that a Bayesian approach is worthwhile in the first place. In these cases, you simply need to justify the choice of prior and be transparent about the range of outcomes that might have been obtained with different prior assumptions.

Our first exercise here will be to emphasize a caution noted in the handout from the book by Lunn et al.: that a uniform distribution is not always a “vague” or uninformative prior. You have to be very cautious when dealing with transformation of parameters. If you want a vague prior for a parameter, make sure the prior is actually what you think it is. In particular, transformations of a uniform are not uniform. This often comes up when putting a vague prior on the variance, since JAGS deals only with the precision (the inverse-variance). A uniform distribution for the precision is not what you actually want – since does not reflect the uncertainty on the scale intended (i.e. on the variance, or possibly on the standard deviation).

**EXERCISE #1**: Convince yourself of this by looking at the distribution for $\theta^2$ if $\theta \sim \mbox{Unif}(0,1)$. Lunn et al. tells us that this new distribution is actually a Beta(0.5,1) – do you agree?

Note that while the above example seems contrived, this exact same issue can come up quite easily in ecology. Take the following scenario: You want to model the state of a bird, and there are three possible states (1) not present, (2) present but not breeding, (3) present and breeding. We can think of this as two independent processes:

$$
\mbox{Present} \sim Bern(\theta)
$$

and,

$$
\mbox{Breeding|Present} \sim Bern(\pi)
$$
If you didn’t know anything about this bird, you would be tempted to use the following “uninformative” priors:

$$
\theta \sim \mbox{Unif}(0,1) \\
\pi \sim \mbox{Unif}(0,1)
$$

However, when we put these two processes together, the prior distribution for these three states is **not** uninformative. 

Next, we are going to explore the various methods one could employ to generate a vague prior for the Binomial parameter p.  This exercise is based on the discussion in Lunn et al. Section 5.2.5 but I want to convince ourselves of this in R since R is easier to work with and we can focus on the core ideas involved.

Lunn et al. presents us with the following situation. You are doing a logistic regression analysis to understand the probability $\theta$ of a binomial event occurring, and you want to put a flat prior on the parameter $\theta$. But here we have several options, because we can use a vague prior for the probability p or we could use a vague prior for the logit($\theta$). In addition, there are several vague priors we might choose. Lunn’s notation is as follows (the “whatever” could be any linear function, its not relevant here):

$$
Y \sim \mbox{Binomial}(\theta) \\
\phi \sim \mbox{logit}(\theta) = \mbox{whatever}
$$
Lunn et al. lays out the options as follows (I have crossed out #3 since we won't get into Jeffrey's priors but have left it so that the numbering is consistent with Lunn's figure):

(1) Use a uniform on $\theta$, let $\phi$ be determined accordingly

(2) Use a uniform on $\phi$, let $\theta$ be determined accordingly

~~(3) Use a Jeffrey’s prior on $\theta$, let $\phi$ be determined accordingly~~

(4) Use a N(mean=0,precision=0.5)=N(mean=0,var=2) prior on $\phi$, let $\theta$ be determined accordingly

(5) Use a N(mean=0,precision=0.368)=N(mean=0,var=2.71) prior on $\phi$, let $\theta$ be determined accordingly

**EXERCISE #2**: Plot these alternatives in R to recreate Lunn et al. Figure 5.1. What are the pros and cons of using each of these options. Is there any practical benefit to Option #1 over #5 (or vice versa)?

Expert elicitation
---------------------

One method of obtaining prior distributions is through expert elicitation. The Delphi method is one way of eliciting expert judgment.

Delphi method:
1) Elicit information from each expert independently

2) Collect results and share with the group

3) Experts reconsider their responses in light of the responses of others

4) Elicitation and feedback cycle continues

We will walk through an example of this in lab.

For more information about this week's topic
------------------------

We do not go into detail on Jeffrey's prior in this course because we have limited time and I don't see Jeffrey's priors used very often in practical ecological analyses. However, here are some references that may be useful if you would like more information.

On Jeffrey's Priors: Notes from lectures at [Berkeley](https://people.eecs.berkeley.edu/~jordan/courses/260-spring10/lectures/lecture7.pdf) and [Duke](https://www2.stat.duke.edu/courses/Fall11/sta114/jeffreys.pdf).

Other papers:

* [Daniels 1999](https://github.com/hlynch/Bayesian2020/tree/master/_data/Daniels1999.pdf)

A chapter from Lunn et al. 2012 $\textit{A BUGS Book: A Practical Introduction to Bayesian Analysis}$

* [Lunn et al. Chapter 5](https://github.com/hlynch/Bayesian2020/tree/master/_data/LunnChapter5.pdf)
