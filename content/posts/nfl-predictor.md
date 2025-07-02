---
title: "Predicting the winner of Super Bowl LIX"
date: 2025-02-09
summary: "Python|Pandas|Tensorflow|scikitlearn"
---

> **TL;DR**- built a cluster of Machine Learning models from scratch that successfully predicted the winner of Super Bowl LIX by pre-processing the openly sourced data, using 10s of attributes for training, validating the models' performance, and finally writing an actual predictor that used data such as field conditions, past record, and more. Since there were many models to choose from, the efficiency of each model used was also analyzed based on this use case. Major tools used- Python, Pandas, NumPy, scikitlearn, Tensorflow

--- 
## Overview

Building on my skills in Machine Learning and Data Science, I started this project in January of this year to predict the results of the conference finals of the last NFL season. Not only did I correctly predict those winners, my models also accurately chose the Philadelphia Eagles as the likely winners of Super Bowl LIX.

## Data Collection and Preprocessing

I used over 35 years of open source data for this project. The 40+ attributes in the dataset were a mix of categorical and numerical data which made data handling more tedious than dealing with simply either just categorical or just numerical data. I used these attributes to build various machine learning models such as Random Forests, Bayesian Networks, Regression models, Random Forests, XGBoost, and Neural Networks. Obviously not all these models were ideal for this use case but the reasoning for building them is discussed later in this article. 

## Data Handling

After this initial screening of data I dealt with missing values. There were some missing data points and in my initial setup I dropped the missing rows. This is not a good approach since the dataset is big enough that I would have lost valuable information. In my subsequent versions of the AI models I imputed these values. For example, if the temperature of the venue was missing, I used linear interpolation which is more sophisticated than a mean value. I also dropped some columns that had the potential of causing more bias without introducing much necessary information. For example, the dataset had a column for QB name and another for QB I.D.. Both columns represent the same thing so having 2 columns was rather unnecessary for my analysis which made me drop one. Next, since models like XGBoost and Linear Regression require numerical data I used one-hot encoding to encode the categorical variables. Then I used label encoding as I was trying to predict the result which could either be a win or a loss. I left room for OT as well (labeled as a tie).

```py
oh_encoder = OneHotEncoder(handle_unknown='ignore', sparse_output=False)
label_encoder = LabelEncoder()

X_categorical = oh_encoder.fit_transform(df_v1[categorical_columns])

X = np.hstack([X_categorical, df_v1[numerical_columns]])
y = label_encoder.fit_transform(df_v1[target])  # Encode target as integers
```

```py
input_df = pd.DataFrame(sample_data)

encoded_input = oh_encoder.transform(input_df[categorical_columns])

final_input = np.hstack([encoded_input, input_df[numerical_columns].values])

prediction = model.predict(final_input)
predicted_class = prediction[0]
predicted_label = label_encoder.inverse_transform([predicted_class])[0]

if predicted_label > 0:
    return f"{home_team} wins"
elif predicted_label < 0:
    return f"{away_team} wins"
else:
    return "Tie game expected"
```

## Models and Training

For the actual training of the models I tuned the hyperparameters based on the models' performance. For the random forest I obviously tested on the accuracy since it is a classifier but for regression based models like neural networks I had to test on metrics like precision, recall, MSE, etc. Nonetheless, once satisfied, I created a function to actually make the prediction. On top of the obvious, i.e. team names and quarterbacks, this function used attributes such as home/away teams, weather (which I got from the forecast), game type (season, playoff, Super Bowl), surface, and more. The prediction was a number which represented how likely the model predicts it is that the home team would win. Value > 0 meant the home team wins. < 0 meant the away team wins, and = 0 meant OT. One obvious "gotcha" here is that in the Super Bowl there is no home team and away team. I went about this by the actual game's announcement of what team "looks like" the home team (the scoreboard was shown as KC-PHI so I used PHI as home team). Even though the game type was set as Super Bowl, this approach may not be optimal.

<!-- ![nfl_code](/images/nfl_code.png) -->

```py
result = predict_winner_rf(
    away_team="PHI",
    home_team="KC",
    season=2025,
    game_type="DIV",
    roof="Open",
    surface="Grass",
    div_game=1,
    location="Neutral",
    model=rf_model,
    oh_encoder=oh_encoder
)
```

## Results and Predictions

Random Forest was my best performing categorical model and best overall performing model with peak accuracy of about 71%. This may not look like a high number but given the size of the dataset and complexity of the model this was a solid model accuracy. Best performing regression model was Neural Network with peak precision of 0.8. In the end the Random Forest predicted both conference winners correctly while the Neural Network predicted both wrong. I improved both and both predicted the Eagles to win the Super Bowl. The random forest felt more robust and thus more reliable than the neural network, and creating all other models such as a naive bayes was a good exercise in practically seeing the results of poor model selection.

While my models had the right winners the "likelihood" was more than a coin flip. This was not surprising due to the nature of the sport but making my own analysis based on these results in what drove me to create more similar projects for other sports.


## Updates

9:15 PM - both the predictions made by the random forest were correct.

2/9/2025 - the RNN continues to show higher accuracy peaking at about 68 with an updated dataset from the conference finals results. I am now using more attributes such as temperature, wind, location, and even the referee for the analysis. Random forest accuracy increased to as good as a coin flip and predicts the Chiefs to win. The RNN predicts to Eagles to win by 1 point. At this stage while the neural network has a better accuracy the random forest feels more robust and reliable. That said, I expect the Eagles to win today.

8:22 PM - correct prediction. Eagles won the Super Bowl.

<!-- ![nfl_prediction](/images/nfl_github.png) -->