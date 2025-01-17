# RUF: Updating Time-Series Predictions with Stream Data

## Citation

If you use this code in your research, please cite the following paper: [Xu, Z., & Leung, J. Y. (2023). A novel formulation of RNN-based neural network with real-time updating–An application for dynamic hydraulic fractured shale gas production forecasting. Geoenergy Science and Engineering, 212491.](https://doi.org/10.1016/j.geoen.2023.212491)

## Background

Forecasting production for a new well, especially without any production history, is a crucial yet challenging task. Many existing methods rely on static properties for predictions, which may be inaccurate. As production data accumulates, we can refine our predictions and increase confidence levels. We propose a method called Recursively Updated Forecasting (RUF) that leverages a continuous production data stream to enhance prediction accuracy.

![image](https://github.com/ziming-zx/RUF/assets/55851734/613936ae-0515-483c-b5ce-c3a6bde7453f)


## Methodology

Given that the predicted time steps are not excessively long, our hypothesis involves incorporating the entire time series during training, eliminating the need for data segmentation. This leads to more efficient and stable training processes. The `delay dimension` or `sliding window` required for the model becomes the `total sequence length -1`, with a `delay time` of `1`. The input and output structure, which includes static features, operational data, and production data, is illustrated in the table.

However, a potential issue arises with this formulation, particularly for new wells where production data is unknown in the input. This may challenge the model's predictions. To address this limitation, we propose a novel prediction framework that supports this architecture, called "recursively updated forecasting."

![image](https://github.com/ziming-zx/RUF/assets/55851734/7481cb41-f9db-4412-893c-04193bbe346d)

The RUF process is outlined in the diagram, using an example of an LSTM network. During prediction, the process begins by using an initializer to predict the first value of the production data, denoted as `U1`. Subsequently, the sequence is padded with zeros for the remaining time steps. The padded sequence is then input into the network model, and the first predicted value, `Y1'`, replaces the zero at `U2`. This process is repeated until all predicted values are generated. Additionally, if certain production history is known, it can be incorporated into the process, allowing for the prediction to start from the middle of the process.

![image](https://github.com/ziming-zx/RUF/assets/55851734/11781fa0-8c84-4551-bc79-149b03271e15)


The underlying theorem supporting this approach is that, for a single-directional RNN, the current output is only dependent on the current and previous inputs and the previous hidden states. Future values do not influence the current output. As a result, we can pad the future values with zeros, and no masking is required during both training and prediction. It's essential to note that the RUF method is not suitable for bi-directional RNNs and encoder-decoders, as their structures consider the entire sequence and thus impact the current prediction.

The initializer employed in the RUF can be any regression model. In this study, we use Radial Basis Function Networks (RBFN) due to their ability to be trained quickly and deliver accurate predictions.

## Training Data

### Dataset 1

The model is initially validated on synthetic data, generated through coupled flow-geomechanical reservoir simulation. This process includes modeling the main fracture using local grid refinement and representing secondary fractures using a discrete fracture network, which is then upscaled into the DPDK model for flow simulation. To account for geomechanics, we incorporate the Bardon-Bandis fracture permeability model into the simulation. For data generation, we conduct 1000 flow simulations. The density distribution of the variables is depicted in the diagram, and the ranges of these variables are outlined in the table.

![image](https://github.com/ziming-zx/RUF/assets/55851734/c91a99f7-e5d7-4eca-bf0d-3072ee7ff46e)

### Dataset 2

We then apply the model to predict Central Montney Shale. For prediction, we extract 39 features, and out of these, 26 out of 99 wells have completed the required features and production history. To simplify the problem, we focus solely on the peak rate and its occurrence time. After that, all time series data are aligned to the same start, and we assume they are independent. Then, 24 out of 99 wells are used for training, and 200 wells are used for testing.

![image](https://github.com/ziming-zx/RUF/assets/55851734/f951962f-4971-479a-838f-63a988ca3b0a)


## Results

### Dataset 1 (Code is provided)

The prediction results of the initializer are displayed in the figure. The initializer exhibits accurate predictions for the peak gas and peak water rates. Following this, we integrate the first value into the RUF, enabling us to generate the entire production profile. The blue cross in the figure denotes the initializer's prediction. We train five models, and their predictions are depicted with different colors. As more data becomes available, we observe the model successfully updating and incorporating the ongoing production data. The prediction results from all five models closely align, showing a high degree of consistency. Additionally, the model can capture the sudden rate change caused by shut-ins.

![image](https://github.com/ziming-zx/RUF/assets/55851734/5427c6fc-96f4-4f86-89ab-9b3c8c8371b0)


### Dataset 2

Due to increased noise in the field data compared to synthetic data, the initializer shows slightly reduced accuracy. Here, we represent test samples closest to the P10, P50, and P90 RMSE values. It is observed that the model's performance is sensitive to the initial guess. However, when it incorporates more production data, the GRU-RUF model exhibits improved prediction accuracy.

![image](https://github.com/ziming-zx/RUF/assets/55851734/df099c62-4b13-4c1c-a422-b0b0002f5391)


