# Team Members
- [David Cherney](https://www.linkedin.com/in/dmcherney/)
- [Shania Thomas](https://www.linkedin.com/in/shania-thomas-atx22/)
- [Thomas Prich](https://www.linkedin.com/in/thomas-prich/)
- [Afolabi Cardoso](https://www.linkedin.com/in/afolabi-cardoso/)

---
## Content

[Problem Statement](#Problem-Statement) | [Data](#DATA) | [Methodology](#Methodology) | [Conclusions and Recommendations](#Conclusions-and-Recommendations)

---

##  Problem Statement

Which class of models provide the best next day predictions of COVID case load for a state population, dynamics specific models or generic times series models?

## Background
<a href="https://en.wikipedia.org/wiki/Autoregressive_integrated_moving_average">ARMIMA models</a> are generic tools for time series prediction that take into account past trends in a variety of ways. <a href="https://en.wikipedia.org/wiki/Compartmental_models_in_epidemiology#The_SIR_model_without_vital_dynamics">SIR models</a> by contrast, aim to model the dyanamics of spead of disease in a finite population. 
We want to determine if SIR or ARIMA is a better predictor of new COVID-19 infections. 

---
## DATA

[United States COVID-19 Cases and Deaths by State over Time](https://data.cdc.gov/Case-Surveillance/United-States-COVID-19-Cases-and-Deaths-by-State-o/9mfq-cb36)


[Population data from census.gov](https://www.census.gov/data/datasets/time-series/demo/popest/2010s-state-total.html#par_textimage_1873399417)

---
## Methodology

 
See <a href="https://github.com/davidcherney/COVID-Infection-Predictions-from-SIR-and-ARIMA/blob/master/Intro_to_SIR_math.pdf">Intro_to_SIR_math.pdf</a> for an introduction to SIR models. 

---
### Data Gathering

Using the [requests](https://docs.python-requests.org/en/latest/#) python Library, we fetched 
- the total number of reported COVID-19 cases in all states from Jan 23, 2020 to Dec 29, 2021
- the total number of vaccinations administered from the day the vaccine was made public. 

This process can be seen in the notebook titled [01 data gathering and cleaning](http://localhost:8890/lab/tree/code/01%20data%20gathering%20and%20cleaning.ipynb).

>>>>>>> 

---

### Feature Engineering

- **`I_actual`** We approximate the number of people with active Covid cases on each day `I` as follows: 
    - The data from [data.cdc.gov](https://data.cdc.gov/Case-Surveillance/United-States-COVID-19-Cases-and-Deaths-by-State-o/9mfq-cb36) had data on total number of people infected to date, a cumulative sum. We used the `.diff(14)`  method from pandas on the column titled `tot_cases` and assigned the values to a `I_actual` column. We used `.diff(14)` to under the assumption that COVID cases last 14 days as described in the file Intro_to_SIR_math.pdf.

- Using the `to_datetime` Pandas method, we converted the submission_date column into date time format and set it as the index

- Because the SIR model needs the numbers currently infected `I` and the number of corrently susceptable `S`, the currently susceptable feature `S` was approximated via 
    - an estimate of the fraction of the population that was initially susceptable, and 
    - a daily of subtracting the number of new cases plus and the numbe of new vaccinations.


From the features `I_actual`, `S` and `V` for each state on each day, we ran SIR one day predictions to generate 
- the ARIMA predicted number of infections `I_arima` in each state on each day,
- the prediction `I_SIR` for each day in each state, 
- the prediction of number of people susceptable `S_SIR` in each state.

The predictions of number of infections in each state via both SIR and ARIMA are collected the file shabang.csv within the data folder.


Further, from that data we generated
- the binary feature `H-I` for each day in each state indicating 1 if `S` was below the herd immunity threshold on that day in that state. 

---
### Vizualizations
Click [here](https://public.tableau.com/app/profile/thomas.prich/viz/Magic8BallsGroupProject/Dashboard1) to view our tableau dashboard. 

---
## Conclusions and Recommendations

In our comparison of SIR and ARIMA models, 
- SIR consistently overshoots,
    - possibly describing worst case scenarios
- ARIMA next day predictions are more accurate 
    - but beyond next day predictions level out to the mean of the previous few days quickly.

A possible reasons why SIR consistently overpredicts is because it does take take into account changes in behavior such as

- Social distancing
- Remote work/school
- Masking
- Hand washing
- Reduced travel
- Closure of public places 

and other COVID-19 related precautions. These precautions very likely prevented the U.S. from reaching the infection numbers predicted by SIR. 

Another noteworthy aspect of the modeling here is that SIR also does not take into account how far apart populations are. For example, cities in rural parts of the country are less dense and therefore will have slower spread statewide comparitive to cities and states such as New York.

SIR also does not account for variants in the virus which have different rates of transmission (R0).

Based on this, SIR isn't the best for actual predictions. However, it is appropriate for predicting worst case scenarios. 

Next steps would include testing SIR models with times steps smaller than one day; the differential equations are meant to be continuous in time, and we discretized to one day time steps. This may be systematically creating over-predictions. Another next step is to attempt to make the SIR parameters time dependent to account for changes in behaviors. This would introduce further non-linearity into the difference equations, possibly leading to less stability, but with greater agreement with intuition that society responded to the pandemic with cautionary action. 

