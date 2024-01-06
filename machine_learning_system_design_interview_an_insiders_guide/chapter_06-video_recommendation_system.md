# Clarify Requirements

Design a homepage video recommendation system. The business objective is to increase user engagement. Each time a user loads the homepage, the system recommends the most engaging videos. Users are located wordwide and videos can be in different languages. The are approximately 10 billion videos on the platform, and recommendations should be served quickly.

# Frame the Problem as an ML Task
## Defining the ML objective
- Maximize the number of user clicks.
- Maximize the number of completed videos.
- Maximize the total watch time.
- Maximize the number of relevant videos.
  - Define relevance based implicit or explicit rules.

## Specifying the system's input and output
Takes a user as input, and a list of recommended videos as output.

## Choosing the right ML category
### Content based filtering
Use video features to recommand similar videos.
- Pros
  - Ability to recommend new videos.
  - Ability to capture the unique interests of users.
- Cons
  - Difficult to discover a user's new interest.
  - Require domain knowledge and feature engineering.

### Collaborative filtering (CF)
Use user-user similarities or video-video similarities to recommend new videos.
- Pros
  - No domain knowledge required.
  - Easy to discover user's new areas of interest.
  - Faster and less compute-intensive.
- Cons
  - Cold-start problem.
  - Cannot handle niche (special) interests.

| | Content based filtering | Collaborative filtering |
|-|--|--|
| Handle new vidoes | Yes | No |
| Discover new interest areas | No | Yes |
| No domain knowledge | No | Yes |
| Efficiency | No | Yes|

### Hynord filtering.

- Parallel: CF based + content based -&gt; Combiner.
- Sequential: CF based -&gt; content based

## Data Preparation
### Data Engineering
- Videos
  - ID, length, tags, titles, likes, views, language, etc.
- Users
  - ID, username, age, gender, city, country, language, timezone, etc.

- User-video interactions
  - impression/click/watch/like/..., timestamps

### Feature Engineering
#### Video features

- ID: Categorical data -&gt; Embedding layer.
- Duration: Direct.
- Language: Embedding layer.
- Tile: Context-aware model model. BERT.
- Tags: Lightweight model. [CBOW](https://www.kdnuggets.com/2018/04/implementing-deep-learning-methods-feature-engineering-text-data-cbow.html).

#### User features
- User demographics
  - ID/Langugage/City/Country: Embedding
  - Age: Bucketize + One-hot.
  - Gender: One-hot.

- Contextual information
  - Day of week: One-hot.
  - Time of day: Bucketize + One-hot.
  - Device: One-hot.

- User historical interactions
  - Search/Like/Watched and Impressions.

# Model Development
## Matrix factorization
### Feedback matrix
- Explicit feedback. Sparse.
- Implicit feedback. Noisy.
- Combination.

### Matrix factorization model
Embedding model. Dot product of a user embedding matrix and a video embedding matrix, all as trainable.

### Matrix factorization training
- Squared distance over observed  pairs
  - Ignore unobserved.
- Squared distance over observed and unobserved  pairs
  - Treat unobserved as negative.
- A weght combination of both.

### Matrix factorization optimization
- SGD: Minimize loss.
- Weights Alternating Least Squares (WALS): Specific to matrix factorization. Fix one and optimize the other. Repeat.

### Matrix factorization inference

Dot product and TopK scores.  (Commonly used in collabrative filtering).

- Pros
  - Training speed: Two embedding matrices to learn.
  - Serving speed: Fast. Embeddings are static.
- Cons
  - Only relies on user-video interactions.
  - Handle new users are difficult.

## Two-tower Neural Network

Feed video/user features to two DNN towers. Run dot product. 

### Constructing dataset
1 for positive. 0 for negative.

### Choosing loss function
Train with cross entropy loss function.

### Inference
Approximate nearest neighbot methods to find TopK similar video embeddings.

- Pros
  - Use video and user features.
  - Handles new users.
- Cons
  - Slower serving.
  - Expensive training.

# Model Evaluation
[StackeOverflow](https://stackoverflow.com/questions/55748792/understanding-precisionk-apk-mapk)
## Offline metrics
- [**Precision@k**](https://medium.com/@m_n_malaeb/recall-and-precision-at-k-for-recommender-systems-618483226c54): Measures the propagation of relevant videos among the top k recommended videos. Multiple k values (e.g. 1, 5, 10) can be used.

- [**mAP**](https://jonathan-hui.medium.com/map-mean-average-precision-for-object-detection-45c121a31173): Measures the ranking quality of recommended video. A good fit because the relevant score are binary.

- Diversity: Measures how dissimilar recommendation videos are to each other.

## Online metrics

- Click-through rate (CTR): $latex CTR = \frac{number\ of\ clicked\ videos}{total\ number\ of\ recommended\ videos}$.
- The number of completed videos.
- Total watch time.
- Explicit user feedback.

# Serving
## Candidate generation
- Narrow down the videos from potentially billions to thousands. Prioritize efficiency over accuracy.
- A model doesn't relies on video features and can handle new users. A two-tower neural network is a good fit.
- Use multiple candidate generation it could improve performance due to different reasons of viewing videos.

## Ranking
- Prioritize accuracy over efficiency.
- Content-based filtering and pick a model based on video features. Larger and heavier two tower neural network.

## Re-ranking
- Re-ranks the videos by adding additional criteria or constraints.
- Clickbait. Region restriction. Misinformation. Duplication. Faireness and bias.

# Challenges
- Serving speed
  - 2 stage.
- Precision
  - More powerful model based on scoring component.
- Diversity
  - Multiple candidate generators.
- Code-start Problem
  - New users: Basic information like age, gender, etc. Recommend and refine.
  - New videos: Recommend to random users and retune.
- Training scalability
  - The model should be easily fine-tuned.
