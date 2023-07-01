---
share: true
---


# RANSAC (RANdom SAmple Consensus)

> Each iteration performs the following steps:
> 
1. Select `min_samples` random samples from the original data and check whether the set of data is valid (see `is_data_valid`).
2. Fit a model to the random subset (`base_estimator.fit`) and check whether the estimated model is valid (see `is_model_valid`).
3. Classify all data as inliers or outliers by calculating the residuals to the estimated model (`base_estimator.predict(X) - y`) - all data samples with absolute residuals smaller than the `residual_threshold` are considered as inliers.
4. Save fitted model as best model if number of inlier samples is maximal. In case the current estimated model has the same number of inliers, it is only considered as the best model if it has better score.

核心思想是用sample的部分数据来避免outlier的影响。通过inlyer最多来判断哪个模型更好（因为outlier总是少数的）。`residual_threshold` 默认是median absolute deviation，也可以手工指定

# Theil-Sen estimator: generalized-median-based estimator

> The TheilSenRegressor estimator uses a generalization of the median in multiple dimensions. It is thus robust to multivariate outliers. Note however that the robustness of the estimator decreases quickly with the dimensionality of the problem. It loses its robustness properties and becomes no better than an ordinary least squares in high dimension.
> 

基本思路是最小均方误差一致，只是会基于median而不是mean，所以对outlier更robust

# Huber Regression

和前两个的不同在于，并不会忽略outlier，而是减小outlier的权重

$$
\min_{w, \sigma} {\sum_{i=1}^n\left(\sigma + H_{\epsilon}\left(\frac{X_{i}w - y_{i}}{\sigma}\right)\sigma\right) + \alpha {||w||_2}^2}
$$

$$
H_{\epsilon}(z) = \begin{cases}
       z^2, & \text {if } |z| < \epsilon, \\
       2\epsilon|z| - \epsilon^2, & \text{otherwise}
\end{cases}
$$