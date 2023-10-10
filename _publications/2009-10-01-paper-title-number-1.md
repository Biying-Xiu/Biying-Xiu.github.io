---
title: "Python work example 1"
collection: publications
permalink: /publication/2009-10-01-paper-title-number-1
excerpt: 'This paper is about the number 1. The number 2 is left for future work.'
date: 2009-10-01
venue: 'Journal 1'
paperurl: 'http://academicpages.github.io/files/paper1.pdf'
citation: 'Your Name, You. (2009). &quot;Paper Title Number 1.&quot; <i>Journal 1</i>. 1(1).'
---
Homework 6: Testing Hypotheses

Reading:

    Testing Hypotheses

Please complete this notebook by filling in the cells provided. Before you begin, execute the following cell.

Directly sharing answers is not okay, but discussing problems with the course staff or with other students is encouraged.

For all problems that you must write our explanations and sentences for, you must provide your answer in the designated space. Moreover, throughout this homework and all future ones, please be sure to not re-assign variables throughout the notebook! For example, if you use max_temperature in your answer to one question, do not reassign it later on.

# Don't change this cell; just run it. 

​

import numpy as np

from datascience import *

​

# These lines do some fancy plotting magic.

%matplotlib inline

import matplotlib.pyplot as plots

plots.style.use('fivethirtyeight')

​

1. Spam Calls
Part 1: 585 Fun

Dr. McLean gets a lot of spam calls. An area code is defined to be a three digit number from 200-999 inclusive. In reality, many of these area codes are not in use. Of these possible numbers, there are currently 317 geographical US area codes and 18 non-geographical US area codes, for a total of 335.

Throughout these questions, you should assume that Dr. McLean's area code is 585 (Rochester NY, home of the garbage plate).

Question 1. Assuming that each US area code is just as likely as any other, what is the probability that the area code of four consecutive spam calls is 585? Assign this value to prob_585.

prob_585 = (1/335)**4

prob_585

7.940004925780556e-11

Question 2. Peter already knows that Dr. McLean's area code is 585. From googling, Peter determines that there are 792 possible prefixes (the next three digits after the area code) for any area code and randomly guesses one of those prefixes and the last 4 digits (0-9 inclusive) of Dr. McLean's phone number. What's the probability that Peter correctly guesses Dr. McLean's number, assuming that he’s equally likely to choose any prefix or digit?

Note: A phone number contains an area code and 7 additional digits, i.e. xxx-xxx-xxxx

prob_mclean_num = 1/(792*10**4)

prob_mclean_num

1.2626262626262626e-07

Dr. McLean suspects that there's a higher chance that the spammers are using his area code (585) to trick him into thinking it's someone from his area calling him. Stephanie thinks that this is not the case, and that spammers are just choosing area codes of the spam calls at random from all possible US area codes. Dr. McLean wants to test his claim using the 50 spam calls he received in the past month.

Here's a dataset of the area codes of the 50 spam calls he received in the past month.

# Just run this cell

spam = Table().read_table('spam.csv')

spam

Area Code
810
630
205
440
585
304
870
610
717
332

... (40 rows omitted)

Question 3. Define the null hypothesis and alternative hypothesis for this investigation.

Hint: Don’t forget that your null hypothesis should fully describe a probability model that we can use for simulation later.

Null hypothesis: The spammers are choosing area codes randomly from all possible US area codes, including 585.

Alternative hypothesis: The spammers are more likely to use the area code 585 than any other area code.

Question 4. Which of the following test statistics would help differentiate between the two hypotheses?

Hint: For a refresher on choosing test statistics, check out the textbook section on Test Statistics.

    The proportion of area codes that are 585 in 50 random spam calls
    The probability of getting an area code of 585 out of all the possible area codes.
    The proportion of area codes that are 585 in 50 random spam calls divided by 2
    The number of times you see the area code 585 in 50 random spam calls

Assign reasonable_test_statistics to an array of numbers corresponding to these test statistics.

reasonable_test_statistics = 1

For the rest of this question, suppose that you decide to use the number of times you see the area code 585 in 50 spam calls as your test statistic.

Question 5. Write a function called simulate that generates exactly one simulated value of your test statistic under the null hypothesis. It should take no arguments and simulate 50 area codes under the assumption each area code is sampled with equal probability. The all_area_codes table contains all US area codes. Your function should return the number of times you saw the 585 area code in those 50 random spam calls.

AreaCodes = Table.read_table('all_area_codes.csv')

AreaCodes

Area Code 	State
201 	NJ
202 	DC
203 	CT
205 	AL
206 	WA
207 	ME
208 	ID
209 	CA
210 	TX
212 	NY

... (325 rows omitted)

def simulate():

    calls=AreaCodes.sample(50)

    return calls.where('Area Code',585).num_rows

    

# Call your function to make sure it works

simulate()

0

Question 6. Generate 20,000 simulated values of the number of times you see the area code 585 in 50 random spam calls. Assign test_statistics_under_null to an array that stores the result of each of these trials.

Hint: Use the function you defined in Question 5.

test_statistics_under_null = make_array()

repetitions = 20000

for i in np.arange(repetitions):

    simulated_value = simulate()

    test_statistics_under_null=np.append(test_statistics_under_null,simulated_value)

    

test_statistics_under_null

array([ 1.,  0.,  0., ...,  0.,  1.,  1.])

Question 7. Using the results from Question 6, generate a histogram of the empirical distribution of the number of times you saw the area code 585 in your simulation. NOTE: Use the provided bins when making the histogram

bins = np.arange(0,5,1) # Use these provided bins

under_null=Table().with_column('test',test_statistics_under_null)

under_null.hist(bins=bins)

Question 8. Compute an empirical P-value for this test.

# First calculate the observed value of the test statistic from the `spam` table.

observed_val = spam.where('Area Code',585).num_rows

p_value = sum(test_statistics_under_null >= observed_val) / repetitions

p_value

0.0095999999999999992

Question 9. Suppose you use a P-value cutoff of 1%. What do you conclude from the hypothesis test? Why?

If we use a P-value cutoff of 1%, we would conclude that there is strong evidence to reject the null hypothesis. This means that it's very unlikely that the spammers are choosing area codes randomly, as the observed number of times 585 appears in the spam calls is significantly different from what you would expect under the null hypothesis. Instead, it suggests that the spammers are more likely to use the area code 585, supporting Dr. McLean's suspicion that they are trying to trick him.

Part 2: Multiple Spammers

Instead of checking if the area code is equal to his own, Dr. McLean decides to check if the area code matches the area code of one of the seven places he's been to recently, and wants to test if it's more likely to receive a spam call with an area code from any of those seven places. These are the area codes of the places he's been to recently: 585, 919, 910, 315, 336, 775, 720.
Question 10. Define the null hypothesis and alternative hypothesis for this investigation.

Reminder: Don’t forget that your null hypothesis should fully describe a probability model that we can use for simulation later.

Null Hypothesis : The probability of receiving a spam call with an area code from one of the seven places Dr. McLean has recently visited is equal to the probability of receiving a spam call with any other area code. There is no preference for spam calls from these specific area codes.

​

Alternative Hypothesis : The probability of receiving a spam call with an area code from one of the seven places Dr. McLean has recently visited is different than the probability of receiving a spam call with any other area code. There is a preference for spam calls from these specific area codes.

Suppose you decide to use the number of times you see any of the area codes of the places Dr. McLean has been to in 50 spam calls as your test statistic.

Question 11. Write a function called simulate_visited_area_codes that generates exactly one simulated value of your test statistic under the null hypothesis. It should take no arguments and return the number of times that you saw any of the area codes of the places Dr. McLean has been to in those 50 spam calls (under the assumption that the calls come from the US area codes with equal probability).

Hint: You may find the textbook section on the sample_proportions function to be useful.

model_proportions = make_array(7/800,793/800)

def simulate_visited_area_codes():

    my_prop = sample_proportions(50,model_proportions)

    return my_prop.item(0)*50

    

# Call your function to make sure it works

simulate_visited_area_codes()

1.0

Question 12. Generate 20,000 simulated values of the number of times you see any of the area codes of the places Dr. McLean has been to in 50 random spam calls. Assign test_statistics_under_null to an array that stores the result of each of these trials.

Hint: Use the function you defined in Question 11.

visited_test_statistics_under_null = make_array()

​

repetitions = 20000

for i in np.arange(repetitions):

    value = simulate_visited_area_codes()

    visited_test_statistics_under_null = np.append(visited_test_statistics_under_null,value)

visited_test_statistics_under_null

array([ 0.,  2.,  1., ...,  0.,  0.,  2.])

Question 13. Using the results from Question 12, generate a histogram of the empirical distribution of the number of times you saw any of the area codes of the places Dr. McLean has been to in your simulation. NOTE: Use the provided bins when making the histogram

bins_visited = np.arange(0,8,1) # Use these provided bins

tbl_statss = Table().with_column("Number of times visted codes",

visited_test_statistics_under_null)

tbl_statss.hist(bins=bins_visited)

Question 14. Compute an empirical P-value for this test.

# First calculate the observed value of the test statistic from the `spam` table.

visited_area_codes = make_array(781, 617, 509, 510, 212, 858, 339, 626)

visited_observed_value = 0

for x in visited_area_codes:

    for y in spam.column(0):

        if x == y:

            visited_observed_value += 1

p_value = sum(visited_test_statistics_under_null>=visited_observed_value)/ repetitions

p_value

0.07145

Question 15. Suppose you use a P-value cutoff of 0.5% (Note: that’s 0.5%, not our usual cutoff of 5%). What do you conclude from the hypothesis test? Why?

The P-value of 0.07145 is greater than the cutoff value of 0.0005. Therefore, that the nullhypothesis can not be rejected, and we can conclude that there is no preference for spam calls from these specific area codes.

Question 16. Is p_value:

    (a) the probability that the spam calls favored the visited area codes,
    (b) the probability that they didn't favor, or
    (c) neither

If you chose (c), explain what it is instead.

c: P-value is the chance of wrongly rejecting the null hypothesis

(c) because the P-value cutoff is the error tolerance, which means that we can wrongly reject the null hypothesis 0.05% of the times.Question 17. Is 0.5% (the P-value cutoff):

    (a) the probability that the spam calls favored the visited area codes,
    (b) the probability that they didn't favor, or
    (c) neither

If you chose (c), explain what it is instead.

(c) because the P-value cutoff is the error tolerance, which means that we can wrongly reject the null hypothesis 0.05%.

Question 18. Suppose you run this test for 400 different people after observing each person's last 50 spam calls. When you reject the null hypothesis for a person, you accuse the spam callers of favoring the area codes that person has visited. If the spam callers were not actually favoring area codes that people have visited, can we compute how many times we will incorrectly accuse the spam callers of favoring area codes that people have visited? If so, what is the number? Explain your answer. Assume a 0.5% P-value cutoff.

we can calculate the upper bound of times we incorrectly accuse the spam callers: (0.05/100)*4000 times = 2 times as the P-value cut-off is the percentage of times that we are prepared to incorrectly reject the null hypothesis.

Part 3: Practice with A/B Tests

Dr. McLean collects information about this month's spam calls. The table with_labels is a sampled table, where the Area Code Visited column contains either "Yes" or "No" which represents whether or not Dr. McLean has visited the location of the area code. The Picked Up column is 1 if Dr. McLean picked up and 0 if he did not pick up. Honestly, unless it's Dr. McLean's family calling, He's not answering the phone...

# Just run this cell

with_labels = Table().read_table("spam_picked_up.csv")

with_labels

Area Code Visited 	Picked Up
No 	0
Yes 	1
No 	1
Yes 	0
No 	0
No 	0
Yes 	0
No 	1
No 	1
No 	1

... (40 rows omitted)

Dr. McLean is going to perform an A/B Test to see whether or not he is more likely to pick up a call from an area code he has visited. Specifically, his null hypothesis is that there is no difference in the distribution of calls he picked up between visited and not visited area codes, with any difference due to chance. His alternative hypothesis is that there is a difference between the two categories, specifically that he thinks that he is more likely to pick up if he has visited the area code. We are going to perform a permutation test to test this. Our test statistic will be the difference in proportion of calls picked up between the area codes Dr. McLean visited and the area codes he did not visit.

Question 19. Complete the difference_in_proportion function to have it calculate this test statistic, and use it to find the observed value. The function takes in a sampled table which can be any table that has the same columns as with_labels. We'll call difference_in_proportion with the sampled table with_labels in order to find the observed difference in proportion.

def difference_in_proportion(sample):

    # Take a look at the code for `proportion_visited` and use that as a 

    # hint of what `proportions` should be assigned to

    proportions = sample.group('Area Code Visited',np.mean)

    proportion_visited = proportions.where("Area Code Visited", "Yes").column("Picked Up mean").item(0)

    proportion_not_visited = proportions.where("Area Code Visited", "No").column("Picked Up mean").item(0)

    return proportion_visited-proportion_not_visited

​

observed_diff_proportion = difference_in_proportion(with_labels)

observed_diff_proportion

0.25

Question 20. To perform a permutation test we shuffle the labels, because our null hypothesis is that the labels don't matter because the distribution of calls he picked up between visited and not visited area codes come from same underlying distribution. The labels in this case is the "Area Code Visited" column containing "Yes" and "No".

Write a function to shuffle the table and return a test statistic using the function you defined in question 19.

Hint: To shuffle labels, we sample without replacement and then replace the appropriate column with the new shuffled column.

def simulate_one_stat():

    shuffled = with_labels.sample(with_replacement=False).column('Area Code Visited')

    original_with_shuffled_labels = with_labels.select('Picked Up').with_column('Area Code Visited', shuffled)

    return difference_in_proportion(original_with_shuffled_labels)

​

one_simulated_test_stat = simulate_one_stat() 

one_simulated_test_stat

0.15808823529411764

​

Question 21. Generate 1,000 simulated test statistic values. Assign test_stats to an array that stores the result of each of these trials.

Hint: Use the function you defined in Question 20.

We also provided code that'll generate a histogram for you after generating a 1000 simulated test statistic values.

trials = 1000

test_stats = make_array()

​

for i in np.arange(trials):

    statistic = simulate_one_stat()

    test_stats = np.append(test_stats,statistic)

​

# here's code to generate a histogram of values and the red line is the observed value

Table().with_column("Simulated Proportion Difference", test_stats).hist("Simulated Proportion Difference");

plots.plot([observed_diff_proportion, observed_diff_proportion], [0, 3], color='red', lw=2);

Question 22. Compute the empirical p-value for this test, and assign it to p_value_ab.

p_value_ab = np.count_nonzero(test_stats <= observed_diff_proportion) / repetitions

p_value_ab

0.04885
