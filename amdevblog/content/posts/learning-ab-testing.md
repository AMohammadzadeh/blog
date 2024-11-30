+++
date = '2024-11-25T18:32:09+03:30'
draft = false
title = 'Exploring the Foundations of A/B Testing'
categories= ['Data Science', 'Experimentation']
tags= ['A/B Testing', 'Experiment Design']
math = true
+++

A/B testing is a method for comparing two versions of a product or feature (Control and Treatment) by randomly assigning users to experience one version and analyzing the resulting data to determine which performs better.

I decided to solidify what I've learned by working through a fictional example.

The company wants to decide whether it should offer its users a prize for their first purchase of greater than $1,000. Before running the experiment, I should define my Overall Evaluation Criterion (OEC). The OEC is a quantitative measure of the experiment's objective. A good OEC has the following properties:

1. **Sensitivity**: It is sensitive enough to show the differences.
2. **Predictive of Long-Term Goals**: I cannot run the test for a long time to know if it has long-lasting effects. So, I need to be sure that changes in the OEC can be attributed to long-term goals.
3. **Measurable in Short Time**: It is measurable in a short time—I shouldn't have to wait too long to see a change in the OEC.

Let's say our goal is to increase the _average first purchase amount of users_.

The randomization unit will be at the user level, and we will identify each user by their user ID, as only signed-in users will be able to see the prize. Furthermore, the randomization unit (user) is the same as the analysis unit (average first purchase amount of _users_), which makes analysis easier.

Now, it is time to calculate how large our experiment should be. Doing that requires understanding _statistical power_, _minimum detectable effect (MDE)_, and the _p-value_. Statistical power is the probability of detecting a meaningful difference between the variants when there really is one, and MDE is the smallest change we want to detect. The p-value is defined as the probability of observing the obtained result, or more extreme results, under the assumption that the null hypothesis is true. The authors of the paper argue that this concept is often misinterpreted despite its importance.

Let's assume that our current average first purchase amount is 500 dollars with a standard deviation of 1,000 dollars, and a 20% change in the purchase amount is of interest. Therefore, our MDE is \[ 20\% \times \$500 = \$100 \], and we want to detect it 80% of the time, which sets our statistical power at 80%. Choosing a p-value threshold of 0.05, the sample size for each equally sized variant can be roughly calculated by the following formula:

$$
n = \frac{16\sigma^2}{\delta^2}
$$

Where \[ \sigma^2 \] is the variance of the metric of interest and \[ \delta \] is the MDE[^1]. The calculation shows that we need \[ \frac{16 \times (1,000)^2}{(100)^2} = 1,600 \] users per variant. Note that since my metric of interest is continuous here, if the metric were binary (e.g., conversion rate), you could use \[ p \times (1 - p) \], where \[ p \] is the baseline conversion rate (e.g., 20%), to calculate \[ \sigma^2 \].

An important consideration here is the normality assumption of sample means, which holds due to the _Central Limit Theorem_. The authors of the book _Trustworthy Online Controlled Experiments_ in Chapter 17 explain this concept and mention that one can calculate the minimum sample size needed for the normality assumption to be valid using the formula \[ 355 s^2 \], where \[ s \] is the skewness coefficient of the sample distribution of the metric.

The final experiment design decision I need to address is how long we should run the test. Based on my previous calculations, we need to run the test until each variant attracts at least 1,600 users. It is very important that we keep the experiment running even if we achieve a significant result (i.e., p-value < 0.05) with fewer than 1,600 users. Also, since our weekday and weekend users may have different behavior, it is recommended that we run the test for at least a full week.

Now we can summarize our experiment design:

- **Hypothesis**: Offering a prize for first purchases of greater than $1,000 will increase the average first purchase amount.
- **Randomization Unit**: User
- **Statistical Power**: 80%
- **p-value Threshold**: 5%
- **Variant Size**: 1,600 users for each variant
- **Experiment Run Time**: At least a full week

This example by no means covers all aspects and nuances of A/B testing, but it is a practice for designing an experiment while avoiding most common pitfalls.

## References

1. Kohavi, R., Deng, A., & Vermeer, L. (2022, June). _A/B Testing Intuition Busters_. In Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining.
2. Larsen, N., Stallrich, J., Sengupta, S., Deng, A., Kohavi, R., & Stevens, N. T. (2024). _Statistical Challenges in Online Controlled Experiments: A Review of A/B Testing Methodology_. The American Statistician, 78(2), 135-149.
3. Kohavi, R., Tang, D., & Xu, Y. (2020). _Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing_. Cambridge University Press.

[^1]: There are various online tools you can use for calculating sample size.
