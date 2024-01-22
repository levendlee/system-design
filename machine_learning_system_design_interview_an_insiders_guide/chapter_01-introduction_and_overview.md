# Chapter 01 Introduction and Overview

## Components of production-ready ML system
- Data
    - Data collection
    - Data verification
    - Feature extraction
- ML Algorithm
- Evaluation Pipeline
- Deployment
    - Process Management Tools
    - Analysis Tools
    - Machine Resource Management
- Monitoring
- Service Infrastructure

## ML system design steps
- Clarifying requirements
- Framing the problem as an ML task
- Data preparation
- Model development
- Evaluation
- Deployment and serving
- Monitoring and infrastructure

## 1. Clarifying Requirements
- Business **objective**
- **Features** the system needs to support
- **Data**
    - Sources? Dataset size? Labeled?
- **Constraints**
    - On-device or datacenter? Improve automatically?
- **Scale** of the system
- **Performance**

## 2. Frame the Problem as an ML Task
### Defining the ML objective
Frame the business objective as an ML objective.

| Application | Business objective | ML objective |
|---|---|---|
| Event ticket selling app | Increase ticket sales | Maximize the number of even registrations |
| Video streaming app | Increase user engagement | Maximize the time users spend watching videos |
| Ad click prediction system | Increase user clicks | Maximize click-through rate |
| Harmful content detection in a social media platform | Improve the platform's safety | Accurately predict is a given content is harmful |
| Friend recommendation system | Increase the rate at which users grow their network | Maximize the number of formed connections |

### Specifying the system's input and output
- Multiple inputs or outputs.
    - Interaction
        - One input and multiple predictions on different interactions.
        - Two inputs and one prediction on the interactions.
- Multiple models.

### Choosing the right ML category
- Supervised learning
    - Classification
        - Binary
        - Multiclass
    - Regression
- Unsupervised learning
    - Clustering
    - Association
    - Dimension reduction
- Reinforcement learning

## 3. Data Preparation
### 3.1. Data engineering
Collecting, storing, retrieving and processing data.

#### Data sources
Who collect? How clean? Trusted? User-generated/system generated?

#### Data storage
Databases.
- SQL
    - Relational: MySQL; PostgreSQL.
- NoSQL
    - Key/Value: Redis, DynamoDB.
    - Column-based: Cassandra, HBase.
    - Graph: Neo4J
    - Document: MongoDB. CouchDB.

#### Extract, transform and load (ETL)
Transform: Data is cleansed, mapped and transformed into a specific format.

#### Data types
- Structured
    - Numerical
        - Discrete
        - Continuous
    - Categorical
        - Ordinal
            - Numerical relationships between data.
        - Nominal
            - No numerical relationships between data.
- Unstructured
    - Audio
    - Video
    - Image
    - Text

Models
- Structured
    - **Deep learning**
    - Decision Trees
    - Linear Models
    - SVM
    - Naive Bayes
    - KNN
- Unstructured
    - **Deep learning**

### 3.2. Feature engineering
- Use domain knowledge to select and extract predictive features from raw data
- Transforming predicive features into format usable by the model

#### 3.2.1. Handiling missing values {.numbered}
##### Deletion
- Column deletion
    - Column has too many missing values.
- Row deletion
    - Row has too many missing values.
##### Imputation
Filing in missing values with 
- Default.
- Mean, median, or mode (the most common value).

#### 3.2.2. Feature scaling
Hard to learn when features are in different ranges or has a skewed distribution.
##### Normalization (min-max scaling). 
$z = \frac{x - x_{min}}{x_{max} - x_{min}}$

##### Standardization (Z-score normalization).
$z = \frac{x - \mu}{\delta}$

##### Log scaling.
Less skewed and converge faster.
$z = log(x)$

#### 3.2.3. Discretication (Bucketing)
Learning a fre categories instead of an infinite number of possibilities.

#### 3.2.3 Encoding categorical features
##### Integer encoding
Natural relationshipt between each other.

##### Onehot encoding
New binary feature is created for each unique value.

##### Embedding learning
When the number of unique values the feature takes is very large.

### Talking points
- Data availability and data collection.
- Data storage.
- Feature engineering.
- Privacy.
- Bias.

## 4. Model Development

### 4.1. Model selection
- Establish a simple baseline.
- Experiment with simple models.
- Switch to more complex models.
- Use an combination of models.
    - Bagging. Boosting. Stacking.

Model options:
- Logistic regression.
- Linear regression.
- Decision trees.
- Gradient boosted decision trees and random forests.
- Support vector machines.
- Naive Bayes.
- Factorization Machines (FM).
- Neural networks.

Trade-offs:
- Amount of data required.
- Training speed.
- Hyperparameters to choose and tuning.
- Possibility of continual learning.
- Compute requirements.
- Model's interpretability.

### 4.2. Model training
#### 4.2.1 Constructing dataset
##### Collect the raw data

##### Identify features and labels
- **Hand labeling**
- **Natural labeling**
No explicit labeling. Inferred automatically without human annotations.

##### Select a sampling strategy
Convenience sampling/Snowball sampling/Stratified sampling/Reservior sampling/Importance sampling.

##### Split the data
Training/evaliation/test dataset.

##### Address any class imbalances
Serious issue as it might not have enough data to learn the minority class.

- **Resampling training data**
Oversample minority or undersample majority.

- **Altering the loss function**
More weights on minority class. Class-balanced loss and focal loss.

#### 4.2.2 Chooing loss function

#### 4.2.3 Training from scratch v.s. fine-tuning

#### 4.2.4 Distributed training

#### Talking points
- Model selection: Time. Data. Compute. Latency. Device/Datacenter. Interpretability. # of parameters. Typical architectures.

- Dataset labels: Obtain. Quality. Natural labels. User feedbacks.

- **Model training**:
    - Loss (Cross-entropy, MSE, MAE, Huber loss, etc).
    - Regularization (L1, L2, Entropy Regularization, K-fold CV, Dropout).
    - Backpropagation.
    - Optimization (SGD, AdaGrad, Momentum, RMSProp).
    - Activation functions (ELU, ReLU, Tanh, Sigmod).
    - Imbalanced dataset.
    - Bias/variance trade-off.
    - Overfitting/underfitting.

- Continual learning: Online with each new data point? Personalize? Retrain?

### 4.3 Model Evaluation

#### 4.3.1 Offline evaluation

During model development phase.

| Task | Offline metrics |
|---|---|
| Classification | Precision, recall, F1 score, *accuracy, ROC-AUC, PR-AUC*, confusion matrix |
| Regression | MSE, MAE, RMSE |
| Ranking | Precision@k, recall@k, *MRR*, mAP, *nDCG* |
| Image generation | *FID*, Inception score |
| NLP | BLEU, *METEOR*, ROUGE, *CIDER, SPICE* |

Note: Haven't figured out all the *tilted* metrics.

#### 4.3.2 Online evaluation

In production after deployment.

| Problem | Online metrics |
|---|---|
| Ad click prediction | Click-through rate, revenue lift, etc |
| Harmful content detection | Prevalence, valid appeals, etc |
| Video recommendation | Click-through rate, total watch time, number of completed videos, etc |
| Friend recommendation | Number of requests sent/accepted per-day, etc |

##### Talking point
- Fairness and bias: How to fix bias. Malicious intent get access.

### 4.4. Deployment and Serving

#### 4.4.1. Cloud vs. On-device deployment
Simplicity / Cost / Network Latency / Inference Latency / Hardware constraints / Privacy / Dependency on internetc connection.

#### 4.4.2. Model compression
- Knowledge distillation
- Pruning
- Quantization

### 4.5. Test in Production
Shadow deployment, canary release, interleaving experiments, bandits, etc.

#### Shadow deployment
Double inference. Old for production. New for testing.

#### A/B testing
Random routing and massive test cases.

### 4.5. Prediction pipeline
#### Batch prediction
Offlined. Pre-computed.

Pros:
- Fast

Cons:
- Less responsive to changing preferences.
- Need to know advance what needs to be pre-computed.

#### Online prediction
Online.

#### Talking points
- Serving fast and reliable.

### 4.6. Monitoring
#### Why a system fails in production
Data distribution shift.
- Training on larger dataset.
- Regularly retrain the model.

#### What to monitor
##### Operation-related metrics
Serving time. Throughput. Number of prediction requests. CPU/GPU utialization, etc.

##### ML-specific metrics
- Monitoring inputs/outputs
- Drifts
  - Input data distribution shifts
- Model accuracy
- Model versions

## Infrastructure
