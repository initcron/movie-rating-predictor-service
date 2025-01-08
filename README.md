# movie-rating-predictor-service

## Overview

This project is a serverless machine learning system that predicts IMDb ratings for movies using a dynamic dataset from Kaggle. The system comprises four main pipelines:

1. `1_historical_data`: Backfills the feature store with historical data.
2. `2_feature_pipeline`: Updates the feature store daily with new data.
3. `3_training_pipeline`: Trains the machine learning model.
4. `4_inference_pipeline`: Uses the trained model to predict ratings for new movies.

To monitor predictions, a user interface (UI) was developed. Users can explore different movies, view their predicted ratings alongside their posters, and click on a movie to be redirected to its IMDb page for the actual rating.

This project leverages **Hopsworks** for the feature store and **GitHub Actions** for automating the daily workflows.

![image](imgs/img1.png)

---

## Dataset

The dataset used is from Kaggle, containing the complete movie list from The Movie Database (TMDB) along with IMDb information. It includes 28 columns, such as:

- `vote_average`, `vote_count`, `status`, `release_date`, `revenue`, `runtime`, `budget`
- `imdb_rating`, `imdb_votes`, `original_language`, `overview`, `popularity`, `tagline`
- `genres`, `production_companies`, `production_countries`, `cast`, `producers`, `directors`, `writers`

The dataset is updated daily, ensuring its dynamic nature.

---

## Methodology

### Historical Backfilling

The `1_historical_data` pipeline filters movies with a `status` of `"Released"`, as movies without IMDb ratings cannot be used as targets for training. Data cleaning was also performed where columns with missing data were dropped.

### Feature Engineering

The following steps were taken during feature engineering:

- Removed features: `vote_average` and `popularity`, as they are proxies for movie ratings. Similarly, `vote_count` (related to `vote_average`) was also excluded.
- Converted `release_date` to `release_year` to group movies by production year.
- Created new features from `producers`, `cast`, and `production_companies` by selecting the first value in each cell. These were label-encoded into `first_producer`, `first_actor`, and `first_company`.
- One-hot encoded the `genres` column and uploaded it to a new feature group. This was later merged with the initial feature group during training.
- Evaluated the inclusion of the `spoken_language` feature. Results showed improved model performance (lower MSE, higher R²) when excluding this feature.

Final features used for training included:

- `budget`, `runtime`, `release_year`, `imdb_votes`
- `first_producer`, `first_actor`, `first_company`
- One-hot encoded `genres` (19 unique genres)

This resulted in a total of **27 features**.

### Model Selection

The following models were evaluated:

- **XGB Regressor** (Best-performing model)
- Random Forest Regression
- Linear Regression
- SVR (Support Vector Regressor)
- Decision Tree Regressor

For inference, predictions were also made for movies with a `status` of `"Post Production"`. For these movies, the `imdb_votes` feature was estimated as `400`.

The UI displays predictions and actual ratings for 20 newly updated movies.

---

## Results

- **Best Model**: XGB Regressor
- **Performance Metrics**:
  - **MSE**: 0.51
  - **R²**: 0.60
- **Runner-up**: Random Forest Regressor

This setup balances simple and complex models, ensuring suitability across varying dataset sizes.

---

## How to Run the Code

1. Create a profile on [Hopsworks.ai](https://www.hopsworks.ai/) and generate an API key.
2. Set up a GitHub account to automate workflows.
3. Clone this repository:
   ```bash
   git clone https://github.com/your-repo/movie-rating-predictor-service.git
   cd movie-rating-predictor-service
