# Personalization_Proj1
Final project for IEOR 4571: Personalization Theory and Application

Group members: Chao Huang (ch3474) <ch3474@columbia.edu>, Lin Jiang (lj2493) <lj2493@columbia.edu>, Shuo Yang (sy2886) <sy2886@columbia.edu>, Han Xu (hx2282) <hx2282@columbia.edu>.

## Description

Build and test a demo Recsys on yelp 2019 dataset, with several algorithms, including MF, ANN, CMF and FM. It is expected to mimic the scenario of recommending a point of interest to a customer. **The final report can be found [here](report/Personalization_Project1_report.pdf).**

## Problems to solve

* Data cleaning
* Performance boost from side information
* Evaluation Metric
* Algorithm selection
* Model fine-tuning
* Ranking based models v.s. regression based models
* Implicit feedback v.s. explicit feedback

## System design

* Data pipeline (load/sample/split)
* Model implementation (unified API)
* Inference and evaluation module (Precision@K/NDCG@K)
* Parameter tuning system and logger with the help of sqlite database
* Result interpretation (graph+table)
* Re-iterate training method


## Structure of source code

* Data loading modules, including data loader, train-test split and cross-validation. 
    * [Data_Loader](src/data_pipeline/Data_Loader.py) Major class for loading data, it splits train/test set once initialized
    * [Data_Loader](src/data_pipeline/Config.py) parses and loads configuration from json files under config folder The default is good for a try. If you want to create a different configuration file, be sure to change the name of the configuration (i.e, the name of dataset) and follow the instructions listed [here](config/README.md). Otherwise it will cause inconsistency in the database. Later commits will try to check this inconsistency automatically and throw an error if the check fails.
    * [pipeline](src/data_pipeline/pipeline.py) includes cross_validation and test_performance function.
    
* Models:
    * [Base model](src/model/BaseModel.py) all models should inherit from this base model by overloading three specific functions, namely fit() and transform(). Check the annotation of the input parameters for more details.
    * [CollectiveMF](src/model/CollectiveMF.py) a model using CMF algorithm from cmfrec package.
    
    See "Local testing and debugging" part below if you want to implement a new model and run within this framework.

* Evaluations:
    * [Evaluator](src/evaluation/Evaluator.py) no need to construct an instance of this manually, high level functions in pipeline parts call it already.

* Utilities:
    * [Database tools](src/utility/DBUtils.py) handle the definition and insertion of a database, where the records of cross_validation and tests are saved. It will bring much convenience of tuning hyper parameters and model selections.
    * [Summary](src/utility/Summary.py) provides functions to query the database, basically summarizing the cross validation results across all folds and fetching result for the final test evaluation.
    * [Others] other two files end with "utils" include functions for handling paths and model result saving.

* Exploratory Data Analysis:
    * Code and Notebooks of exploratory data analysis and final result plotting can be found [here](src/eda).

## Local testing and debugging
Say if you want to debug your own model using existing methods and functions, follow the following step:

1. Download dataset from the web. Create a log directory and a data directory (can be somewhere else outside this project directory).

2. Create a copy of the [default config file](./config/default_config.json), e.g. as "custom_config.json". Modify the "data_root_path", "db_path", "cache_path" as corresponding 
**absolute** path you set in step 1. 

3. Create a directory called "test" under the root directory and after this step, the repo should look like this:
    ```bash
    .
    ├── config
    ├── LICENSE
    ├── README.md
    ├── recsys_all_in_one.ipynb
    ├── report
    ├── requirements.txt
    ├── src
    └── test
    ```
    It will not be included in version control since it's set ignored.
4. Copy [this notebook](./recsys_all_in_one.ipynb) to your "test" directory.

5. Do all of your testing in the copied notebook. Happy debugging!

6. (Optional) Implement your own model
    * First, all models should inherit from the [BaseModel](src/model/BaseModel.py) 
    * Next, implement a ***fit(train_df, user_info, item_info)*** function, which fits your own algorithm. 
        * The inputs are three **pandas** dataframes. "train_df" will have the 
    schema of (user_id, business_id, stars). "user_info" will have the schema of (user_id, feature_1, ..., feature_n). "business_info" will 
    have the schema of (business_id, feature_1, ..., feature_n). Although your model may not need side information, the consistency of input format
    guarantees the fit function in base model is overridden.
    * Next, implement a ***transform(ui_pairs)*** function, which provides predictions for new user-item pairs.
        * The input is a **pandas** dataframe, with the schema of (user_id, business_id, stars). 
        * The output is a new dataframe with the schema of (user_id, business_id, stars, prediction). Note that all column names should be **exactly the same** as shown here.
    * Finally, if a model have degenerated version, e.g. an FM model only uses user or item side information or none of them. It's recommended to give different names for different versions. If not, these models should have a different
    hyper parameter at least (e.g., use_user_info＝False/True).
     
    

Note that all package dependencies are specified in the [requirements](requirements.txt) file. Make sure you have [spark](https://spark.apache.org/) installed (for this project, we use the latest version 2.4.4) and environment variables already set up. Also if you want to use the database logging system, make sure to have [sqlite3](https://www.sqlite.org/download.html) installed in your environment.
