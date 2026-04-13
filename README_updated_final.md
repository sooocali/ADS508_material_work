# Netflix Rating Prediction and Recommendation

## Overview
This project builds a cloud-based recommendation workflow in AWS using the Netflix Prize dataset. Our hypothetical company, **Vector Industries**, helps streaming platforms improve user engagement through better personalization. The goal of this project is to predict how a user would rate a movie and use those predicted ratings to support more relevant movie recommendations.

## Business Problem
Streaming platforms depend on personalization to keep users engaged. When users are shown content that matches their interests, they are more likely to continue watching, interact with the platform more often, and remain subscribed. Our project focuses on designing a scalable workflow that predicts likely user ratings for unseen movies so platforms can rank content more effectively and improve content discovery.

## Dataset
We used the **Netflix Prize dataset**, which contains over 100 million historical movie ratings from anonymous users along with movie metadata. The dataset documentation included in the project references the original Netflix Prize website and can be downloaded here:
https://www.kaggle.com/datasets/netflix-inc/netflix-prize-data/data

Main raw files used in this project were:

- `combined_data_1.txt`
- `combined_data_2.txt`
- `combined_data_3.txt`
- `combined_data_4.txt`
- `movie_titles.csv`
- `probe.txt`
- `qualifying.txt`
- `README`

For this project, the working sample used about **400,000 rows** from the combined rating files for faster exploration and model development.

## AWS Services Used
This project used multiple AWS services in addition to SageMaker:

- **Amazon S3** for storing raw and prepared data
- **Amazon SageMaker Studio** for notebooks, preprocessing, and experimentation
- **Amazon Athena** for SQL-based validation and exploration of prepared data
- **SageMaker JumpStart** for the initial baseline XGBoost benchmark
- **SageMaker built-in Factorization Machines** for the recommendation-oriented comparison model

## Repository Contents
The main notebooks used in this project are:

- `dataexploring.ipynb`  
  Ingests raw Netflix files, parses the semi-structured text data, performs cleaning, and explores rating patterns.

- `dataprep.ipynb`  
  Uploads prepared data to S3, sets up Athena, validates row counts and distributions, and creates the time-based train/validation/test split. The prepared dataset was uploaded to S3 and made available for downstream model training workflows. After this preparation stage, SageMaker JumpStart was used to run the initial baseline XGBoost experiment using the prepared S3 data location.

- `factorization_model.ipynb`  
  Trains the SageMaker Factorization Machines model using RecordIO-Protobuf formatted data.

- `model_evaluations.ipynb`  
  Contains the earlier model comparison workflow used during initial testing.

- `model_evaluations_final.ipynb`  
  Contains the final updated model comparison and the final test-set results.  
  **Note:** if this notebook is not yet pushed to `main`, it should replace or be added alongside the older evaluation notebook so the repository matches the final report.

## Project Workflow
1. Upload the raw Netflix data into Amazon S3.
2. Parse the raw `combined_data` text files into structured user-movie-rating rows.
3. Join movie titles and release years to the ratings data.
4. Explore rating patterns, movie popularity, user activity, and time distribution.
5. Store the prepared dataset in S3 and validate it with Athena.
6. Split the data using a **time-based 80/10/10 split** for training, validation, and testing.
7. Use the prepared S3 data files to run an initial baseline XGBoost experiment in **SageMaker JumpStart**.
8. Train and compare recommendation-oriented models, including **NMF / Matrix Factorization** and **Factorization Machines**.
9. Evaluate all models using **RMSE** and **MAE**.

## Data Preparation Summary
To prepare the data for modeling, we:

- Parsed raw Netflix files into structured rows
- Standardized column names
- Converted rating dates into usable time values
- Removed duplicates and checked for malformed rows
- Joined movie metadata such as title and release year
- Stored prepared files in Amazon S3
- Validated row counts and patterns in Athena
- Created train, validation, and test files

We used a **time-based split** because a recommendation system should learn from earlier user behavior and be tested on later behavior. This is more realistic than a random split, which mixes older and newer ratings together.

## Features and Target
The main structured training dataset included:

- `user_id`
- `movie_id`
- `rating`
- `year`
- `title`
- `release_year`

The **target variable** was `rating`, since that is the value we wanted the model to predict.

For the **baseline XGBoost model**, the final numeric input features were:

- `user_id`
- `movie_id`
- `year`
- `release_year`

The raw `title` column was excluded from XGBoost because it could not be used directly in that numeric baseline without additional encoding.

## Key Findings from Exploration
Exploration showed several important patterns in the data:

- Ratings clustered mostly around **3 to 4 stars**
- Some movies were rated far more often than others
- Some users were much more active than average
- Much of the sampled data was concentrated in **2004 and 2005**
- The dataset showed signs of **popularity bias**, **sparse user-item interactions**, and **time imbalance**

These patterns influenced our modeling and evaluation choices.

## Models Compared
We compared four main approaches:

- **User-Movie Bias Model**  
  A simple benchmark based on average user and movie rating tendencies.

- **XGBoost Regressor**  
  Our baseline benchmark model for structured numeric data. It was first explored through SageMaker JumpStart as an AWS baseline experiment, and final XGBoost comparison metrics were produced in the evaluation workflow.

- **NMF / Matrix Factorization**  
  A recommendation-oriented model that tries to uncover hidden user-item preference patterns.

- **Factorization Machines**  
  A SageMaker-built recommendation model designed to learn interactions between features.

## Final Results
The final test-set results from the updated evaluation were:

- **NMF / Matrix Factorization** — RMSE: **1.2434**, MAE: **0.9846**
- **XGBoost Regressor** — RMSE: **1.2677**, MAE: **1.0007**
- **User-Movie Bias Model** — RMSE: **1.3162**, MAE: **1.0154**
- **Factorization Machines** — RMSE: **2.6911**, MAE: **2.4276**

Based on the final evaluation, **NMF / Matrix Factorization** performed best overall on the test set.  
**XGBoost** remained a strong baseline benchmark.  
**Factorization Machines** produced the weakest results in the current implementation.

## Recommendation
Our final recommended model is **NMF / Matrix Factorization** because it achieved the best predictive performance under the final time-based evaluation. This makes it the strongest current option for ranking movies by likely user preference.

At the same time, **XGBoost** was valuable as a strong baseline benchmark because it gave a reliable point of comparison and performed competitively on the same dataset.

## Limitations
This project has several important limitations:

- The Netflix dataset is sparse, meaning most users rated only a small subset of movies.
- The sampled working dataset is concentrated in later years, especially 2004 and 2005.
- Raw text fields such as movie title could not be used directly in the XGBoost baseline.
- The current project used a subset of the full dataset for faster development.
- Factorization Machines underperformed in the current implementation and would require more tuning before being deployment-ready.

## Future Work
Future improvements could include:

- Testing stronger collaborative filtering methods such as ALS
- Building hybrid recommenders that combine collaborative and content-based signals
- Adding richer movie metadata such as genre, cast, director, and plot
- Expanding feature engineering with user averages, movie averages, and popularity signals
- Applying more systematic hyperparameter tuning
- Moving toward a real-time recommendation pipeline

## How to Run

> Note: To rerun this project, users will need to substitute their own S3 bucket names and paths unless they already have permission to access the original project buckets.

1. Obtain the Netflix Prize dataset or an accessible hosted copy of the same raw files, then upload it to **your own Amazon S3 bucket**.
2. Update the bucket names and S3 paths in the notebooks so they point to your bucket instead of the original project bucket.
3. Open the notebooks in SageMaker Studio.
4. Run `dataexploring.ipynb` to unzip, parse, and explore the raw data.
5. Run `dataprep.ipynb` to validate the structured dataset, create the time-based split, and prepare files in S3 for downstream workflows.
6. Use the prepared S3 data location to run the initial XGBoost baseline in SageMaker JumpStart. Go to models in SageMaker
7. Run `factorization_model.ipynb` for the SageMaker Factorization Machines workflow.
8. Run `model_evaluations_final.ipynb` for the final comparison of all models and final metrics.

## Team
- Christopher Andra
- Caly Nguyen
- Aaron Gabriel
