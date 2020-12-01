# CS 211 Final Project Writeup
### Luke Fredrickson
#### December 1, 2020

## Problem Statement
I wanted to examine the [NYC Citywide Payroll Data (Fiscal Year)](https://data.cityofnewyork.us/City-Government/Citywide-Payroll-Data-Fiscal-Year-/k397-673e) dataset from https://data.cityofnewyork.us. The dataset is massive, so I trimmed it down to just the 2020 data, which still contains 590,210 rows (each row corresponds to one person), and data for 156 agencies throughout the city.

My goal was to try to answer clearly important questions about the data on a per-agency basis, such as the average pay, agency size, and most common work location, job title, and pay basis.

## Technical Description
As I said earlier, the original dataset including data from all fiscal years was far too large, so I trimmed it down to just the 2020 data and called it `payroll`. From there, I added a column `Total Paid`, which is the sum of `Regular Gross Paid`, `Total OT Paid`, and `Total Other Paid`.

`dp_agency_sizes()` gets the differentially private sizes of the agencies. To do this, I grouped `payroll` by `Agency Name` and stored the size of each group in a vector. I then ran that vector through `laplace_mech_vec()` to add noise, and reattached the agency names via the index, returning a dataframe containing `Agency Name` and `DP Agency Size`.

`select_agencies_by_size()` uses `dp_agency_sizes()` to create a dataframe of all agencies between a given min and max size.

`get_mean_value()`  returns a dataframe of simple, non-private mean values for a given key on a per-agency basis for given agencies.

`get_dp_mean_value()` uses the `auto_avg()` function, which itself uses `above_threshold()`, to automatically pick a clipping parameter, compute a noisy sum and noisy count, and calculate a differentially private mean. I modified the clipping parameter list, `bs`, to use `np.logspace` with 1000 values from 10^0 to 10^6 instead of a simple linear progression, because the functions were taking a very long time to run. `get_dp_mean_value()` splits the given epsilon between the number of agencies it needs to get mean values from.

`get_pay_by_agency()` aggregates the non-private means, the dp-mean, and the percent error between the two into a dataframe.

`get_max_in_col_by_agency()` uses `report_noisy_max()` to find the most common value for each column in a given set of columns on a per-agency basis. Epsilon is split between each column, and then further between each agency. The function returns a dataframe of the agencies and the most common value for each column in the provided columns.

All the following code sets parameters for epsilon and which agencies and columns to analyze. I chose to analyze agencies between 500 and 1000 employees in size, and to look at the most common `Work Location Burough`, `Title Description`, and `Pay Basis` for each, along with mean total pay.

## Results
I chose the parameters above mainly because if I did anything more the analysis would take too long to run. The mean total pay was fairly accurate (within less than 5% off) when examining fewer than 10 or so medium-sized agencies, but began to decrease as more agencies were added, especially agencies with many employees. The categorical maxes were accurate mostly because employees within an agency aren't that different from each other (lots of police officers working in Manhattan in the Police Department, big surprise).

I could have played around with using different methods of differential privacy if I wanted to get faster and more accurate results, such as Zero-Concentrated Differential Privacy, especially because these queries get run many, many times as the number of agencies and employees increases, but for simplicity's sake I stuck with the Laplace Mechanism.