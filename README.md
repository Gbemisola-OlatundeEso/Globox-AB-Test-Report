## Globox-AB-Test-Report
### Hypothesis Testing in Python
 

```python
# To import the packages and load the Globox dataset
import pandas as pd
import numpy as np
from scipy import stats

globox_df = pd.read_csv('/content/Globox_analysis_dataset.csv')
```
```python
# Inspect the first 5 rows of the dataset
globox_df.head()
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/7087425a-e64d-4401-9788-0b5390dac5b6)

```python
# To understand the total number of entries in each column, data types and missing values.
globox_df.info()
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/35f901e0-3011-45f4-bc99-813291fa5340)

```python
#Find the count of globox user and conversion in each group
globox_df_summary = globox_df.groupby(by=['group'],
                                      as_index=False).agg({'converted': ['count', 'sum']})

# Calculate the conversion rate per group
globox_df_summary['conversion_rate'] = np.round((globox_df_summary.converted['sum'] /
                                        globox_df_summary.converted['count']),4)
globox_df_summary
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/19b61ddb-00ab-4e6e-ae65-6cb94b20d1b9)

### Hypotheses- Conversion rate

- Null Hypothesis (H₀):
There is no difference in the user conversion rate between the control group(Group A) and the treatment group (Group B).

- Alternative Hypothesis (H₁):
There is a difference in the user conversion rate between the control group(Group A) and the treatment group (Group B).

#### Statistical notations:

H₀: p₁ = p₂

H₁: p₁ ≠ p₂

Here, p₁ is the proportion of populations in the control group, and p₂ represents the proportion of the population in the treatment group.

```python
# Calculate the number of users in Group A- Control and Group B- Treatment
n_A = globox_df[globox_df['group'] == 'Control']['id'].nunique()
n_B = globox_df[globox_df['group'] == 'Treatment']['id'].nunique()

print('Group A- Control users:', n_A)
print('Group B- Treatment users:', n_B)
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/68ef4fda-9b55-47e1-a217-c4ad1a0ec079)

```python
# Calculate the baseline conversion rate for Group A- Control and Group B- Treatment
p_A = globox_df[globox_df['group'] == 'Control']['converted'].mean()
p_B = globox_df[globox_df['group'] == 'Treatment']['converted'].mean()

print('Group A- Control conversion rate:', p_A)
print('Group B- Treatment conversion rate:', p_B)
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/67fbb365-ba34-46f6-8733-8834b5ac02e4)

Performed the calculation of the two-sample Z-test with pooled proportion to compare the conversion rates between the two groups.

Significance threshold (𝛼) = 0.05
```python
# Calculate the pooled proportion
#Calculate the conversion rate per user
conversion_rate = np.round(globox_df.converted.mean(),4)
print(f'Conversion rate: {conversion_rate}')
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/9a502be6-1bec-489e-86ab-b5bcaf6f95c6)

```python
# Calculate standard error using the pooled proportion
SE = (conversion_rate * (1 - conversion_rate) * (1/n_A + 1/n_B)) ** 0.5

# Calculate z-test statistic
z_test_statistic = np.round((p_A- p_B) / SE,4)

# Calculate p-value
p_value = np.round(2 * (1 - stats.norm.cdf(abs(z_test_statistic))),4)

# Analyze the results

alpha = 0.05
print(f"Z-test Statistic: {z_test_statistic}")
print(f"P-value: {p_value}")

if p_value < alpha:
  print("Reject the null hypothesis that there is no significant difference in the \n
user conversion rate between the control and treatment groups.")
else:
  print("Fail to reject the null hypothesis that there is no significant difference \n
in the user conversion rate between the control and treatment groups.")
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/f04d5705-b11e-4801-b5b3-0a5670bf4c32)


The above is a hypothesis test conducted to assess whether there is a difference in the conversion rates between the two groups, 
the resulting p-value is 0.0001 which is less than the significance level (𝛼) 0.05.

Consequently, we reject the null hypothesis, indicating a difference in the conversion rates between Group A (control) and Group B(Treatment).


#### Calculated the 95% confidence interval using the two-sample Z-interval for a difference in proportions.
```python
# Calculate 95% confidence intervals
#Calculate unpooled proportions and standard errors

se_A = np.sqrt(p_A * (1 - p_A) / n_A)
se_B = np.sqrt(p_B * (1 - p_B) / n_B)

# Calculate the margin of error
margin_of_error_conversion = 1.96 * np.sqrt(se_A**2 + se_B**2)

# Calculate the confidence interval for the difference
confidence_interval_lower = (p_B - p_A) - margin_of_error_conversion
confidence_interval_upper = (p_B - p_A) + margin_of_error_conversion

# Print Results
print(f"95% Confidence interval for the difference: ({np.round(confidence_interval_lower,4)}, {np.round(confidence_interval_upper,4)})")
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/d8c0e193-e329-4e36-966c-750dc5270a58)

### Hypotheses - Average amount spent

- Null Hypothesis (H₀):
There is no difference in the average amount spent per user between the control group(Group A) and the treatment group (Group B).

- Alternative Hypothesis (H₁):
There is a difference in the average amount spent per user between the control group(Group A) and the treatment group (Group B).

### Statistical notation:

H0: μ1 = μ2

H1: μ1 != μ2

Here, μ1 is the means for populations in the control group, and μ2 represents the means of the population in the treatment group.

```python
# Separate the data into Group A- Control and Group B- Treatment
cont_grp = globox_df[globox_df['group'] == 'Control']['total_spent']
treat_grp = globox_df[globox_df['group'] == 'Treatment']['total_spent']

# Perform a t-test
t_stat, p_value = stats.ttest_ind(cont_grp, treat_grp, equal_var=False)

# Analyze the results

alpha = 0.05

print(f'T-statistic: {np.round(t_stat,4)}')
print(f'P-value: {np.round(p_value,3)}')

if p_value < alpha:
  print("Reject the null hypothesisthat there is no difference in the \n
average amount spent per user between the control and treatment.")
else:
  print("Fail to reject the null hypothesis that there is no difference in the \n
average amount spent per user between the control and treatment.")
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/e494398c-c6b9-4b33-bdc2-008f684652a3)

```python
#Calculate the average amount spent per user
avg_amount_spent = np.round(globox_df.total_spent.mean(),2)
print('Average amount spent per user:', avg_amount_spent)
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/0cab8671-4c3e-4cfa-8017-45194522a84d)

```python
# Calculate the means and standard deviations for each group
mean_treat, mean_cont = treat_grp.mean(), cont_grp.mean()
std_treat, std_cont = treat_grp.std(), cont_grp.std()

n_treat, n_cont = len(treat_grp), len(cont_grp)

# Set the confidence level and degrees of freedom
confidence_level = 0.95
degrees_of_freedom = min(n_treat - 1, n_cont - 1)

# Calculate the standard error of the difference
standard_error = ((std_treat ** 2 / n_treat) + (std_cont ** 2 / n_cont)) ** 0.5

# Calculate the margin of error
margin_of_error_spent = stats.t.ppf((1 + confidence_level) / 2, degrees_of_freedom) * standard_error

# Calculate the confidence interval
confidence_interval_lo_spent = (mean_treat - mean_cont) - margin_of_error_spent
confidence_interval_up_spent = (mean_treat - mean_cont) + margin_of_error_spent

# Display the results
print(f'95% Confidence interval for the difference: ({confidence_interval_lo_spent}, {confidence_interval_up_spent})')
```
![image](https://github.com/Gbemisola-OlatundeEso/Globox-AB-Test-Report/assets/169208623/430e256c-67be-4e51-9d7d-fc525a8ebe04)

