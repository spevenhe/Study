# Data Distribution Shifts

## Types of Data Distribution Shifts	14
  ### Covariate Shift	15
Statistics: a covariate is an independent variable that can influence the outcome of a given statistical trial.
Supervised ML: input features are covariates
Input distribution changes, but for a given input, output is the same

### Label Shift	17
Output distribution changes but for a given output, input distribution stays the same.


### Concept Drift	18
Concept drift, also known as posterior shift, is when the input distribution remains the same but the conditional distribution of the output given an input changes. You can think of this as “same input, different output”. 




## General Data Distribution Shifts	18
![image](https://github.com/spevenhe/Study/assets/42630862/1dd08956-76cf-49da-a0e7-36cfdfe1da47)


## Handling Data Distribution Shifts	19

### 大数据analysis介入
Detecting Data Distribution Shifts	20
Statistical methods	20
Time scale windows for detecting shifts	22





![image](https://github.com/spevenhe/Study/assets/42630862/ab14610d-fe8f-4c70-966d-3e0c1285b5e3)


Addressing Data Distribution Shifts

Train model using a massive dataset

Retrain model with new data from new distribution

## Monitoring and Observability
![image](https://github.com/spevenhe/Study/assets/42630862/ce6fa688-b287-469a-b390-9673ec3e4c33)

![image](https://github.com/spevenhe/Study/assets/42630862/7e60b96c-e246-4d4c-ac16-7aa9bee6a343)

### 1. accuracy-related metrics

### 2. predictions

### 3. features




