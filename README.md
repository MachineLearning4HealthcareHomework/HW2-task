# Task 1
## Organisational
Next to this file which discusses some of the decisions made in the code and (tries to) discuss the results, there is the conda environment in a .yml-file, a [data](https://archive.ics.uci.edu/ml/datasets/diabetes+130-us+hospitals+for+years+1999-2008) folder with 3 .csv-files in it and a Jupyter-Notebook with visualisations, code and (partial) results.

## Preprocessing
### First stage
I had a quick look at the descriptions at the different columns and made these changes:

#### diag
The a priori most interesting are the 'diag_X_desc' columns, where the most interesting text is, but if we look further, we can see, that most of the information is represented by the [ICD-9](https://en.wikipedia.org/wiki/List_of_ICD-9_codes) codes of them in the columns 'diag_X'. I chose to delete the 'diag_X_desc' columns, since I assumed that without a lot of work, a general model of these texts would at best yield the information of the ICD-9 codes, at worst introduce noise and make the model worse. I could think about doing a more specialised keyword search of words such as "grave", "severe", "light" or "healthy". 
I decided to save time and not do that but use the chapters of the ICD-9 codes to create new features, which are just the count of diagnoses in these chapters per patient. In these ICD-9 codes, the medical_specialty is also already included.

#### admission
The two admission columns have only a few different values, so I tried to map them to integers, as good as I could without any experience with american/english healthcare. In a different setting, I would have asked an expert to validate my mapping and discuss it with him.

#### discharge_terms
Here is the feature, where I used the most NLP-tech, eventhough there is not much variation in possible values. Like in admission I made a map from text to integers. In this case I only check for one word, instead of trying to match the whole text, because there is too much volatility in them. Again, a more complicated system would (in my opinion) be too complicated for the ~20 different texts, we can bin into 5 categories.  

#### payer_code
Unfortunately, I could not make sense of payer code, even with the help of Google and with the assurance of the authors of the dataset I got rid of that feature.

[*Payer code was removed since it had a high percentage of missing values and it was not considered relevant to the outcome.*](https://www.hindawi.com/journals/bmri/2014/781670/)

#### too constant columns
 The following columns were dropped, because they had too little information, either, like the case of weight they were mostly empty/NaN or too constant. You can come to the same insights, if you look at the first report with statistics in the file.

>     'weight','chlorpropamide','nateglinide','repaglinide','acetohexamide','tolbutamide','glyburide.metformin','glipizide.metformin','glimepiride.pioglitazone','metformin.rosiglitazone','metformin.pioglitazone','acarbose',> 'miglitol', 'troglitazone','tolazamide', 'examide', 'citoglipton'

### Second stage
There are still some problems at this stage, namely that many values are not integers yet and there are a lot of NaNs.
To solve this I categorized the different features in three classes, where each gets treated differently:

**identity_columns:**

> 'Unnamed: 0','diag_1','diag_2', diag_3','admission_type_id','discharge_disposition_id', admission_source_id'

**integer_columns**

> 'age', 'time_in_hospital', 'number_diagnoses', 'num_lab_procedures', 'num_procedures', 'num_medications','number_outpatient', 'number_emergency', 'number_inpatient',

**imputable_columns**

> 'race', 'gender','max_glu_serum', 'A1Cresult','metformin', 'glimepiride', 'glipizide', 'glyburide', 'pioglitazone','rosiglitazone', 'insulin', 'change', 'diabetesMed'

  #### imputable_columns
 The first step was to change the values from their (String) values to integers, we do this blindly with a [LabelEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.LabelEncoder.html#sklearn.preprocessing.LabelEncoder), since many values can't be compared e.g. gender.
 Afterwards the most seen value is used to fill the holes.
  #### identity_columns
  identity_columns already have a good integer mapping, so it should not be changed by a LabelEncoder, that could change them up again, so we keep their identity.
  But the process to fill the NaN's still is the same.
 #### integer_columns
 Much like identitiy_columns, the integer ones must not be changed.
 Unlike identity_columns, the value to fill the NaN's is the mean, since those values are (logically) continious.

## Visualisations
You have some statistics and plots, both from before the preprocessing and from afterwards. Check out the notebook to view it.

We can see, that unfortunately, there is no heavy dependence on our target variable, both in the pre-and post-processed variables.
Further, I found it interesting that all chapters usually only have one case per patient, some chapters even have no case at all. The chapter with the most cases is my chapter 6 (VII, # diseases of the circulatory system). 

## Machine Learning
There are numerous models tested, shallowly tuned with gridsearch and also enhanced with feature eliminiation.
The models are:

 - GradientBoosting (sklearn) 
 - DecisionTree (sklearn)
 - SVC (sklearn)
 - LinearSVC (sklearn)
 - Nearest Neighbot (sklearn)
 - (standard) Neural Net (keras+tensorflow)

## Results
Please consult the end of the notebook for the exact numbers. As of right now, my models are still running for about a day now. I will upload a new notebook with code changes clearly indicated, while still leaving the current code. 
In general, I failed to get any information out of the data. All results have accuracy ~60%, what is the normal split of the test-data, meaning, always saying "yes" is the best estimator found. Because of that, I doubt that an ensemble would fix the problems and there is nothing that comes to my mind, that could (easily) fix my dataset.

## Personal Learnings
I found some new libraries for data visualisation and I got another experience in writing (somewhat) readable code, both, in pre-processing with pandas and in evaluating with the different models.

Also on a less optimistic side, I found that not doing the obvious decision (heavy NLP techniques) can backfire heavily and expose me for the hack that I am.
Still, I do think that doing the project like that is a priori sensible and sets a good baseline for the efforts that should follow, utilising the NLP techniques that I mentioned (which also are not heavy).
Next, I still seem to either severly underestimate the time it takes to train models and do hyperparameterization or I do something false and it takes too long. In this case, I have been training this model for about a day now and have not gotten results, eventhough the computer I am running this on is reasonable for a homesystem.
