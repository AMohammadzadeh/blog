+++
date = '2024-11-19T21:52:37+03:30'
draft = false
title = 'Be Careful What You Control For'
tags= ['Casual Inference']
math = true
+++

What is the effect of online learning on students' performance? Does attending a top-tier university significantly affect future earnings?
Would a customer still purchase if we hadn’t offered a discount?

We often encounter questions like these in our work or research. They aim to uncover the causal effect of a treatment (e.g., online learning, attending a prestigious university, offering a discount) on a particular outcome (e.g., students' performance, future earnings, customer purchases). Intuitively, we recognize that answering such questions is challenging—it requires us to observe the same individual in two different scenarios at the same time. For instance, we’d need to know whether the same customer would have purchased if they had not received the discount. This concept is referred to as **potential outcomes**.

- $Y_0$: The outcome an individual would experience without the treatment.
- $Y_1$: The outcome the same individual would experience under the treatment.

Now, let’s assume we can randomly assign discounts to customers. In such a scenario, the following regression equation suffices:

$$Purchase_i = \beta_0 + \beta_1 Discount_i + u_i$$

It can be shown that the estimate of $\beta_1$ ($\hat{\beta}\_1$) represents the effect of the discount on purchases.

Sounds straightforward, right? Unfortunately, conducting randomized controlled trials (RCTs) isn't always feasible. This brings us to our next challenge: how can we extract causal insights from **observational data**? Specifically, what controls should we include in the regression? Can’t we just include every available variable and let the model do its magic? (Spoiler: if that worked, there would be no need for this article!)

### Identifying Proper Controls

First, let’s discuss the variables we must include in our model. In our earlier example, assume the **advertising budget** influences both discounts and purchases. Retailers may pair discounts with large advertising campaigns, and advertising itself can increase purchases by raising awareness. Here, the advertising budget acts as a **confounder**. If we omit this confounder from our regression, the estimated impact of discounts ($\hat{\beta}_1$) will also capture the effect of advertising.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import statsmodels.api as sm

np.random.seed(42)

n = 500
confounder = np.random.normal(0, 1, n) # Advertising budget
X = 2 * confounder + np.random.normal(0, 1, n) # Discount
Y = 3 * X + 4 * confounder + np.random.normal(0, 1, n) # Purchase

data = pd.DataFrame({'X': X, 'Z': confounder, 'Y': Y})

X_only = sm.add_constant(data['X'])
model_omitted = sm.OLS(data['Y'], X_only).fit()

XZ = sm.add_constant(data[['X', 'Z']])
model_controlled = sm.OLS(data['Y'], XZ).fit()

print("Regression without controlling for Z (omitted variable bias):")
print(model_omitted.summary())
print("Regression controlling for Z (true model):")
print(model_controlled.summary())
```

```
Regression without controlling for confounder (omitted variable bias):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          0.0616      0.089      0.693      0.488      -0.113       0.236
X              4.6321      0.042    110.762      0.000       4.550       4.714
==============================================================================
```

```
Regression controlling for confounder (true model):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          0.1065      0.045      2.361      0.019       0.018       0.195
X              3.0745      0.046     66.445      0.000       2.984       3.165
Z              3.7972      0.100     37.887      0.000       3.600       3.994
==============================================================================

```

By excluding the confounder, we overestimate the effect of discounts on purchases.

---

### Avoiding Bad Controls

Now that we understand the importance of including confounders, let’s discuss **bad controls**—variables that should **not** be included in the model.

#### 1. Mediators

Imagine we aim to measure the effect of education ($X$) on income ($Y$), but we include skills as a control variable. Since skills are developed through education and directly affect income, they act as a **mediator** because they lie on the causal path between the treatment and the outcome. Controlling for skills would block part of education’s effect on income, leading to a biased estimate of the **total effect**.

```python
np.random.seed(42)

n = 500
education = np.random.normal(12, 2, n)
skills = 1.5 * education + np.random.normal(0, 1, n)
income = 2 * education + 4 * skills + np.random.normal(0, 2, n)

data = pd.DataFrame({'Education': education, 'Skills': skills, 'Income': income})

X1 = sm.add_constant(data['Education'])
model1 = sm.OLS(data['Income'], X1).fit()

X2 = sm.add_constant(data[['Education', 'Skills']])
model2 = sm.OLS(data['Income'], X2).fit()

print("Without controlling for skills:\n", model1.summary())
print("Controlling for skills:\n", model2.summary())
```

```
Without controlling for skills:
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          2.8712      1.256      2.286      0.023       0.404       5.339
Education      7.7897      0.103     75.499      0.000       7.587       7.992
==============================================================================
```

```
Controlling for skills:
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          0.8596      0.562      1.531      0.126      -0.244       1.963
Education      1.7228      0.143     12.054      0.000       1.442       2.004
Skills         4.1489      0.093     44.833      0.000       3.967       4.331
==============================================================================
```

In practical terms, when we control for skills, we inadvertently narrow our focus to only the variation in education that does not impact skills—ignoring a crucial mechanism through which education influences income. As a result, the regression estimates reflect only the direct effect of education on income, not the complete picture.

A similar problem arises if we control for variables that are **descendants of mediators**—those influenced by mediators. For instance, let’s say education enhances skills (a mediator), which in turn leads to better job performance evaluations. Controlling for job performance would bias our estimate of education’s total effect on income because we would once again be blocking part of the causal path.

Keep in mind that while controlling for skills (a mediator) biases the estimate of the total effect of education on income, it allows us to estimate the direct effect of education that is not transmitted through skills. Depending on the research question, estimating the direct effect might be of interest.

```python
np.random.seed(42)

n = 500
education = np.random.normal(12, 2, n)
skills = 1.5 * education + np.random.normal(0, 1, n)
performance = 2 * skills + np.random.normal(0, 1, n)
income = 2 * education + 4 * skills + np.random.normal(0, 2, n)

data = pd.DataFrame({'Education': education, 'Skills': skills, 'Income': income, 'Performance':performance})

X1 = sm.add_constant(data['Education'])
model1 = sm.OLS(data['Income'], X1).fit()

X2 = sm.add_constant(data[['Education', 'Performance']])
model2 = sm.OLS(data['Income'], X2).fit()

print("Without controlling for performance:\n", model1.summary())
print("Controlling for performance:\n", model2.summary())
```

```
Without controlling for performance:
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          1.2331      1.206      1.023      0.307      -1.136       3.602
Education      7.9135      0.099     79.897      0.000       7.719       8.108
==============================================================================
```

```
Controlling for performance:
===============================================================================
                  coef    std err          t      P>|t|      [0.025      0.975]
-------------------------------------------------------------------------------
const          -0.9580      0.736     -1.301      0.194      -2.404       0.488
Education       3.4954      0.163     21.473      0.000       3.176       3.815
Performance     1.5262      0.052     29.208      0.000       1.424       1.629
==============================================================================
```

---

#### 2. Colliders

Colliders are variables influenced by both the treatment and the outcome. For example, consider a scenario where both education and income impact **networking opportunities**. Let's see what happens when we control for this:

```python
np.random.seed(42)

n = 500
education = np.random.normal(12, 2, n)
income = 2 * education + np.random.normal(0, 2, n)
networking = 1.5 * education + 2 * income + np.random.normal(0, 1, n)

data = pd.DataFrame({'Education': education, 'Income': income, 'Networking':networking})

X1 = sm.add_constant(data['Education'])
model1 = sm.OLS(data['Income'], X1).fit()

X2 = sm.add_constant(data[['Education', 'Networking']])
model2 = sm.OLS(data['Income'], X2).fit()

print("Without controlling for networking:\n", model1.summary())
print("Controlling for networking:\n", model2.summary())
```

```
Without controlling for networking:
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          0.9697      0.542      1.789      0.074      -0.095       2.035
Education      1.9246      0.045     43.216      0.000       1.837       2.012
==============================================================================
```

```
Controlling for networking:
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         -0.1398      0.134     -1.045      0.296      -0.403       0.123
Education     -0.5292      0.030    -17.679      0.000      -0.588      -0.470
Networking     0.4613      0.005     88.057      0.000       0.451       0.472
==============================================================================
```

Controlling for networking, we see that education has a negative effect on income. When we control for networking, we're looking at the relationship between education and income as if everyone had the same level of networking. Since both income and education affects networking, it is possible that a person with **high education** and **low income**, or **low education** and **high income** has a good level of networking. In other words, we're essentially comparing people who have achieved similar networking status but possibly through different paths.

The same logic applies to variables that are descendants of colliders. Controlling for such variables introduces a spurious association between the treatment and outcome, leading to misleading conclusions.

---

#### 3. M-Structure Bias

The **M-structure** is a slightly more complex form of bias that occurs when a variable is influenced by two unobserved factors that also independently affect the treatment and the outcome. For example:

- $U_1$ represents intrinsic motivation, influencing both education (treatment) and participation in extracurricular activities ($W$).
- $U_2$ represents social networks, influencing both income (outcome) and extracurricular activities ($W$).

Extracurricular activities ($W$) thus act as a **collider** affected by $U_1$ and $U_2$.

These variables and causal paths form a structure which looks like the letter 'M' (hence the name). This is shown in [Figure 1](#figure-1).
{{< figure src="/mstructure.png" alt="mstructure" align=center caption="M-Structure (adapted from Ding, P., & Miratrix, L. W. (2015))" width="200" >}}

Let's see what will happen if we control for W:

```python
np.random.seed(42)

n = 500
u1 = np.random.normal(0, 1, n)
u2 = np.random.normal(0, 1, n)
education = 1.5 * u1 + np.random.normal(12, 2, n)
income = 2 * education + 3 * u2 + np.random.normal(0, 2, n)
w = 1.5 * u1 + 2.5 * u2 + np.random.normal(0, 1, n)

data = pd.DataFrame({
    'Education': education,
    'Income': income,
    'W': w
})

X1 = sm.add_constant(data['Education'])
model1 = sm.OLS(data['Income'], X1).fit()

X2 = sm.add_constant(data[['Education', 'W']])
model2 = sm.OLS(data['Income'], X2).fit()

print("Without controlling for W (total effect):\n", model1.summary())
print("Controlling for W (biased estimate):\n", model2.summary())
```

```
Without controlling for W (total effect):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         -0.2980      0.804     -0.371      0.711      -1.877       1.281
Education      2.0376      0.064     31.604      0.000       1.911       2.164
==============================================================================
```

```
Controlling for W (biased estimate):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const          3.1650      0.608      5.206      0.000       1.971       4.359
Education      1.7489      0.049     35.758      0.000       1.653       1.845
W              0.8515      0.040     21.032      0.000       0.772       0.931
==============================================================================
```

The result shows that $W$ has a significant coefficient in the second regression due to the spurious association introduced by conditioning on it, despite $W$ not being part of the direct causal pathway between education and income. Why is that? As we've discussed before, conditioning on W means we decide to focus only on people who participate in extracurricular activities ($W$) at a similar level. What happens here is that different combinations of $U_1$ and $U_2$ lead to the same $W$. For instance, Person A might have a high $U_1$ (high intrinsic motivation) but a low $U_2$ (weak social network), leading to moderate participation in extracurriculars ($W$). Person B might have a low $U_1$ but a high $U_2$, resulting in the same level of $W$. Person A's high $U_1$ leads to higher education ($T$), while Person B's high $U_2$ leads to higher income ($Y$). When we condition on $W$, these two effects seem related: people with higher education also appear to have higher income; but this relationship is spurious and introduced by conditioning on $W$.

---

#### 4. Bias Amplification

Let’s consider a scenario where we already have an **omitted variable bias (OVB)** due to an unmeasured factor, such as innate ability ($U_1$), which affects both education and income. There is also access to resource (X) like good schools, which only affects education.

```python
np.random.seed(42)

n = 500
u1 = np.random.normal(0, 1, n)
x = np.random.normal(0, 1, n)
education = 1.5 * u1 + 4 * x + np.random.normal(12, 2, n)
income = 2 * education + 3 * u1+ np.random.normal(0, 2, n)

data = pd.DataFrame({
    'Education': education,
    'Income': income,
    'X': x
})

X1 = sm.add_constant(data['Education'])
model1 = sm.OLS(data['Income'], X1).fit()

X2 = sm.add_constant(data[['Education', 'X']])
model2 = sm.OLS(data['Income'], X2).fit()

print("Without controlling for X (biased due to omitted variable u_1):\n", model1.summary())
print("Controlling for X (bias amplified due to conditioning on X):\n", model2.summary())
```

```
Without controlling for X (biased due to omitted variable u_1):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         -1.6623      0.457     -3.640      0.000      -2.560      -0.765
Education      2.1416      0.035     61.873      0.000       2.074       2.210
==============================================================================
```

```
Controlling for X (bias amplified due to conditioning on X):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         -8.3924      0.741    -11.327      0.000      -9.848      -6.937
Education      2.6943      0.059     45.326      0.000       2.577       2.811
X             -3.0782      0.282    -10.913      0.000      -3.632      -2.524
==============================================================================
```

It seems that controlling for X exacerbates our existing bias. This is because by focusing on people with the same X, any difference in education between these individuals must now come from their innate ability ($U_1$). That is, if two people had the same access to resources, the one with higher education likely had a higher innate ability. As a result, since X is no longer contributing to the differences in education, innate ability explains a larger share of the variation in education. Remember, $U_1$ also directly affects income. So, when education appears higher for someone due to their innate ability, part of their higher income is due to the same innate ability. This means some of the effect of innate ability on income get wrongly attributed to education.

# Conclusion

In this article, we explored various sources of bias that can arise when deciding which variables to include in a regression model. These include:

- **Mediator bias**: Blocking part of the causal path by controlling for variables that mediate the effect of the treatment on the outcome.
- **Collider bias**: Introducing spurious associations by controlling for variables influenced by both the treatment and the outcome.
- **M-bias**: Creating artificial relationships between unobserved variables by conditioning on colliders.
- **Bias amplification**: Exacerbating existing biases by conditioning on a variable that has no effect on the outcome except for an indirect effect via treatment.

The key takeaway is that we should carefully consider the causal relationships among variables before deciding whether to adjust for a particular variable.

## References

1. Cinelli, C., Forney, A., & Pearl, J. (2022). _A Crash Course in Good and Bad Controls. Sociological
   Methods & Research_, 0 (0), 1-34.
2. Ding, P., & Miratrix, L. W. (2015). _To adjust or not to adjust? Sensitivity analysis of M-bias and butterfly-bias. Journal of Causal Inference_, 3(1), 41-57.
3. Facure, M. (2023). Causal Inference in Python. " O'Reilly Media, Inc.".
