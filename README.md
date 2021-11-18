# diabetes

Modeling High Risk PAID Scores of Participants in Austin, TX Diabetes Self-Management Program

## Project Motivation

Type 2 Diabetes is a chronic metabolic disorder that affects many
Americans characterized by high blood glucose levels through insulin
resistance. It can lead to complications elsewhere in the body, such as
retinopathies, kidney failure, and amputation of limbs due to poor wound
healing. Within the United States, T2 Diabetes affects 26 million
people. ([NIH T2 Diabetes](https://www.nih.gov/research-training/accelerating-medicines-partnership-amp/type-2-diabetes)).

As of 2017, the annual burden of diabetes treatment on the US healthcare
system is approximately 327 billion USD. Of this approximation, 237
billion USD is associated with direct medical costs, including inpatient
hospital treatment, prescriptions and medical supplies, and outpatient
visits. ([American Diabetes Association
Stats](https://www.diabetes.org/resources/statistics/cost-diabetes)).

The Problem Areas in Diabetes (PAID) questionnaire is a short,
self-guided questionnaire completed by patients with diabetes. The PAID
questionnaire is a robust measure that can evaluate emotional distress
as well as predict future blood glucose control in patients with
diabetes. The questionnaire is scored on a scale from 0-100; scores of
40 and higher are strongly associated with emotional burnout, while
scores below 10 with poor glucose control are associated with patients
in denial of their condition. ([PAID Questionnaire UConn](https://www.huskyhealthct.org/providers/provider_postings/diabetes/PAID_problem_areas_in_diabetes_questionnaire.pdf)).

##Exploratory Data Analysis
### Dataset description

The data for this project was obtained through the [Datacamp Career Hub
Repository](https://github.com/datacamp/careerhub-data/tree/master/Diabetes%20Self-Management).
It is publicly sourced from Public Health Department of Austin, Texas
through their Diabetes Self-Management Education program. Each
observation from the data set are responses from individual participants
who completed the program.

The raw dataset contains 1688 observations and 21 unique variables, 3 of which are numeric and 18 that are categorical. The majority of the variables appear to be self-explanatory, while some are open to interpretation. I have defined
the variables in a data dictionary below, as one was not provided with
the dataset. For the purposes of my analysis, I have created three
additional categorical variables, which I have also defined in the data
dictionary.

'has_pcp' is a simplification of the 'medical_home'
variable. It is not particularly important which specific clinic that a
participant goes for their primary care, but rather that they *do* have
a primary care provider with whom they can follow up regularly over
time. Emergency rooms are acute care providers, and not primary care
providers.

'low_income' is an indicator variable which describes if the participant
has a low income, dependent on their 'insurance' enrollment.
Participants must have a recognized low gross income to qualify for
either Medicaid or MAP (Medical Access Program).

'PAID_high_risk' is an indicator variable dependent on the 'PAID_score'
of the participant. Without any data to assess a participant's glucose
control, it is difficult to assess whether a participant is in denial of
their condition. A 'PAID_score' \>= 40 is considered to be high risk, as
it is shown to indicate emotional burnout in a participant. For this
analysis, 'PAID_high_risk' will be the response variable for
classification.

There are many variables with missing values. Some of these values are missing not at random and can be explained. Cases where the participant does not have diabetes will not have a PAID score, and can not have their risk evaluated. Similarly, variables such as 'has_pcp' and 'low_income' are dependent on
'medical_home' and 'insurance', respectively, and will therefore have
the same missing values as their parent columns.

However, the remaining missing values are of unknown nature. The raw
dataset did not provide any guidance on the interpretation of these
missing values. Dropping observations with any missing values will
deplete the dataset drastically. One-hot encoding the variables will be the better option to pursue here, including "variable_NA" columns to possibly
identify any other variables with missing not at random values.

## Model Development
Prior to modeling the data, the dataset will have to be treated. As discussed earlier, the variables in the dataset will be one-hot encoded to address the many missing values in the dataset without dropping observations.The dataset will be split into two tables: one containing predictor variables and the other containing only the response variable, PAID_high_risk. The table of predictor variables needed to be modified to conduct logistic regression. 

To do so, the predictor variables were grouped by character variables, which consist of survey questions indicating status with Y/N/NA answers (e.g. whether the participant has diabetes) and ordinal variables, which consist of survey questions with answers on a spectrum (e.g. how many days in a week does the participant exercise). For the purposes of logistic regression modeling and to reduce the number of dummy columns, the answers to the ordinal variables were releveled to follow "low, medium, high" scoring. The character and ordinal variable tables were then converted so that each variable had a column for each possible response, including NA values. The characters and ordinals tables were joined together to make a treated table of all predictors.

The treated predictors table was joined with the response table. The training dataset consisted of all participants with diabetes and a valid PAID score. The test dataset comprises all participants without diabetes or do not have a valid status.

Since the training dataset only contains 621 observations, each model will be trained using repeated cross validation 10 times with 5 repeats.

Four types of models (generalized linear regression models, generalized linear models combined with lasso and ridge regression, random forests, and gradient boosting) were generated and tested. Multiple glm and rf models were tested, including median vs. kNN imputation of missing values and removal of any zero variance or near zero variance predictors. Models were initially evaluated by their AUC-ROC values. The best model from each type was then compared against each other by how long the scripts for each model to run 10 times.

The random forest model with kNN imputation and removal of zero variance predictors had the highest AUC-ROC value of 0.7427. However, the time to run this random forest model was 3x that of the generalized linear models (glm_zv_knn and glmnet). Ultimately, the glm model with kNN imputation and zero variance predictors removed was selected as it was faster than the glmnet with a negligible difference in AUC-ROC values (0.7206 vs. 0.7226). Generally, models with AUC-ROC scores between 0.70-0.80 are acceptable.

The test set, diabetes_rw, contains all participants of the survey who did not have a confirmed diabetes diagnosis (diabetes = No, NA). The glm model with kNN imputation and without zero variance predictors will be used on the test dataset once to predict how many participants would be considered high-risk if they theoretically had diabetes.

The glm model indicates that 658 of the 957 observations in the test dataset would be classified with a high risk PAID score if they have diabetes and assuming no other status changes. That is approximately 69% of the test dataset.

### Discussion

There was no available information from the data source regarding when the participants had their blood test for diabetes conducted or if the PAID questionnaire was completed before or after the educational program. Unlike certain viral antibody tests that confirm if a patient has been exposed to a virus in their lifetime (e.g. chickenpox, Epstein-Barr virus), a Type 2 diabetes blood test does not rule out future development of the illness. By that logic, the participants of this survey could potentially develop diabetes in the future. Understanding the indicators that contribute to a high PAID score could help identify patients who could be high-risk diabetic patients. Furthermore, Type 2 diabetes is a chronic illness with no cure as of today. Given the high annual cost of diabetes treatment in the United States, addressing socioeconomic and behavioral predictors of high-risk diabetes could help reduce economic and social burdens on both patients and health systems; in this case, this would concern Austin, TX.

This case study was limited by the relatively small size of the dataset and its many missing values. Having more observations would help improve model development. Gathering more data from the diabetes class sessions within Austin, TX or similar programs across the United States would address this issue. Model development could improve by reducing the number of variables in the model. Depending on the quantity of the additional data added, a different type of model could be used altogether. A generalized linear regression model was a good model in balancing model run time and AUC-ROC scores. However, with more data, random forests or gradient boosted models may have the advantage.

