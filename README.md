# Baseball_Analytics

## Data Collection Process
- This project involves data from multiple baseball leagues, each with their unique playing styles. The data for these analyses was collected from the following source:
https://www.kaggle.com/competitions/mlb-player-digital-engagement-forecasting/data?select=train.csv

## Script Structure

    ```
    ├── KBO_preprocessing
    │   └── preprocess_kbo_data.py
    ├── MLB_prprocessing
    │   ├── extract_mlb_boxscore.py
    │   ├── extract_mlb_game_data.py
    │   └── team_season_stat_gen.py
    ├── Models
    │   ├── LDA.py
    │   ├── __pycache__
    │   │   ├── preprocess_data.cpython-310.pyc
    │   │   └── preprocess_data_LDA.cpython-310.pyc
    │   ├── lasso_ridge_model.py
    │   ├── lda_feature_coefficients.csv
    │   ├── multinom_logreg.py
    │   ├── preprocess_data_LDA.py
    │   └── pythagorean_expectation_modeling.py
    ├── NPB_preprocessing
    ├── README.md
    └── datasets
        ├── KBO_datasets
        │   ├── kbo_train.csv
        │   ├── kbobattingdata.csv
        │   └── kbopitchingdata.csv
        ├── MLB_datasets
        │   ├── MLB_team_season_statistics.csv
        │   ├── MLB_yearly_boxscore
        │   │   ├── year_2018.csv
        │   │   ├── year_2019.csv
        │   │   ├── year_2020.csv
        │   │   └── year_2021.csv
        │   ├── mlb_data.csv
        │   ├── mlb_teams.csv
        │   ├── mlb_win_loss_probabilities.csv
        │   ├── pe_model_yearly_outputs
        │   │   ├── pythagorean_output_2018.csv
        │   │   ├── pythagorean_output_2019.csv
        │   │   ├── pythagorean_output_2020.csv
        │   │   └── pythagorean_output_2021.csv
        │   └── team_season_statistics.csv
        └── NPB_datasets
    
    12 directories, 29 files
    ```
    
## Extracting Yearly Team Box Score Data (mlb_collect.py)
- This script is used to process and transform the MLB data sourced from the Kaggle MLB Player Digital Engagement Forecasting competition. The raw data includes nested JSON within a CSV file, which is then extracted and transformed into a more usable format.

- Here's a breakdown of the steps:
  1. **Download Data**: The raw data can be downloaded directly from the Kaggle competition page. Make sure to download the 'train.csv' file.

  2. **Load Data**: The dataset is loaded into a pandas DataFrame from the 'train.csv' file.
     
  ```
  import pandas as pd
  import json
  df = pd.read_csv("./train.csv")
  ```
  
  3. **Extract and Transform Data**: The "games" column from the DataFrame, which contains the JSON data of interest, is isolated. The JSON string is transformed into actual JSON objects and then converted to a pandas DataFrame using the json_normalize function.
     
  ```
  teamBoxScores_data = df["games"].dropna()
  teamBoxScores_data = teamBoxScores_data.apply(lambda x: json.loads(x.replace('null', 'None')) if pd.notnull(x) else None)
  teamBoxScores_df = pd.json_normalize(teamBoxScores_nested_json)
  ```
  
  4. **Preprocessing the Data**: The "gameDate" column, which contains the dates of games, is converted to datetime format. The year from each game date is extracted and stored in a new column, "year".
     
    ```
    data = pd.read_csv('mlb_data.csv')
    data['gameDate'] = pd.to_datetime(data['gameDate'])
    data['year'] = data['gameDate'].dt.year
    ```
    
  5. **Yearly Team Box Score Data Extraction**: The data is grouped by year and by team ID to compute yearly statistics for each team. The computed yearly team stats are saved to separate CSV files for each year, which provides us with the yearly team box score data.
     
   ```
   for year in data['year'].unique():
    yearly_data = data[data['year'] == year]
    yearly_team_stats = yearly_data.groupby('teamId').sum()
    yearly_team_stats.to_csv(f'year_{year}.csv')
   ```
   
  To run this script, use the following command:
  
   ```
   python mlb_collect.py
   ```
   
   Please ensure that the input CSV file ('train.csv') is in the correct path before running the script. After the script has run, check the output CSV files to see the extracted yearly team box score data.

Remember to check and confirm that the output files ('year_{year}.csv') are in the desired output directory after running the script.

Keep in mind that the structure of your input data file ('train.csv') should conform to the structure expected by the script for it to run successfully. Check the example data provided for the expected format and structure.

##  Standardize different datasets from different leagues (team_season_stat_gen.py)

  - The function preprocess_league_data inside team_season_stat_gen.py script is used to standardize different datasets from different leagues into a consistent and structured format. This is crucial in ensuring the integrity and compatibility of the data during subsequent analysis.
  
  - The function takes as input three CSV files: the first containing data about each game (including the season, home team, away team, home team wins, and home team losses), the second containing more detailed statistics about each game, and the third containing information about the teams.
    
  ```
  df1 = pd.read_csv(df1_path)
  df2 = pd.read_csv(df2_path)
  df_team = pd.read_csv(team_info_path)
  ```

The function performs several transformations and aggregations on the data:

  - It calculates the total wins and losses for each team in each season, both for home games and away games.
    
  - It then calculates the win-loss percentage for each team in each season.
    
  - It averages the detailed statistics for each team in each season.
    
  - It merges these different dataframes into a final dataframe.
    
  - The resulting dataframe provides a complete and detailed view of the performance of each team in each season. Each row represents one team in one season and includes information about the number of wins and losses, the win-loss percentage, and other statistics averaged over all the games in that season.

  ```
  df1_home = df1.groupby(['season', 'homeId']).agg({
      'homeWins': 'sum',
      'homeLosses': 'sum'
  }).reset_index().rename(columns={'homeId': 'teamId', 'homeWins': 'wins', 'homeLosses': 'losses'})
  
  df1_away = df1.groupby(['season', 'awayId']).agg({
      'awayWins': 'sum',
      'awayLosses': 'sum'
  }).reset_index().rename(columns={'awayId': 'teamId', 'awayWins': 'wins', 'awayLosses': 'losses'})
  
  team_season_data = pd.concat([df1_home, df1_away])
  
  team_season_data = team_season_data.groupby(['season', 'teamId']).agg({'wins': 'sum', 'losses': 'sum'}).reset_index()
  ```

  - Finally, the function also renames the columns in the final dataframe to match a standard format. This makes the data easier to work with, as the column names are consistent across different leagues and datasets.

  ```
  column_mapping = {...}
  final_df.rename(columns=column_mapping, inplace=True)
  final_df = final_df.drop(columns='id')
  ```

  - By preparing the data in this way, the preprocess_league_data function makes it easier to analyze the performance of different teams across different seasons and even across different leagues.


## Linear Discriminant Analysis (LDA)

  - In this section, I utilize the Linear Discriminant Analysis (LDA) method to perform a linear transformation on the features of our dataset. The primary goal of this approach is to increase class separability by reducing dimensionality, which will further help us in creating a model that can accurately classify the data.
  - Class Methods
  - The class contains several methods dedicated to specific steps in the LDA process. Here's a brief description of each method:

    - fit_lda: This function takes the train data, standardizes it, and then fits an LDA model. The function returns the trained LDA model and the transformed data.
    
    - plot_lda: This function plots the LDA-transformed data.
    
    - lda_coef: This function extracts the coefficients of the LDA model, which are used to select features based on the given threshold.
    
    - check_collinearity: This function checks the collinearity of features using Variance Inflation Factor (VIF).
    
    - select_features_based_on_coef: This function selects features whose absolute coefficients are greater than a given threshold.
    
    - plot_vif_data: This function plots the VIF scores of the features.
    
    - run_analysis: This function executes the entire LDA process from preprocessing the data to plotting the VIF scores and LDA coefficients. This is the main function that you'll call to run the analysis.

### Instructions
  - First, import the necessary modules and the LDA class. Then, create an instance of the LDA class, passing the dataframe and threshold as parameters. Finally, call the run_analysis method on the instance.
  
  ```
  df = pd.read_csv("team_season_statistics.csv")
  lda = LDA(df)
  lda.run_analysis()
  ```
  
  - This will preprocess the data, run the LDA, and output the analysis results, which includes plots, explained variance ratio, LDA coefficients, and the selected features based on the given threshold.

  - This process helps to transform the feature space, reduce dimensionality, and increase class separability, making it easier to develop accurate and efficient classification models.

  - For more detailed code understanding, please refer to the inline comments and the function definitions inside the class.

### Expected Output
  - The script will output the Mean Squared Error (MSE) and R-Squared (R^2) for each year in the data. Here are some example outputs you might expect:
    ```
    Pythagorean Expectation: MSE = 0.043825052872218384, R^2 = 0.04972759343921895
    Pythagorean Expectation: MSE = 0.03606235134234921, R^2 = -0.17769687170829518
    Pythagorean Expectation: MSE = 0.004137536723744783, R^2 = 0.6199216545300004
    Pythagorean Expectation: MSE = 0.0035629027629842376, R^2 = -0.37132119335757463
    ```
- Additionally, the script will create an LDA scatter plot and a vif plot to measure collinearity. An example of such a plot might look something like this:

![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/0a961a73-b4be-4af6-ad89-00c69217ac4e)
![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/0dfd41dd-83f9-4a09-8073-d14f948796dd)

## extract_mlb_game_data.py
- This script loads, parses, and transforms a dataset containing information about ba!
seball games. The source data is assumed to be in CSV format, with one of the columns containing JSON strings that encapsulate detailed game data.
  
  1. Data Loading
  The script starts by loading a CSV file into a Pandas DataFrame.
  
  2. Dataframe Initialization
  An empty DataFrame is initialized. This will be used to store the processed game data.
  
  3. Row Iteration
  The script iterates over each row in the original DataFrame, checking for any missing or malformed data in the 'games' column.
  
  4. JSON Data Parsing
  For rows with valid 'games' data, the script attempts to parse the JSON data into a Python object. It handles any potential JSON decoding errors gracefully by printing an error message and continuing with the next row.
  
  5. JSON to DataFrame Conversion
  Upon successful parsing, the script converts the Python object (derived from the JSON data) into a DataFrame. This process is known as 'flattening' the JSON structure.
  
  6. Appending to the Main DataFrame
  The new DataFrame is then appended to the main DataFrame, ensuring a continuous index across the whole dataset.
  
  7. Saving the DataFrame
  Finally, the complete DataFrame, containing the parsed and transformed game data, is saved to a new CSV file.
  
  This script is particularly useful for scenarios where game data is stored in JSON format within a CSV file, and where this data needs to be extracted, transformed, and saved in a flat, tabular format. Note that the script specifically looks for a 'games' column in the source CSV file and expects this to contain valid JSON strings.
  
## PreprocessModelData Class

The PreprocessModelData class is a Python utility that allows for the preprocessing of structured datasets (e.g., CSV files). The class provides functions for handling missing data, separating features from target variables, scaling numerical features, and splitting datasets into training and testing subsets.

  - **Initialization**
    - The class is initialized with a dictionary (impute_strategy_dict) that specifies the imputation strategy ('mean', 'median', or 'mode') for each column in the dataframe that has missing data.
  
  - **handle_missing_data method**
    - This method goes through each column in the dataframe and fills missing values according to the specified strategy.
  
  - **separate_features_target method**
    - This method separates the features and the target column in the dataframe. If the target column is 'win_loss_percentage', it creates a new column 'categorical_win_loss_prob' which categorizes 'win_loss_percentage' into 'low', 'medium', or 'high' categories.
  
  - **scale_features method**
    - This method standardizes the features in the dataframe using the StandardScaler from scikit-learn. This is often a necessary step before using many machine learning algorithms.
  
  - **split_data method**
    - This method splits the dataset into training and testing subsets. It uses scikit-learn's train_test_split function, which shuffles the dataset and splits it. The default test size is 20% of the total dataset.
  
  - The PreprocessData class can be used in combination with the extract_mlb_game_data.py script. The script extracts and transforms the game data, and the class preprocesses it for machine learning applications. The imputation of missing values, scaling of features, and splitting of data are all common steps in a machine learning pipeline, and this class provides an easy and reusable way to perform these tasks.

## pythagorean_expectation_modeling.py
### Preprocessing the Data
  1. The script starts by loading two data files: team_win_loss_probabilities.csv and teams.csv. The former is presumed to contain historical win-loss probability data for baseball teams, while the latter contains team identifiers and corresponding team names. A dictionary mapping team IDs to team names is created from the teams.csv data.
  
  ```
  team_mapping = teams.set_index('id')['name'].to_dict()
  ```
  
  2. Then, the script enters a loop that ranges from the year 2018 to 2021 (the last year is exclusive in Python's range function). For each year in the range:
  
  ```
  for year in range(2018, 2022):
  ```
  
  3. The script reads in the yearly team box score data from a CSV file named in the format year_{year}.csv (for example, year_2018.csv for 2018). This data file is presumed to contain box score data (e.g., runs scored and runs allowed) for each team for that year. It then creates a column year and sets it to the current year being processed.
  
  The box score data is grouped by team ID and year, and the total (sum) runs scored and runs allowed for each team in that year is computed.
  ```
  team_boxscore_grouped = team_boxscore.groupby(['teamId', 'year']).agg({
    'runsScored': 'sum',
    'runsPitching': 'sum',  
  }).reset_index()
  ```
  4. Finally, it merges the processed box score data with the win-loss probability data based on team name and year. If the merged data is empty (which could be due to missing win-loss probability data for that year or team), the script will again skip the rest of the current loop iteration and move to the next year.
  ```
  dataset = pd.merge(team_boxscore_grouped, win_loss_prob, left_on=['teamName', 'year'], right_on=['homeName', 'season'])
  if dataset.empty:
      print(f'Merged DataFrame is empty for year {year}. Check your merge operation.')
      continue
  ```
  
  At the end of this preprocessing step, for each year in the given range, you will have a dataset that includes total runs scored, total runs allowed, team names, and corresponding win-loss probabilities for each team. These datasets are then saved as pythagorean_output_{year}.csv.


## Pythagorean Expectation Calculation and Modeling
- The Pythagorean expectation is a formula that estimates a team's win percentage based on the number of runs it scores and allows. In baseball analytics, it's often used to evaluate a team's performance compared to their actual win percentage.

- The script computes the Pythagorean expectation for each team in the processed data and adds it as a new column, pythag_expect.
  ```
  df['pythag_expect'] = (df[runsScored] ** 2) / (df[runsScored] ** 2 + df[runsAllowed] ** 2)
  ```

- The script then prepares the data for the machine learning model. It creates a 2-D feature matrix X with runs scored and runs allowed, and a target variable y with the win-loss probability.
  ```
  X = df[[runsScored, runsAllowed]]
  y = df[win_loss_probability]
  ```

- Next, the script uses scikit-learn's PolynomialFeatures to generate polynomial and interaction features from the runs scored and runs allowed. This expanded feature matrix X_poly is then split into training and testing datasets, with 80% of the data used for training and 20% for testing.
  ```
  poly = PolynomialFeatures(degree=2)
  ```

- A linear regression model is trained using the training data. The model's performance is evaluated on the test data using the mean squared error (MSE) and the R-squared (R²) statistic, which are printed out for each year.
  ```
  model = LinearRegression()
  model.fit(X_train, y_train)
  y_pred = model.predict(X_test)
  mse_pythag = mean_squared_error(y_test, y_pred)
  r2_pythag = r2_score(y_test, y_pred)
  print(f'Pythagorean Expectation: MSE = {mse_pythag}, R^2 = {r2_pythag}')
  ```

- This process is repeated for each year in the range, resulting in a separate model and performance metrics for each year. It allows you to see how the model's performance varies from year to year and potentially spot any trends or anomalies.

## Expected Output

The Pythagorean expectation model generates an informative scatter plot that compares the actual win probability with the predicted win probability for the MLB teams over the years in scope. The plot also includes a regression line (red) that indicates the relationship between these two variables. 

![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/2a16ab56-c6cb-4e21-8f44-3366231a58d7)

Each data point in the scatter plot represents a specific team in a specific year. The x-coordinate of the point indicates the team's actual win probability, while the y-coordinate indicates the win probability predicted by the Pythagorean expectation model. The closer a point is to the regression line, the closer the model's prediction was to the actual outcome.

For a more detailed, interactive exploration of the plot, you can view the Plotly-generated HTML file [here](./Baseball_Analytics_main/figures/pythagorean_model_MLB.html).


## Feature Selection Using Lasso Regression and Ridge Regression (ridge_reg_model.py)
- This script provides a workflow for using Lasso Regression and Ridge Regression for feature selection and understanding feature importance.

### Instructions
  ```
  ridge_model, selected_features, coefficients = perform_ridge_with_lasso_selection(
          train_df,
          'win_loss_percentage', 
          test_size=0.3, 
          random_state=42
  )
  ```

  - In the above example, the dataset kbo_train.csv is loaded. The target variable is the column win_loss_percentage. The dataset is split into training and testing sets with a test size of 30%. The random state for the train-test split is set as 42.
  
  - After running the function, you will receive the trained Ridge model, a list of selected features based on Lasso's coefficients, and the Ridge model's coefficients.
  
  - This approach helps in reducing the dimensionality of your data and selects features that are most important in predicting the target variable. Using both Lasso and Ridge regression, it takes advantage of both Lasso's feature selection capabilities and Ridge's ability to work well even with correlated features.
  
  - Note: Ensure that the dataset file path, target column name, test size, and random state are correctly specified before running the function.

### Expected output
  - After running the perform_ridge_with_lasso_selection function, you can expect a printed output and a plot.
  - output will indicate how many features were selected and how many were eliminated by the Lasso regression. The coefficient values for the selected features will also be displayed. Lastly, the best alpha values found for both the Lasso and Ridge regressions will be printed.
  ```
  Lasso picked 23 variables and eliminated the other 27 variables
  average_age          -0.000350
  avg_runs_allowed     -0.062682
  wins                  0.052297
  losses               -0.050331
  run_average_9         0.000322
  runs_x                0.088040
  walks                 0.000385
  hit_batter           -0.000075
  hits_9               -0.000109
  homeruns_9           -0.001074
  strikeouts_9          0.000711
  average_batter_age    0.000485
  avg_runs_scored       0.064126
  runs_y               -0.088847
  hits_y               -0.002030
  triples              -0.000279
  RBI                   0.000739
  bases_on_balls       -0.001674
  batting_average       0.000496
  GDP                   0.000697
  HBP                  -0.000387
  sacrifice_hits        0.000241
  sacrifice_flies       0.000057
  dtype: float64
  
  Best alpha for Lasso:  0.0001
  Best alpha for Ridge:  0.019306977288832496
  ```

  - Additionally, a plot will be displayed showing the feature importance based on the coefficients from the Lasso regression. The plot is a horizontal bar plot where the x-axis represents the coefficient value and the y-axis represents the feature names. The longer the bar, the higher the importance of that feature.
    
![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/4fd3b22d-efe9-43eb-87fe-353dafac5bb0)

## Log5 Method for Predicting Winning Percentages

The log5 method is a simple, yet powerful model for predicting the outcome of a matchup between two teams. It was developed by Bill James, a pioneer in baseball analytics. The method takes into account the individual winning percentages of the two teams, and uses these to predict the outcome of a matchup.

Here's how the formula works:

Given two teams, A and B, with winning percentages WinA and WinB respectively, the probability that team A will win against team B is given by:
    ```
    P(A beats B) = (WinA - WinA*WinB) / (WinA + WinB - 2*WinA*WinB)
    ```

- The formula accounts for the strengths of both teams. If both teams have a winning percentage of .500, the Log5 estimate is also .500, indicating an equal chance for both teams. But if one team is stronger, say team A has a winning percentage of .600 and team B has a winning percentage of .400, the Log5 estimate increases the probability that team A will win (to approximately .600) and decreases the probability that team B will win.
The Python code in log5_model.py calculates these probabilities for all pairings of teams in a given league. The output is a 2D matrix where both rows and columns represent teams. For example, the entry at row i and column j represents the probability that team i will beat team j.

- The output is a 2D matrix where both rows and columns represent teams. For example, the entry at row i and column j represents the probability that team i will beat team j. 
    ```
    KBO Probabilities: 
    |                   | Binggre Eagles | MBC Blue Dragons | Chungbo Pintos | Sammi Superstars |
    |-------------------|----------------|------------------|----------------|------------------|
    | Binggre Eagles    | NaN            | 0.415986         | 0.239861       | 0.170191         |
    | MBC Blue Dragons  | 0.584014       | NaN              | 0.51466        | 0.408016         |
    | Chungbo Pintos    | 0.760139       | 0.48534          | NaN            | 0.218646         |
    | Sammi Superstars  | 0.829809       | 0.591984         | 0.781354       | NaN              |
    
    MLB Probabilities:
    
    |     | 108      | 109      | 110      | 111      |
    |-----|----------|----------|----------|----------|
    | 108 | NaN      | 0.646016 | 0.659023 | 0.469629 |
    | 109 | 0.353984 | NaN      | 0.514338 | 0.326687 |
    | 110 | 0.340977 | 0.485662 | NaN      | 0.314196 |
    | 111 | 0.530371 | 0.673313 | 0.685804 | NaN      |
    
    ```

- The Python code calculates these probabilities for all pairs of teams in a given dataset, returning a pandas DataFrame where each cell at the intersection of team_a and team_b represents the probability that team_a will win against team_b.

- Visual representation of this output using a heatmap is also provided. The heatmap is a useful tool for quickly identifying which teams are more likely to win against others. In the heatmap, darker colors represent higher probabilities.
  
![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/cff5df2f-4437-4ffc-9141-d3464a7660a9)
![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/e5a4c31a-9415-4f96-922c-3cf7a133dfbb)


## Poisson and Negative Binomial Models (poisson_negative_binomial_model.py)

- Using a Poisson regression model, which is suitable for predicting counts.  Treating the number of runs scored as a count, I built a model that predicts the number of runs a team is expected to score based on their stats.
    
    - **Poisson Regression Model**: This is a type of Generalized Linear Model (GLM) that transforms the dependent variable using the natural logarithm. It's useful for modeling outcomes that represent counts, like the number of runs scored in a game. The assumption here is that the data follows a Poisson distribution.
    
    - **Negative Binomial Regression Model**: This model is a generalization of the Poisson model. It's used when the variance of the data is greater than the mean (which is often the case in real-world count data), resulting in overdispersion. The negative binomial model accounts for this overdispersion by adding a parameter to model the variance separately from the mean.

### **Usage**
- To use poisson_negative_binomial_mode.py, you first need to define your features for each dataset, like so:
    ```
    kbo_features = ['average_age', 'avg_runs_allowed', 'ERA', 'total_games', 'walks', 'strikeouts_x', 'batting_average', 'OBP', 'SLG', 'OPS']
    mlb_features = ['wins', 'losses', 'home', 'avg_runs_scored', 'doubles', 'triples', 'homeruns', 'strikeouts_x', 'bases_on_balls', 'hits_x', 'at_bats']
    ```
- Then, call the predict_win_probability function, passing in the relevant DataFrame and list of features:
    ```
    df_kbo = predict_win_probability(df_kbo, kbo_features)
    df_mlb = predict_win_probability(df_mlb, mlb_features)
    ```
- After running the model, you can visualize the results with the plot_predicted_win_probability function:
    ```
    plot_predicted_win_probability(df_kbo, 'KBO')
    plot_predicted_win_probability(df_mlb, 'MLB')
    ```
- These functions will add a new column to your DataFrame with the predicted win probabilities and generate a histogram showing the distribution of these probabilities.

## Expected Output

```
Using 'total_runs_scored' as the dependent variable
Defining independent variables
Adding constant to the independent variables
Fitting the model, this may take a while...
Predicting win probabilities
Applying sigmoid transformation
Model summary:
                 Generalized Linear Model Regression Results                  
==============================================================================
Dep. Variable:      total_runs_scored   No. Observations:                  323
Model:                            GLM   Df Residuals:                      312
Model Family:                 Poisson   Df Model:                           10
Link Function:                    Log   Scale:                          1.0000
Method:                          IRLS   Log-Likelihood:                -1501.6
Date:                Thu, 13 Jul 2023   Deviance:                       354.09
Time:                        14:55:28   Pearson chi2:                     354.
No. Iterations:                     4   Pseudo R-squ. (CS):              1.000
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------------
const                2.8069      0.076     36.767      0.000       2.657       2.957
average_age          0.0002      0.002      0.141      0.888      -0.003       0.003
avg_runs_allowed     0.0535      0.020      2.656      0.008       0.014       0.093
ERA                 -0.0660      0.021     -3.216      0.001      -0.106      -0.026
total_games          0.0094      0.000     19.728      0.000       0.008       0.010
walks             7.247e-05   4.48e-05      1.617      0.106   -1.54e-05       0.000
strikeouts_x      1.909e-05   2.66e-05      0.718      0.473    -3.3e-05    7.12e-05
batting_average      0.9343      0.392      2.384      0.017       0.166       1.702
OBP                  9.9074      4.757      2.083      0.037       0.584      19.231
SLG                  7.9076      4.767      1.659      0.097      -1.436      17.251
OPS                 -6.0085      4.763     -1.261      0.207     -15.345       3.328
====================================================================================
Using 'RBI' as the dependent variable
Defining independent variables
Adding constant to the independent variables
Fitting the model, this may take a while...
Predicting win probabilities
Applying sigmoid transformation
Model summary:
                 Generalized Linear Model Regression Results                  
==============================================================================
Dep. Variable:                    RBI   No. Observations:                  124
Model:                            GLM   Df Residuals:                      112
Model Family:                 Poisson   Df Model:                           11
Link Function:                    Log   Scale:                          1.0000
Method:                          IRLS   Log-Likelihood:                -206.75
Date:                Thu, 13 Jul 2023   Deviance:                      0.31470
Time:                        14:55:28   Pearson chi2:                    0.311
No. Iterations:                     4   Pseudo R-squ. (CS):            0.09173
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          z      P>|z|      [0.025      0.975]
-----------------------------------------------------------------------------------
const               0.4225      2.521      0.168      0.867      -4.519       5.364
wins            -8.389e-07   3.07e-05     -0.027      0.978   -6.09e-05    5.93e-05
losses           2.717e-06   3.36e-05      0.081      0.935   -6.31e-05    6.85e-05
home               -0.0572      0.729     -0.078      0.937      -1.487       1.372
avg_runs_scored     0.2177      0.227      0.958      0.338      -0.228       0.663
doubles             0.0321      0.273      0.118      0.906      -0.504       0.568
triples             0.0695      0.818      0.085      0.932      -1.535       1.674
homeruns            0.0032      0.270      0.012      0.991      -0.526       0.533
strikeouts_x       -0.0047      0.069     -0.068      0.946      -0.140       0.130
bases_on_balls      0.0175      0.125      0.140      0.888      -0.227       0.262
hits_x             -0.0204      0.161     -0.127      0.899      -0.336       0.295
at_bats             0.0045      0.097      0.046      0.963      -0.185       0.194
===================================================================================
```

![user_input_1](https://github.com/Junho-eum/Baseball_Analytics/assets/74083204/9595dac9-564a-43c2-afab-5d49bc1fa8f7)
