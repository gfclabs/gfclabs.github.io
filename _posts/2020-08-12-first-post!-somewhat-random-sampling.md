---
layout: post
mathjax: true
---

As a first post, I thought I would re-visit some fun work I did in February of 2018. 
For context, I wanted to run a bootstrapped analysis by sampling sub-groups of individuals from a large population, but the bootstrapped samples needed to have a different distribution than that of the population from which I was sampling. 
For example, if I had a normally distributed population of 100,000 individuals with an average age of 50 years and a standard deviation of 10 years, I needed to extract a sample with a mean of 90 years and a standard deviation of 5 years. 
From this initial idea, many questions naturally followed. 
For example, what is the largest population of this size I can extract, is there a unique set of individuals that define this population or does this problem behave like a permutation/combination set, and what technique should I use to compute this efficiently?

In this post, I will outline my proposal for a technique to perform this sampling, provide an example, and finally provide some discussion on the technique. 
Note that the techniques I'm presenting probably will bear strong resemblance to those foundational to the study of importance sampling. 
As I discovered the practice of importance sampling after completing this work, I may not follow exact conventions in my outline of the process. 

#### Nomenclature 
I will refer to the population from which we are sampling as the *source* population, and to the sampled population as the *target* population. 
I use the $s(x)$ to denote the probability density function p.d.f. of the *source* population, and $t(x)$ to denote the p.d.f. of the desired *target* population. 
The likelihood of an observation from the *source* population occurring in the *target* population will be referred to as the *relative propensity*. 
I will refer to the expectation of the log-likelihood of $n$ observations drawn from a distribution as the *expected likelihood*.  

#### Step 1: Compute the Relative Propensity

The first step in this process involves understanding how likely events from the desired *target* distribution $t(x)$ are to appear in the *source* distribution $s(x)$.
To understand this, let's compute what we'll refer to as the relative propensity, think of it like an individual likelihood ratio.
Say, for example, we have a series of values we have observed, $x_1, x_2, ..., x_n$. 
The likelihood of the value $x_i$ in the *source* distribution is just the density at that point, $s(x_i)$. 
The same goes for the likelihood of $x_i$ *target* distribution, it's value is $t(x_i)$.

I define relative propensity, $p(x)$, as the ratio of $s(x)$ and $t(x)$. As such, $p(x_i) = \frac{s(x_i)}{t(x_i)}$.

The underlying principle motivating the definition of relative propensity is that the ratio of the two likelihoods will inform how the *source* distribution should be sampled with respect to the *target* distribution. 
In other words, events that have a *lower* relative propensity should be sampled with greater priority in the queue than those with *higher* relative propensity. 
This is especially true because we are sampling without replacement and need to ensure that events that are likely in the *target* but unlikely in the *source* are prioritized in sampling. 


#### Step 2: Adding Pseudo-randomness + Sampling 

For each of the observations in the sample, $x_1, x_2,...,x_n$ we can compute a relative propensity $p(x_1), p(x_2),...,p(x_i)$.
In order to sample from this bunch, we need to add a certain degree of pseudo-randomness to the observations. 

A commonly used technique to sample from a population and preserve the underlying distribution is to sample using a uniform random variable. 
If we want to generalize this technique to the application at hand, imagine we set $s(x)=t(x)$ which makes $p(x)=1$. 
Let $U$ be the uniformly distributed random variable we use for sampling. 
Sorting by $p(x)*U$, or just $U$ in this case, we can select the top $n$ data points and be fairly confident that they are indeed a random sample from the desired distribution. 


The procedure I propose for when $s(x) \ne t(x)$ is basically the same, but requires one additional step. 
The first step is exactly the same, sort the observations, $x_1, x_2, ..., x_i$, by $p(x)*U$. 
However, unlike the uniform case, we can't just take any $n$ cases and be confident they are indeed from the desired target distribution. 
We need to perform a test to make sure the top $n$ cases we select are indeed from $t(x)$. 
The test you perform is really up to you. 
What I did was increment $n$ by some pre-selected value (depending on the problem) and check the likelihood until we maximized it for the *target* distribution. 
In the example, I will provide this visualization via a likelihood ratio for different sample sizes.  

#### Example

I picked an example where the *source* distribution was skewed and the *target* distribution was not.  
I thought the result would be more visually pleasing if I could eliminate the skewed signal when sampling. 
I also decided to use parametric distributions because they are easy to generate and many of them have closed-form log likelihoods. 

For the *source* distribution I used a [Weibull](https://en.wikipedia.org/wiki/Weibull_distribution) distribution with parameters $ \lambda=65 $ and $ k=10 $. 
I chose the Weibull distribution because I was familiar with it from my work on survival analysis and am fascinated by its elegance and broad applicability.
Also, the Weibull distribution is skewed, which was part of my pre-determined selection criteria. 

For the *target* distribution I used a Normal distribution with $\mu=45$ and $\sigma=5$. 
I chose the Normal distribution because its likelihood function is easy to work with and it has no skewness. 
This way, I could test if the sampling could remove the skewed signal from the Weibull distribution and I could perform my likelihood test with a closed form expression.  

<div style="text-align: center">
    <img src="/assets/images/distribution_compare.png" alt="drawing" width="500"/>
</div>

Next, I computed the weighted sampling variable $p(x)*U$ for all observations in the dataset and sorted. 
Before I sampled, however, I plotted the relative propensity function because I was curious what it looked like for this problem. 
I was amazed at how large some of the values were, so much so that I could only find a small area of the chart that would display well on a linear axis scale. 
 
<div style="text-align: center">
    <img src="/assets/images/relative_propensity.png" alt="drawing" width="500"/>
</div>

The test I chose to do was the maximum likelihood test. To standardize for plotting, I divided by the expected likelihood, and looked for where the chart crossed one. 
I define the likelihood ratio (LR) as

<div style="text-align: center">
    $LR= \frac{-\dfrac{n}{2}\text{log}\sigma^2-\dfrac{n}{2}\text{log}(2\pi)-\dfrac{\sum(x_i-\mu)^2}{2\sigma^2}}{-\dfrac{n}{2}\text{log}\sigma^2-\dfrac{n}{2}\text{log}(2\pi)- \frac{n}{2}},$
</div>

where the observed likelihood is in the numerator and the expected is in the denominator. 

I then plotted the LRs based on the $n$ cases selected from the sorted $p(x)*U$ variable.
These charts behaved as expected. 
As you can see below, when the sample size is small the plot is noisy. 
As the cases increase, the noise decreases and the LR tends towards 1. 
After hovering around one for a little while, it picks back up and heads upward. 

Here is the plot of likelihood ratios for the full population in my test. 
 
<div style="text-align: center">
    <img src="/assets/images/likelihood_ratios.png" alt="drawing" width="480"/>
</div>

Here is a plot zoomed around where it touches 1. 

<div style="text-align: center">
    <img src="/assets/images/likelihood_ratios_zoomed.png" alt="drawing" width="500"/>
</div>

This plot indicates that between 2000 and 3000 is a good place to pick the sample group from. 
I decided to go right down the middle and take the top 2500 as my sample. 
In the plot below, you will see that the result does appear to be Normally distributed.
I still think there is a small amount of the skewed signal remaining, but most has been eliminated. 

<div style="text-align: center">
    <img src="/assets/images/sampling_results.png" alt="drawing" width="500"/>
</div>

#### Discussion 

The example demonstrates that the presented techniques can work under certain circumstances. 
Although there are likely scenarios under which these procedures would break down, they are likely applicable in many areas. 
One particular extension I would like to explore was to use a non-parametric kernel density as the source/target distributions. 
I also think that there is an application for a discrete version, where we used a histogram density for our source/target distributions. 

One area that I find fascinating regarding the approach is how there is still a pseudo-random component to the sampling. 
Although individuals who have a lower relative propensity will appear higher in the queue more often, they are still not guaranteed a top spot. 
This element of randomness is key as it gives us the ability to use this procedure in bootstrapped analyses.
Bootstrapping was indeed the original application for which I explored this technique. 
I imagine that we could over/under-sample complex datasets based on relative propensity. 
I even think that we could probably extend this technique to 3+ dimensions (we just need to compute the N dimensional density of the source/target). 

All in all, I thought that this would be an interesting first blog post. 
I worked hard on this analysis but never had a forum to share my work with others. 
In the coming weeks/months/years I plan on adding similar analyses to this page, along with some less technical work as well. 
Thanks for reading and please reach out with questions, comments, concerns! 
