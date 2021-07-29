
# Matplotlib Homework - The Power of Plots

## Pymaceuticals

While your data companions rushed off to jobs in finance and government, you remained adamant that science was the way for you. Staying true to your mission, you've joined Pymaceuticals Inc., a burgeoning pharmaceutical company based out of San Diego. Pymaceuticals specializes in anti-cancer pharmaceuticals. In its most recent efforts, it began screening for potential treatments for squamous cell carcinoma (SCC), a commonly occurring form of skin cancer.
As a senior data analyst at the company, you've been given access to the complete data from their most recent animal study. In this study, 249 mice identified with SCC tumor growth were treated through a variety of drug regimens. Over the course of 45 days, tumor development was observed and measured. The purpose of this study was to compare the performance of Pymaceuticals' drug of interest, Capomulin, versus the other treatment regimens. You have been tasked by the executive team to generate all of the tables and figures needed for the technical report of the study. The executive team also has asked for a top-level summary of the study results.

-----

# Observable Trends

* Based on the calculated total tumor volume displayed in the box plots below, Capomulin and Ramicane seem to be more effective than Infubinol and Ceftamine.

* The data collected from this study is very statistically significant becaue we only observe one outlier.

* Based on the strong r-correlation (.84) between mouse weight and tumor volume, drug regimens are less effective on heavier mice.

-----

# Set up jupyter notebook

```python
# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import scipy.stats as st

# Study data files
mouse_metadata_path = "data/Mouse_metadata.csv"
study_results_path = "data/Study_results.csv"
```

-----

# New Mouse Data DataFrame

* Before beginning the analysis, check the data for any mouse ID with duplicate time points and remove any data associated with that mouse ID.

```Python
# Read and display the mouse data
mouse_metadata = pd.read_csv(mouse_metadata_path)
mouse_metadata.sample(10)

# Read and display the study results
study_results = pd.read_csv(study_results_path)
study_results.sample(10)

# Combine the data into a single dataset
mouse_study = pd.merge(mouse_metadata, study_results, on='Mouse ID')

# Display the data table for preview
mouse_study.sample(10)

# Checking the number of mice.
len(mouse_study)

# Getting the duplicate mice by ID number that shows up for Mouse ID and Timepoint. 
# Optional: Get all the data for the duplicate mouse ID. 
duplicate_mice = mouse_study[mouse_study.duplicated(['Mouse ID','Timepoint'])]
duplicate_mice

# Create a clean DataFrame by dropping the duplicate mouse by its ID.
mouse_study.drop(mouse_study[mouse_study['Mouse ID']=='g989'].index, inplace=True)
mouse_study

# Checking the number of mice in the clean DataFrame.
len(mouse_study)
```

-----

# Summary Statistics

* Generate a summary statistics table consisting of the mean, median, variance, standard deviation, and SEM of the tumor volume for each drug regimen.

```Python
# Generate a summary statistics table of mean of the tumor volume for each regimen
tumor_volume_mean = mouse_study.groupby('Drug Regimen').mean()['Tumor Volume (mm3)']
print(tumor_volume_mean)

# Generate a summary statistics table of median of the tumor volume for each regimen
tumor_volume_median = mouse_study.groupby('Drug Regimen').median()['Tumor Volume (mm3)']
print(tumor_volume_median)

# Generate a summary statistics table of variance of the tumor volume for each regimen
tumor_volume_var = mouse_study.groupby('Drug Regimen').var()['Tumor Volume (mm3)']
print(tumor_volume_var)

# Generate a summary statistics table of standard deviation of the tumor volume for each regimen
tumor_volume_std = mouse_study.groupby('Drug Regimen').std()['Tumor Volume (mm3)']
print(tumor_volume_std)

# Generate a summary statistics table of SEM of the tumor volume for each regimen
tumor_volume_sem = mouse_study.groupby('Drug Regimen').sem()['Tumor Volume (mm3)']
print(tumor_volume_sem)

# Generate initial dataframe for summary table for summary statistics of tumor volume for each regimen
tumor_volume_summary_statistics = pd.DataFrame(tumor_volume_mean)
tumor_volume_summary_statistics = tumor_volume_summary_statistics.rename(columns={'Tumor Volume (mm3)':'Tumor Volume Mean'})
tumor_volume_summary_statistics

# Generate summary table for summary statistics of tumor volume for each regimen
tumor_volume_summary_statistics['Tumor Volume Median']=tumor_volume_median
tumor_volume_summary_statistics['Tumor Volume Variance']=tumor_volume_var
tumor_volume_summary_statistics['Tumor Volume Standard Deviation']=tumor_volume_std
tumor_volume_summary_statistics['Tumor Volume Standard Error']=tumor_volume_sem
tumor_volume_summary_statistics
```

-----

# Bar and Pie Charts

* Generate a bar plot using both Pandas's DataFrame.plot() and Matplotlib's pyplot that shows the total number of timepoints for all mice tested for each drug regimen throughout the course of the study.

* Generate a pie plot using both Pandas's DataFrame.plot() and Matplotlib's pyplot that shows the distribution of female or male mice in the study.

* NOTE: These plots should look identical.

```Python
# Pull total timepoints by drug regimen
drug_regiment_timepoints = mouse_study['Drug Regimen'].value_counts()
drug_regiment_timepoints

# Generate a bar plot showing the total number of timepoints for all mice tested for each drug regimen using Pandas.
mouse_study_pdBarPlot = drug_regiment_timepoints.plot.bar(title='Mice Tested by Drug Regimen')
mouse_study_pdBarPlot.set_xlabel('Drug Regimen')
mouse_study_pdBarPlot.set_ylabel('Mice Count')

# Generate a bar plot showing the total number of timepoints for all mice tested for each drug regimen using pyplot.
mouse_study_pyplot_df = pd.DataFrame(drug_regiment_timepoints)
mouse_study_pyplot_df.plot.bar(legend=False)
plt.title('Mice Tested by Drug Regimen')
plt.xlabel('Drug Regimen')
plt.ylabel('Mice Count')
plt.show()

# Pull mice sex data needed to create pie charts
mouse_study_pie_data = mouse_study.groupby('Sex').nunique()['Mouse ID']
mouse_study_pie_data

# Generate a pie plot showing the distribution of female versus male mice using Pandas
mouse_study_pie_data.plot.pie(autopct='%1.1f%%',title="Mice Sex Distribution")

# Generate a pie plot showing the distribution of female versus male mice using pyplot
mouse_study_pyplot_pie_df=pd.DataFrame(mouse_study_pie_data)
plt.pie(mouse_study_pie_data, labels=mouse_study_pie_data.index, autopct='%1.1f%%')
plt.title('Mice Sex Distribution')
plt.show()
```

-----

# Quartiles, Outliers, and Boxplots

* Calculate the final tumor volume of each mouse across four of the most promising treatment regimens: Capomulin, Ramicane, Infubinol, and Ceftamin. Calculate the quartiles and IQR and quantitatively determine if there are any potential outliers across all four treatment regimens.

* Using Matplotlib, generate a box and whisker plot of the final tumor volume for all four treatment regimens and highlight any potential outliers in the plot by changing their color and style.

* Hint: All four box plots should be within the same figure. Use this Matplotlib documentation page for help with changing the style of the outliers.

```Python
# Start by getting the last (greatest) timepoint for each mouse
max_timepoint = pd.DataFrame(mouse_study.groupby('Mouse ID')['Timepoint'].max().sort_values()).reset_index().rename(columns={'Timepoint': 'Max Timepoint'})
max_timepoint

# Merge max timepoint onto data_df
merged_mousedata = pd.merge(mouse_study, max_timepoint, on='Mouse ID')
merged_mousedata.sample(10)

# Calculate the final tumor volume of each mouse across four of the treatment regimens:  
# Calculate the IQR and quantitatively determine if there are any potential outliers:

# Place Capomulin, Ramicane, Infubinol, and Ceftamin drug regiments into a list
drug_regimens = ['Capomulin', 'Ramicane', 'Infubinol', 'Ceftamin']

# Create empty list to fill with tumor vol data (for plotting)
drug_values = []

# Put treatments into a list for for loop (and later for plot labels)
for drug in drug_regimens:

    # Create initial df to begin pulling total tumor volume by mouse from datafame of last (greatest) timepoint for each mouse
    volume_df = merged_mousedata.loc[merged_mousedata['Drug Regimen']==drug]

    # Locate the rows which contain mice on each drug and get the tumor volumes
    final_volume_df = volume_df.loc[volume_df['Timepoint'] == volume_df['Max Timepoint']]
    values = final_volume_df['Tumor Volume (mm3)']
    drug_values.append(values)
    
    # Determine outliers using upper and lower bounds
    quartiles = values.quantile([.25,.5,.75])
    lowerq = quartiles[0.25]
    upperq = quartiles[0.75]
    iqr = upperq-lowerq
    print(f'IQR for {drug}: {iqr}')
    
    # Find upper and lower bounds to help identify outliers for each regimen
    lower_bound = lowerq - (1.5*iqr)
    upper_bound = upperq + (1.5*iqr)
    print(f'Lower Bound for {drug}: {lower_bound}')
    print(f'Upper Bound for {drug}: {upper_bound}')
    
    # Quantitatively determine if there are any potential outliers
    outliers_count = (values.loc[(final_volume_df['Tumor Volume (mm3)'] >= upper_bound) | 
                                        (final_volume_df['Tumor Volume (mm3)'] <= lower_bound)]).count()
    print(f'Number of {drug} outliers: {outliers_count}')

# Generate a box plot of the final tumor volume of each mouse across four regimens of interest

# Highlight outliers
flierprops = dict(marker='o', markerfacecolor='blue', markersize=8, markeredgecolor='black')

# Plot box plot of all four regiments in one visual
plt.boxplot(drug_values, flierprops=flierprops)
plt.title('Final Tumor Volume by Drug Regimen')
plt.ylabel('Final Tumor Volume (mm3)')
plt.xticks([1, 2, 3, 4], ['Capomulin', 'Ramicane', 'Infubinol', 'Ceftamin'])
plt.show()
```

-----

# Line and Scatter Plots

* Calculate the correlation coefficient and linear regression model between mouse weight and average tumor volume for the Capomulin treatment. * Plot the linear regression model on top of the previous scatter plot.

```Python
#Generate a line plot of tumor volume vs. time point for a mouse treated with Capomulin
# Identify data points of a mouse treated with Capomulin
capomulin_data_lp = mouse_study.loc[mouse_study['Mouse ID'] == 'c766']

# Plot a line chart with the timepoint values on the x-axis and the tumor volume values on the y-axis
plt.plot(capomulin_data_lp['Timepoint'], capomulin_data_lp['Tumor Volume (mm3)'], marker = 'o')
plt.xlabel("Time (days)")
plt.ylabel("Tumor Volume (mm3)")
plt.title("Capomulin Treatment of Mouse c766")
plt.show()

# Generate a scatter plot of average tumor volume vs. mouse weight for the Capomulin regimen
capomulin_data_sp = mouse_study.loc[mouse_study['Drug Regimen'] == 'Capomulin']

# Find average tumor volume for each mouse

avg_tumor_vol = pd.DataFrame(capomulin_data_sp.groupby('Mouse ID')['Tumor Volume (mm3)'].mean().sort_values()).reset_index().rename(columns={'Tumor Volume (mm3)': 'Average Tumor Volume'})

# Merge average tumor volume onto data_df and drop duplicates
avg_tumor_vol = pd.merge(capomulin_data_sp, avg_tumor_vol, on='Mouse ID')
final_avg_tumor_vol = avg_tumor_vol[['Weight (g)', 'Average Tumor Volume']].drop_duplicates()
final_avg_tumor_vol

x = final_avg_tumor_vol['Weight (g)']
y = final_avg_tumor_vol['Average Tumor Volume']

# Create a scatter plot based on new dataframe above with circle markers and listed colors
plt.scatter(x, y)
plt.xlabel("Mouse Weight (g)")
plt.ylabel("Average Tumor Volume (mm3)")
plt.title('Average Tumor Volume by Weight')
plt.show()
```

-----

# Correlation and Regression

* Calculate the correlation coefficient and linear regression model between mouse weight and average tumor volume for the Capomulin treatment. Plot the linear regression model on top of the previous scatter plot.

```Python
# capomulin_data_sp = mouse_study.loc[mouse_study['Drug Regimen'] == 'Capomulin']
avg_tumor_vol = pd.DataFrame(capomulin_data_sp.groupby('Mouse ID')['Tumor Volume (mm3)'].mean().sort_values()).reset_index().rename(columns={'Tumor Volume (mm3)': 'Average Tumor Volume'})
avg_tumor_vol = pd.merge(capomulin_data_sp, avg_tumor_vol, on='Mouse ID')
final_avg_tumor_vol = avg_tumor_vol[['Weight (g)', 'Average Tumor Volume']].drop_duplicates()
final_avg_tumor_vol
x = final_avg_tumor_vol['Weight (g)']
y = final_avg_tumor_vol['Average Tumor Volume']

# Calculate the correlation coefficient between weight and average tumor volume
correlation = st.pearsonr(x,y)
print(f'The correlation coefficient between weight and average tumor volume on the Capomulin regimen is {round(correlation[0],2)}.')

# Calculate linear regression
(slope, intercept, rvalue, pvalue, stderr) = st.linregress(x, y)
regress_values = x * slope + intercept
line_eq = 'y = ' + str(round(slope,2)) + 'x +' + str(round(intercept,2))
line_eq

# Plot linear regression on top of scatter plot
plt.scatter(x,y)
plt.plot(x,regress_values,'r-')

# Annotate linear regression
plt.annotate(line_eq,(25,50),fontsize=20,color='red')

# Add labels and title to plot
plt.xlabel('Weight (g)')
plt.ylabel('Average Tumor Volume (mm3)')
plt.title('Average Tumor Volume by Weight')
plt.show()
```