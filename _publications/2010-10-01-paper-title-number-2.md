---
title: "Python work example 2"
collection: Projects
---
Lab 6: Crime and Penalty

# Run this cell to set up the notebook, but please don't change it.

​

# These lines import the Numpy and Datascience modules.

import numpy as np

from datascience import *

​

# These lines do some fancy plotting magic.

import matplotlib

%matplotlib inline

import matplotlib.pyplot as plt

plt.style.use('fivethirtyeight')

import warnings

warnings.simplefilter('ignore', FutureWarning)

1. A/B Testing

A/B testing is a form of hypothesis testing that allows you to make comparisons between two distributions.

You'll almost never be explicitly asked to perform an A/B test. Make sure you can identify situations where the test is appropriate and know how to correctly implement each step.

Question 1.1: The following statements are the unordered steps of an A/B hypothesis test:

    Find the value of the observed test statistic

    Choose a test statistic (typically the difference in means between two categories)

    Shuffle the labels of the original sample, find your simulated test statistic, and repeat many times

    Calculate the p-value based off your observed and simulated test statistics

    Define a null and alternate model

    Use the p-value and p-value cutoff to draw a conclusion about the null hypothesis

Make an array called ab_test_order that contains the correct order of an A/B test, where the first item of the array is the first step of an A/B test and the last item of the array is the last step of an A/B test

ab_test_order = make_array(5,2,1,3,4,6)

Question 1.2: If the null hypothesis of an A/B test is correct, should the order of labels affect the differences in means between each group? Why do we shuffle labels in an A/B test?

No.
2: Murder Rates

Punishment for crime has many philosophical justifications. An important one is that fear of punishment may deter people from committing crimes.

In the United States, some jurisdictions execute people who are convicted of particularly serious crimes, such as murder. This punishment is called the death penalty or capital punishment. The death penalty is controversial, and deterrence has been one focal point of the debate. There are other reasons to support or oppose the death penalty, but in this project we'll focus on deterrence.

The key question about deterrence is:

    Through our exploration, does instituting a death penalty for murder actually reduce the number of murders?

You might have a strong intuition in one direction, but the evidence turns out to be surprisingly complex. Different sides have variously argued that the death penalty has no deterrent effect and that each execution prevents 8 murders, all using statistical arguments! We'll try to come to our own conclusion.
The data

The main data source for this lab comes from a paper by three researchers, Dezhbakhsh, Rubin, and Shepherd. The dataset contains rates of various violent crimes for every year 1960-2003 (44 years) in every US state. The researchers compiled the data from the FBI's Uniform Crime Reports.

Since crimes are committed by people, not states, we need to account for the number of people in each state when we're looking at state-level data. Murder rates are calculated as follows:

murder rate for state X in year Y=number of murders in state X in year Ypopulation in state X in year Y∗100000

(Murder is rare, so we multiply by 100,000 just to avoid dealing with tiny numbers.)

murder_rates = Table.read_table('crime_rates.csv').select('State', 'Year', 'Population', 'Murder Rate').sort('State')

murder_rates.set_format("Population", NumberFormatter)

State 	Year 	Population 	Murder Rate
Alabama 	1960 	3,266,740 	12.4
Alabama 	1961 	3,302,000 	12.9
Alabama 	1962 	3,358,000 	9.4
Alabama 	1963 	3,347,000 	10.2
Alabama 	1964 	3,407,000 	9.3
Alabama 	1965 	3,462,000 	11.4
Alabama 	1966 	3,517,000 	10.9
Alabama 	1967 	3,540,000 	11.7
Alabama 	1968 	3,566,000 	11.8
Alabama 	1969 	3,531,000 	13.7

... (2190 rows omitted)

Murder rates vary over time, and different states exhibit different trends. The rates in some states change dramatically from year to year, while others are quite stable. Let's plot a few, just to see the variety.

Question 2.1. Create a table three_states with three columns of murder rates for the given states, in addition to a column of years. This table will have the following structure:
Year 	Murder rate in North Carolina 	Murder rate in Texas 	Murder rate in New York
1960 	10.6 	8.6 	2.9
1961 	9.2 	8.1 	3.5
1962 	7.9 	7.2 	3.6
... (41 rows omitted)

# Fill in this line to make a table like the one pictured above.

nc = murder_rates.where('State',are.equal_to('North Carolina')).select('Year','Murder Rate').relabel(1,'Murder rate in North Carolina')

tx = murder_rates.where('State',are.equal_to('Texas')).column('Murder Rate')

ny = murder_rates.where('State',are.equal_to('New York')).column('Murder Rate')

three_states =nc.with_columns('Murder rate in Texas',tx,'Murder rate in New York',ny)

three_states

Year 	Murder rate in North Carolina 	Murder rate in Texas 	Murder rate in New York
1960 	10.6 	8.6 	2.9
1961 	9.2 	8.1 	3.5
1962 	7.9 	7.2 	3.6
1963 	8.2 	7.4 	3.8
1964 	8 	7.6 	4.6
1965 	8.3 	7.5 	4.6
1966 	9.2 	9.1 	4.8
1967 	9.9 	9.9 	5.4
1968 	10.2 	10.6 	6.5
1969 	11.3 	11.3 	7.2

... (34 rows omitted)

Question 2.2: Using the table three_states, draw a line plot that compares the murder rates in North Carolina, Texas, and New York over time.

# Draw your line plot here

three_states.plot('Year')

3. The Death Penalty

Some US states have the death penalty, and others don't, and laws have changed over time. In addition to changes in murder rates, we will also consider whether the death penalty was in force in each state and each year.

Using this information, we would like to investigate how the presence of the death penalty affects the murder rate of a state.

Question 3.1. We want to know whether the death penalty causes a change in the murder rate. Why is it not sufficient to compare murder rates in places and times when the death penalty was in force with places and times when it wasn't?

Because such a comparison doesn't establish causation. While it's possible to observe correlations or associations between the presence of the death penalty and murder rates, correlation does not imply causation.
A Natural Experiment

In order to attempt to investigate the causal relationship between the death penalty and murder rates, we're going to take advantage of a natural experiment. A natural experiment happens when something other than experimental design applies a treatment to one group and not to another (control) group, and we have some hope that the treatment and control groups don't have any other systematic differences.

Our natural experiment is this: in 1972, a Supreme Court decision called Furman v. Georgia banned the death penalty throughout the US. Suddenly, many states went from having the death penalty to not having the death penalty.

As a first step, let's see how murder rates changed before and after the court decision. We'll define the test as follows:

    Population: All the states that had the death penalty before the 1972 abolition. (There is no control group for the states that already lacked the death penalty in 1972, so we must omit them.) This includes all US states except Alaska, Hawaii, Maine, Michigan, Wisconsin, and Minnesota.

    Treatment group: The states in that population, in 1973 (the year after 1972).

    Control group: The states in that population, in 1971 (the year before 1972).

    Null hypothesis: Murder rates in 1971 and 1973 come from the same distribution.

    Alternative hypothesis: Murder rates were higher in 1973 than they were in 1971.

Our alternative hypothesis is related to our suspicion that murder rates increase when the death penalty is eliminated.

Question 3.2: Should we use an A/B test to test these hypotheses? If yes, what is our "A" group and what is our "B" group?

Yes. A group should be murder rates in 1971 and B group being murder rates in 1973.

The death_penalty table below describes whether each state allowed the death penalty in 1971.

non_death_penalty_states = make_array('Alaska', 'Hawaii', 'Maine', 'Michigan', 'Wisconsin', 'Minnesota')

​

def had_death_penalty_in_1971(state):

    """Returns True if the argument is the name of a state that had the death penalty in 1971."""

    # The implementation of this function uses a bit of syntax

    # we haven't seen before.  Just trust that it behaves as its

    # documentation claims.

    return state not in non_death_penalty_states

​

states = murder_rates.group('State').select('State')

death_penalty = states.with_column('Death Penalty', states.apply(had_death_penalty_in_1971, 0))

death_penalty

State 	Death Penalty
Alabama 	True
Alaska 	False
Arizona 	True
Arkansas 	True
California 	True
Colorado 	True
Connecticut 	True
Delaware 	True
Florida 	True
Georgia 	True

... (40 rows omitted)

Question 3.3: Use the death_penalty and murder_rates tables to find murder rates in 1971 for states with the death penalty before the abolition. Create a new table preban_rates that contains the same information as murder_rates, along with a column Death Penalty that contains booleans (True or False) describing if states had the death penalty in 1971. This table should only include those states with the death penalty in 1971.

# States that had death penalty in 1971

preban_rates = murder_rates.where('Year',1971).join('State',death_penalty.where('Death Penalty',True))

preban_rates = preban_rates.sort("State")

preban_rates

State 	Year 	Population 	Murder Rate 	Death Penalty
Alabama 	1971 	3,479,000 	15.1 	True
Arizona 	1971 	1,849,000 	6.7 	True
Arkansas 	1971 	1,944,000 	10.5 	True
California 	1971 	20,223,000 	8.1 	True
Colorado 	1971 	2,283,000 	6.5 	True
Connecticut 	1971 	3,081,000 	3.1 	True
Delaware 	1971 	558,000 	6.1 	True
Florida 	1971 	7,041,000 	13.3 	True
Georgia 	1971 	4,664,000 	16 	True
Idaho 	1971 	732,000 	3.3 	True

... (34 rows omitted)

Question 3.4: Create a table postban_rates that contains the same information as preban_rates, but for 1973 instead of 1971. postban_rates should only contain the states found in preban_rates and all states should show that they do not have the death penalty.

postban_rates = murder_rates.where('Year',1973).join('State',preban_rates).select('State','Year','Population','Murder Rate').with_column('Death Penalty',False)

postban_rates = postban_rates.sort("State")

postban_rates

State 	Year 	Population 	Murder Rate 	Death Penalty
Alabama 	1973 	3,539,000 	13.2 	False
Arizona 	1973 	2,058,000 	8.1 	False
Arkansas 	1973 	2,037,000 	8.8 	False
California 	1973 	20,601,000 	9 	False
Colorado 	1973 	2,437,000 	7.9 	False
Connecticut 	1973 	3,076,000 	3.3 	False
Delaware 	1973 	576,000 	5.9 	False
Florida 	1973 	7,678,000 	15.4 	False
Georgia 	1973 	4,786,000 	17.4 	False
Idaho 	1973 	770,000 	2.6 	False

... (34 rows omitted)

Question 3.5: Use preban_rates_copy and postban_rates to create a table change_in_death_rates that contains each state's population, murder rate, and whether or not that state had the death penalty for both 1971 and 1973. The table should contain two rows for each state and have the same column labels as preban_rates.

Hint: tbl_1.append(tbl_2) will create a new table that includes rows from both tbl_1 and tbl_2. Both tables must have the exactly the same columns, in the same order.

preban_rates_copy = preban_rates.copy()

change_in_death_rates = preban_rates_copy.append(postban_rates)

change_in_death_rates

State 	Year 	Population 	Murder Rate 	Death Penalty
Alabama 	1971 	3,479,000 	15.1 	True
Arizona 	1971 	1,849,000 	6.7 	True
Arkansas 	1971 	1,944,000 	10.5 	True
California 	1971 	20,223,000 	8.1 	True
Colorado 	1971 	2,283,000 	6.5 	True
Connecticut 	1971 	3,081,000 	3.1 	True
Delaware 	1971 	558,000 	6.1 	True
Florida 	1971 	7,041,000 	13.3 	True
Georgia 	1971 	4,664,000 	16 	True
Idaho 	1971 	732,000 	3.3 	True

... (78 rows omitted)

Run the cell below to view the distribution of death rates during the pre-ban and post-ban time periods.

change_in_death_rates.hist('Murder Rate', group = 'Death Penalty')

Question 3.6: Create a table rate_means that contains the average murder rates for the states that had the death penalty and the states that didn't have the death penalty. It should have two columns: one indicating if the penalty was in place, and one that contains the average murder rate for each group.

rate_means = change_in_death_rates.select('Death Penalty','Murder Rate').group('Death Penalty',np.average)

rate_means

Death Penalty 	Murder Rate average
False 	8.12045
True 	7.51364

Question 3.7: We want to figure out if there is a difference between the distribution of death rates in 1971 and 1973. Specifically, we want to test if murder rates were higher in 1973 than they were in 1971.

What should the test statistic be? How does it help us differentiate whether the data supports the null and alternative?

The test statistic should be the difference in the average murder rates between the states that had the death penalty and the states that didn't.Large values favor the alternative hypothesis. 

Question 3.8: Set observed_difference to the observed test statistic using the rate_means table

observed_difference = rate_means.column('Murder Rate average').item(0)-rate_means.column('Murder Rate average').item(1)

observed_difference

0.6068181600659095

Question 3.9: Given a table like change_in_death_rates, a value column label, and a group column group_label, write a function that calculates the appropriate test statistic.

def find_test_stat(table, labels_col, values_col):

    new_table =table.group(labels_col,np.average)

    return new_table.column(values_col + ' average').item(0) - new_table.column(values_col + ' average').item(1)

    

​

find_test_stat(change_in_death_rates, "Death Penalty", "Murder Rate")

0.6068181600659095

When we run a simulation for A/B testing, we resample by shuffling the labels of the original sample. If the null hypothesis is true and the murder rate distributions are the same, we expect that the difference in mean death rates will be not change when "Death Penalty" labels are changed.

Question 3.10: Write a function simulate_and_test_statistic to compute one trial of our A/B test. Your function should run a simulation and return a test statistic.

def simulate_and_test_statistic(table, labels_col, values_col):

    shuffled_column =table.sample(with_replacement=False).column(labels_col)

    new_table=table.drop(labels_col).with_column('shuffled label',shuffled_column)

    return find_test_stat(new_table,shuffled_column,values_col)

​

simulate_and_test_statistic(change_in_death_rates, "Death Penalty", "Murder Rate")

-1.065909105252274

Question 3.11: Simulate 5000 trials of our A/B test and store the test statistics in an array called differences

# This cell might take a couple seconds to run

differences = make_array()

​

for i in np.arange(5000):

    test = simulate_and_test_statistic(change_in_death_rates,'Death Penalty','Murder Rate')

    differences=np.append(differences,test)

                                                 

differences

array([-0.19772722, -1.50227272,  0.71136359, ..., -1.25227276,
       -1.08863626, -1.40681816])

Run the cell below to view a histogram of your simulated test statistics plotted with your observed test statistic

Table().with_column('Difference Between Group Means', differences).hist()

plt.plot([observed_difference, observed_difference], [0, 0.5], color='red', lw=2);

Question 3.12: Find the p-value for your test and assign it to empirical_P

empirical_P = np.sum(differences>=observed_difference) /5000

empirical_P

0.26740000000000003

Question 3.13: Using a 5% P-value cutoff, draw a conclusion about the null and alternative hypotheses. Describe your findings using simple, non-technical language. What does your analysis tell you about murder rates after the death penalty was suspended? What can you claim about causation from your statistical analysis?

We fail to reject the null hypothesis since p-value is larger than 5%. The data is consistent with null hypothesis, so we can't say anything about causation as murder tates after the death penalty was suspended did not change a lot comparing to before.

You're done! Congratulations.
