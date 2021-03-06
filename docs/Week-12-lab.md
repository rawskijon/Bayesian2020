Week 12 Lab
========================================================

Before we even get to mark recapture models, we need to introduce a couple of tricks in JAGS that can be used to handle non-standard distributions.

The 'zeros' trick
-------------------

The ‘zeroes trick’ is JAGS goes as follows. Let’s say we want to model

$$
Y \sim g(\theta)
$$

but $G()$ is not a distribution that is built into JAGS. We can achieve the same effect (i.e. have the same likelihood enter the model) if we instead say that we observed a value of $Y=0$ from the following model

$$
Y \sim Pois(\lambda)
$$

where 

$$
\lambda = -\mbox{log}(g(y,\theta))
$$

and $g(y,\theta)$ is the likelihood associated with distribution $G$. In other words, the probability of getting a 0 from $\mbox{Pois}(-\mbox{log}(g(y,\theta))$ is just $g(y,\theta)$, which is the likelihood we wanted in the first place. 

(Reminder, the pmf of a Poisson is $\frac{\lambda^{x}}{x!}e^{-\lambda}$ so the probability of getting $x=0$ is $e^{-\lambda}$.)

In practice, we add a constant $C$ to get $\lambda = -\mbox{log}(g(y,\theta))+C$ to ensure that $\lambda > 0$.)

Let’s take for example the Cauchy distribution

$$
f(y|\theta) = \frac{1}{\pi}\frac{1}{1+(y-\theta)^{2}}
$$
The Cauchy is an evil distribution, as you all know. Nevertheless, we can use the zeros trick to sample from it as follows:


```r
Zeros[i]<-0
lambda[i]<- -log(1/(1+pow(y[i]-theta,2)))
Zeros[i]~dpois(lambda[i])
```

Notice that I just need the portion of the pdf that involves the data and the parameters because all that matters is that I get something proportional to the correct likelihood, and so I have dropped the constants involved in $f(y|\theta)$.

The 'ones' trick
-------------------

The ‘ones trick’ is similar, but it uses a dummy variable of 1 drawn from a Binomial distribution:

$$
Y \sim \mbox{Bern}(p) \\
p = g(y,\theta)
$$

The code for the Cauchy looks like


```r
Ones[i]<-1
p[i]<- 1/(1+pow(y[i]-theta,2))
Ones[i]~dbern(p[i])
```

##Initial values

Sometimes, the hardest part about getting JAGS to run successfully is supplying it with good initial values. For mark-recapture models I suggest trying NAs up to the first capture, 1 through the period of known survival (from marking to last resighting) and 0 for the period where its fate is unknown (after the last resighting).

##First, a warm up

Let’s look through some code for occupancy modeling that will serve as a warm up to the Lab and the Problem Set. It has been posted on Bb under “code for Week 12 Lab warm up.R”. This code simulates the detection of an amphibian (usually detected through calling) at 100 sites assuming 30 surveys each breeding season, and detection probabilities that vary as a function of humidity. (This is a bit of a contrived example. We assume 30 replicate surveys at each site, assuming a closed population and assuming that each site has fixed humidity. Nevertheless, these assumptions simplify the data simulation and are easily relaxed.) 

(Note that this warm up exercise involves occupancy of an animal species at different sites, whereas the lab itself involves mark-recapture of individual animals over several years. Occupancy and mark-recapture are related problems, but the actual model code will be different.)

##Fitting mark-recapture models

Mark-recapture models are a huge field of modeling, and so we will only just barely scratch the surface on what can be done. Nevertheless, they are a common application of Bayesian models, and they will give us some practice writing Bayesian models in a wider variety of applications.

We are going to start with a fairly straightforward dataset on the survival of the European Dipper, which was described in the paper by McCarthy and Masters (2005) that you read for class this week. I have posted the survival data on Bb; copy and paste from “dipper_data.txt”. (While the code used by McCarthy and Masters is online as part of the supplement, no peaking! You’ll learn more working through it from the beginning.)

Write a mark-recapture model to estimate survival (constant over time) and resighting rate (also constant over time) for the Dipper. We’ll start with an uninformative prior, and we have time can go back and discuss how McCarthy and Masters generated a more informative prior if we have time.

**Exercise #1**: Use the Method #1 (Brute force) approach described in lecture on Monday. This is the most straightforward and the easiest to code.

**Exercise #2**: Use the Method #2 (Modelling the entire capture history) approach described on Monday. You’ll have to think carefully how to represent the likelihood for the entire recapture history. (Hint: You may need the ‘ones’ or ‘zeroes’ trick.)

