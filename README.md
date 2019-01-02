# Yi-Hua-ds501
# Music Box Capstone Project- Yi Hua
## Introduction

The music box data features large number of log files that contain the events of users created with different songs. In this project, I mainly addressed two goals: to predict churn of a user through the user's behavior data and to build a recommender system that recommends the appropriate song to a user, based on his/her history data. In both missions, feature engineering is critical yet challenging due to the size of the data. Throughout the project, apart from down-sampling the data to 10%, I used PySpark as the tool to generate features as well as for building the recommendation system. 

The best performance of churn rate prediction is through XGBoost model with 0.9119 test AUC. The RMSE of the recommendation system is 0.19, based on a continuous rating from 0 to 1.

## ETL

I down-sampled by the user id down to 10% of the original and concatenate all user events separately. I then used PySpark to generate the following features:
* Label: Use 2017-03-30 ~ 2017-04-28 data as the feature time window and check each user's activity during 2017-03-30 ~ 2017-04-28. If a user is active during the second period, he/she is labeled 1, otherwise 0
    * label
* Frequency features: 
  For each event (Play/Search/Download), record the number of records for the last 1, 3, 7, 14 and 30 days as the      frequency feature. 
    * Play: freq_P_last_1, freq_P_last_3, freq_P_last_7, freq_P_last_14:, freq_P_last_30
    * Search: freq_S_last_1, freq_S_last_3, freq_S_last_7, freq_S_last_14:, freq_S_last_30
    * Download: freq_D_last_1, freq_D_last_3, freq_D_last_7, freq_D_last_14:, freq_D_last_30
* Recency features: Using the end date of the feature time window (2017-04-28) as the snapshot day, count the number of days from last event to the sanpshot day. This feature is generated for each event.
    * Play : time_to_last_P
    * Search : time_to_last_S
    * Download : time_to_last_D
* Profile feature: 
    * device_type: 1 if device is iPhone, 2 if otherwise.
* Total play time features: For each Play event, record the total play time for the last 1, 3, 7, 14 and 30 days
    * playtime__last_1, playtime__last_3, playtime__last_7, playtime__last_14, playtime__last_30
* Favorite songs features: For each Play event, record the number of songs played more than 80% of the song length for the last 1, 3, 7, 14 and 30 days
    * fav_songs__last_1, fav_songs__last_3, fav_songs__last_7, fav_songs__last_14, fav_songs__last_30
* Acceleration features: Derived features from the frequency features
    * play_1d_over_play_7d, down_1d_over_play_1d,  play_1d_over_search_1d, down_1d_over_play_7d, play_1d_over_search_7d,



## Churn Rate Prediction

For churn rate prediction, I used sklearn and tried with logistic regression, random forest, GBDT, neural network, and XGBoost models. The XGBoost, GBDT and random forest model all have very high test AUC that are above 91%. The random forest model give the highest test performance of 0.913710 with a slight overfit (training set AUC : 0.954208). The XGBoost model also gives a very good test performance of 0.911977, but with less overfit (training set AUC: 0.915801).

## Recommender System

For recommender system, I used the play data and generate the ratings as following:  

      ratings(i,j) = (total play time of user i to song j)/ (user i's longest playing time of all songs)  
      
The ratings is a continous rating between 0 and 1.
With spark ALS, I fit the matrix factorization model and obtained RMSE = 0.1998681 on the test set. 

## Discussions

The ratings of the recommender system here is generated solely from the play data. This system might work even better if I could combine the play data with the search and download data and generate a single rating metric that combines all three events. Due to the limitation of time and resources, I wasn't able to fine tune the hyperparameters of the recommender system. 
