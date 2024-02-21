
# Batch Prediction vs. Online Prediction
![image](https://github.com/spevenhe/Study/assets/42630862/d31bb354-4e61-41a8-89d7-64d0851f525f)

## Batch prediction is not always cheap and not effcient in performance.

batch 跑批需要针对所有用户，而实时只针对访问的用户

## online prediction

1. A (near) real-time pipeline that can work with incoming data, extract streaming features (if needed), input them into a model, and return a prediction in near real-time. A streaming pipeline with real-time transport and a stream computation engine can help with that.

2. A fast-inference model that can generate predictions at a speed acceptable to its end users. For most consumer apps, this means milliseconds.

![image](https://github.com/spevenhe/Study/assets/42630862/d7166dd3-3a42-436a-a793-697788f4b31b)

## make model run faster

1. do inference faster
2. model smaller
3. hardware which model’s deployed on run faster


# Model Compression
1. Low-rank Factorization: 低秩矩阵分解
2. Knowledge Distillation: a small model (student) is trained to mimic a larger model or ensemble of models
3. Pruning: find parameters least useful to predictions and set them to 0.
4. Quantization: using fewer bits to represent its parameters


# ML on the Cloud and on the Edge





