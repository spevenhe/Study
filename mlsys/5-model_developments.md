# Model Development and Training	
## Evaluating ML Models	
### Six tips for model selection	
1. Avoid the state-of-the-art trap

2. Start with the simplest models

3. Avoid human biases in selecting models
   
4. Evaluate good performance now vs. good performance later

5. Evaluate trade-offs

6. Understand your model’s assumptions

## Ensembles	
模型合并，使用多个模型
![image](https://github.com/spevenhe/Study/assets/42630862/81f064ab-3646-4d65-bcda-e7422f6f5dff)


### Bagging	
![image](https://github.com/spevenhe/Study/assets/42630862/4d742e96-18df-41c2-a4b4-961b21f4c021)


### Boosting	

![image](https://github.com/spevenhe/Study/assets/42630862/7d25c07a-5b38-41e4-b31f-f0b5a7702799)


### Stacking	
![image](https://github.com/spevenhe/Study/assets/42630862/12fe2a04-efe1-4d96-ae49-fe2d3dc9a50b)



## Experiment Tracking and Versioning
The process of tracking the progress and results of an experiment is called experiment tracking. 

The process of logging all the details of an experiment for the purpose of possibly recreating it later or comparing it with other experiments is called versioning. These

### Experiment tracking



### Versioning



## AutoML
**There’s a joke that a good ML researcher is someone who will automate themselves out of job, designing an AI algorithm intelligent enough to design itself**
** Google intended on replacing ML expertise with 100 times more computational power, introducing AutoML to the excitement and horror of the community**


### Soft AutoML: Hyperparameter Tuning
scikit-learn with auto-sklearn, TensorFlow with Keras Tuner, Ray with Tune


### Hard AutoML: Architecture search and learned optimizer

![image](https://github.com/spevenhe/Study/assets/42630862/4a559cb6-8d65-49a5-adcd-0fb873643bf6)


https://zhuanlan.zhihu.com/p/266939342#:~:text=%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E7%94%A8%E5%A4%A7%E9%87%8F%E7%9A%84,%E5%8F%82%E6%95%B0%EF%BC%88%E5%A6%82%E5%AD%A6%E4%B9%A0%E7%8E%87%EF%BC%8Cbatch

# Distributed Training

## Data Parallelism
做法：
![image](https://github.com/spevenhe/Study/assets/42630862/c0e31aea-da02-4761-b92c-7c1038c5ebe2)

梯度更新有两种方式：
![image](https://github.com/spevenhe/Study/assets/42630862/6c51a076-ff58-4e25-a970-dbb3a4f5b8f0)
How to aggregate gradient updates?

Synchronous: have to wait for stragglers

Asynch: gradients become stale



## Model Parallelism
![image](https://github.com/spevenhe/Study/assets/42630862/0083ecae-c4b0-4937-8ff5-66fb6c73f20d)



![image](https://github.com/spevenhe/Study/assets/42630862/2ebde628-6c46-463e-a1c5-11ae3cf08c53)

![image](https://github.com/spevenhe/Study/assets/42630862/0b6669f2-9054-4e72-afd5-6282507824bb)

![image](https://github.com/spevenhe/Study/assets/42630862/954640b1-cf10-4d3a-b9e5-0fe5bf97882e)

