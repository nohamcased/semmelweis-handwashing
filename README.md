# semmelweis-handwashing

## 1. Meet Dr. Ignaz Semmelweis

<img width="269" alt="Screen Shot 2023-04-08 at 21 51 59" src="https://user-images.githubusercontent.com/86967515/230751758-313097fd-6490-437b-930e-d5bb2abddb8b.png">

<p>This is Dr. Ignaz Semmelweis, a Hungarian physician born in 1818 and active at the Vienna General Hospital. If Dr. Semmelweis looks troubled it's probably because he's thinking about <em>childbed fever</em>: A deadly disease affecting women that just have given birth. He is thinking about it because in the early 1840s at the Vienna General Hospital as many as 10% of the women giving birth die from it. He is thinking about it because he knows the cause of childbed fever: It's the contaminated hands of the doctors delivering the babies. And they won't listen to him and <em>wash their hands</em>!</p>
<p>In this notebook, I'm going to reanalyze the data that made Semmelweis discover the importance of <em>handwashing</em>.

```
# Importing modules
import pandas as pd

# Read datasets/yearly_deaths_by_clinic.csv into yearly
yearly = pd.read_csv("datasets/yearly_deaths_by_clinic.csv")

# Print out yearly
print(yearly)
```
<img width="311" alt="Screen Shot 2023-04-08 at 21 52 36" src="https://user-images.githubusercontent.com/86967515/230751761-bcc39a17-d3bd-4e1b-b973-0eeff575d597.png">

## 2. The alarming number of deaths
<p>The table above shows the number of women giving birth at the two clinics at the Vienna General Hospital for the years 1841 to 1846. You'll notice that giving birth was very dangerous; an <em>alarming</em> number of women died as the result of childbirth, most of them from childbed fever.</p>
<p>We see this more clearly if we look at the <em>proportion of deaths</em> out of the number of women giving birth. I'm going to zoom in on the proportion of deaths at Clinic 1.</p>

```
# Calculate proportion of deaths per no. births
yearly["proportion_deaths"] = yearly["deaths"] / yearly["births"]

# Extract Clinic 1 data into clinic_1 and Clinic 2 data into clinic_2
clinic_1 = yearly[yearly["clinic"] == "clinic 1"]
clinic_2 = yearly[yearly["clinic"] == "clinic 2"]

# Print out clinic_1
print(clinic_1)
```
<img width="448" alt="Screen Shot 2023-04-08 at 21 53 10" src="https://user-images.githubusercontent.com/86967515/230751763-673fda07-516b-4fde-b5c9-48ac2f4ed12c.png">

## 3. Death at the clinics
<p>If I now plot the proportion of deaths at both Clinic 1 and Clinic 2  I'll see a curious pattern…</p>

```
# Import matplotlib
import matplotlib as plt

# This makes plots appear in the notebook
%matplotlib inline

# Plot yearly proportion of deaths at the two clinics
ax = clinic_1.plot(x="year", y="proportion_deaths", label="Clinic 1")
clinic_2.plot(x="year", y="proportion_deaths", label="Clinic 2", ax=ax, ylabel="proportion of deaths")
```
<img width="592" alt="Screen Shot 2023-04-08 at 21 53 48" src="https://user-images.githubusercontent.com/86967515/230751801-83911a7e-f6d5-471e-a6b0-14e19d3e5ff1.png">

## 4. The handwashing begins
<p>Why is the proportion of deaths consistently so much higher in Clinic 1? Semmelweis saw the same pattern and was puzzled and distressed. The only difference between the clinics was that many medical students served at Clinic 1, while mostly midwife students served at Clinic 2. While the midwives only tended to the women giving birth, the medical students also spent time in the autopsy rooms examining corpses. </p>
<p>Semmelweis started to suspect that something on the corpses spread from the hands of the medical students, caused childbed fever. So in a desperate attempt to stop the high mortality rates, he decreed: <em>Wash your hands!</em> This was an unorthodox and controversial request, nobody in Vienna knew about bacteria at this point in time. </p>
<p>I'm going to load in monthly data from Clinic 1 to see if the handwashing had any effect.</p>


```
# Read datasets/monthly_deaths.csv into monthly
monthly = pd.read_csv("datasets/monthly_deaths.csv", parse_dates=["date"])

# Calculate proportion of deaths per no. births
monthly["proportion_deaths"] = monthly["deaths"] / monthly["births"]

# Print out the first rows in monthly
print(monthly.head())
```

## 5. The effect of handwashing
<p>With the data loaded we can now look at the proportion of deaths over time. In the plot below I haven't marked where obligatory handwashing started, but it reduced the proportion of deaths to such a degree that you should be able to spot it!</p>

```
# Plot monthly proportion of deaths
ax = monthly.plot(x="date", y="proportion_deaths", label="Proportion deaths")
```
<img width="592" alt="Screen Shot 2023-04-08 at 21 54 19" src="https://user-images.githubusercontent.com/86967515/230751773-ef7a3976-974b-4d46-99f5-d75372d28d72.png">

## 6. The effect of handwashing highlighted
<p>Starting from the summer of 1847 the proportion of deaths is drastically reduced and, yes, this was when Semmelweis made handwashing obligatory. </p>
<p>The effect of handwashing is made even more clear if we highlight this in the graph.</p>


```
# Date when handwashing was made mandatory
handwashing_start = pd.to_datetime('1847-06-01')

# Split monthly into before and after handwashing_start
before_washing = monthly[monthly["date"] < handwashing_start]
after_washing = monthly[monthly["date"] >= handwashing_start]

# Plot monthly proportion of deaths before and after handwashing
ax = before_washing.plot(x="date", y="proportion_deaths", label="Before")
after_washing.plot(x="date", y="proportion_deaths", label="After", ax=ax, ylabel="Proportion Deaths")
```
<img width="592" alt="Screen Shot 2023-04-08 at 21 54 34" src="https://user-images.githubusercontent.com/86967515/230751778-32ec56cf-edca-4f07-9354-a4a078233b18.png">

## 7. More handwashing, fewer deaths?
<p>Again, the graph shows that handwashing had a huge effect. How much did it reduce the monthly proportion of deaths on average?</p>

```
# Import numpy
import numpy as np

# Difference in mean monthly proportion of deaths due to handwashing
before_proportion = before_washing["proportion_deaths"]
after_proportion = after_washing["proportion_deaths"]
mean_diff = np.mean(after_proportion) - np.mean(before_proportion)
mean_diff
```

## 8. A Bootstrap analysis of Semmelweis handwashing data
<p>It reduced the proportion of deaths by around 8 percentage points! From 10% on average to just 2% (which is still a high number by modern standards). </p>
<p>To get a feeling for the uncertainty around how much handwashing reduces mortalities I could look at a confidence interval (here calculated using the bootstrap method).</p>

```
# A bootstrap analysis of the reduction of deaths due to handwashing
boot_mean_diff = []
for i in range(3000):
    boot_before = before_proportion.sample(frac=1, replace=True)
    boot_after = after_proportion.sample(frac=1, replace=True)
    boot_mean_diff.append(boot_after.mean() - boot_before.mean())

# Calculating a 95% confidence interval from boot_mean_diff 
confidence_interval = pd.Series(boot_mean_diff).quantile([0.025,0.975])
confidence_interval
```

## 9. The fate of Dr. Semmelweis
<p>So handwashing reduced the proportion of deaths by between 6.7 and 10 percentage points, according to a 95% confidence interval. All in all, it would seem that Semmelweis had solid evidence that handwashing was a simple but highly effective procedure that could save many lives.</p>
<p>The tragedy is that, despite the evidence, Semmelweis' theory — that childbed fever was caused by some "substance" (what we today know as <em>bacteria</em>) from autopsy room corpses — was ridiculed by contemporary scientists. The medical community largely rejected his discovery and in 1849 he was forced to leave the Vienna General Hospital for good.</p>
<p>One reason for this was that statistics and statistical arguments were uncommon in medical science in the 1800s. Semmelweis only published his data as long tables of raw data, but he didn't show any graphs nor confidence intervals. If he would have had access to the analysis we've just put together he might have been more successful in getting the Viennese doctors to wash their hands.</p>

```
# The data Semmelweis collected points to that:
doctors_should_wash_their_hands = True
```
