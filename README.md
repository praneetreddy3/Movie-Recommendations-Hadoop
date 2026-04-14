# Movie-Recommendations-Hadoop

**Collaborative Filtering Movie Recommendation System using Hadoop MapReduce**

## Overview

This project implements a **movie recommendation engine** using Hadoop's MapReduce framework. The system builds a collaborative filtering pipeline that analyzes user-movie rating patterns to predict movies a user is likely to enjoy. The pipeline processes raw rating data through five sequential MapReduce jobs: user grouping, co-occurrence matrix generation, normalization, matrix multiplication, and recommendation scoring.

**Framework:** Hadoop MapReduce (Java)
**Algorithm:** Item-based Collaborative Filtering
**Dataset:** Netflix Prize-style movie ratings (100 movies, 500K+ ratings)

## How It Works

The recommendation pipeline follows a five-stage MapReduce workflow:

```
Raw Ratings (UserID, MovieID, Rating)
    |
[Stage 1: DataDividerByUser]
- Groups all movie ratings by user
- Output: UserID → {Movie1:Rating, Movie2:Rating, ...}
    |
[Stage 2: CoOccurrenceMatrixGenerator]
- Builds movie-movie co-occurrence matrix
- Counts how often each pair of movies was rated by the same user
- Output: MovieA:MovieB → CoOccurrenceCount
    |
[Stage 3: Normalize]
- Normalizes co-occurrence values into transition probabilities
- Output: MovieB → MovieA=NormalizedScore
    |
[Stage 4: Multiplication]
- Multiplies normalized co-occurrence matrix with user ratings
- Combines relationship scores with actual user preferences
- Output: UserID:MoviePair → PredictedRating
    |
[Stage 5: Sum]
- Aggregates partial scores into final recommendations
- Output: User → Movie:PredictedRating (sorted by score)
```

## Architecture

### MapReduce Jobs

**Stage 1 - DataDividerByUser:** Groups raw `(UserID, MovieID, Rating)` tuples by user, producing a consolidated list of all movies each user has rated.

**Stage 2 - CoOccurrenceMatrixGenerator:** For each user's movie list, generates all movie pairs and counts co-occurrences across all users. If User A rated both Movie 1 and Movie 2, the pair `(1:2)` gets a count of 1.

**Stage 3 - Normalize:** Converts raw co-occurrence counts into probabilities. For each movie, divides each co-occurrence count by the total co-occurrences for that movie, producing a transition probability matrix.

**Stage 4 - Multiplication:** Joins the normalized matrix with original user ratings. The CooccurrenceMapper emits `(MovieB, MovieA=relation)` and the RatingMapper emits `(MovieB, UserID:Rating)`. The reducer multiplies relation scores by user ratings.

**Stage 5 - Sum:** Aggregates all partial recommendation scores per (User, Movie) pair to produce final predicted ratings.

### Data Flow

```
Input Format (per line):
UserID,MovieID,Rating

Example:
1,10001,5.0
1,10002,3.0
2,10001,4.0
```

## Prerequisites

- Java JDK 8+
- Hadoop 2.7+ (HDFS + YARN)
- Maven (optional, for building)

## Setup & Usage

### 1. Compile the Java Sources

```bash
# Set Hadoop classpath
export HADOOP_CLASSPATH=$(hadoop classpath)

# Compile all Java files
javac -classpath $HADOOP_CLASSPATH -d build/ src/main/java/*.java

# Create JAR file
jar -cvf recommender.jar -C build/ .
```

### 2. Prepare HDFS

```bash
# Create input directory in HDFS
hdfs dfs -mkdir -p /input/ratings

# Upload rating data
hdfs dfs -put data/sample/*.txt /input/ratings/
```

### 3. Run the Pipeline

```bash
hadoop jar recommender.jar Driver \
  /input/ratings \
  /output/step1_userMovieList \
  /output/step2_coOccurrence \
  /output/step3_normalize \
  /output/step4_multiply \
  /output/step5_recommendations
```

### 4. View Results

```bash
hdfs dfs -cat /output/step5_recommendations/part-r-00000
```

## Repository Structure

```
Movie-Recommendations-Hadoop/
├── README.md
├── .gitignore
├── src/
│   └── main/
│       └── java/
│           ├── Driver.java                      # Pipeline orchestrator
│           ├── DataDividerByUser.java           # Stage 1: Group ratings by user
│           ├── CoOccurrenceMatrixGenerator.java # Stage 2: Build co-occurrence matrix
│           ├── Normalize.java                   # Stage 3: Normalize to probabilities
│           ├── Multiplication.java              # Stage 4: Matrix-rating multiplication
│           └── Sum.java                         # Stage 5: Aggregate recommendations
└── data/
    └── sample/                                  # Sample rating files
        ├── mv_0000001.txt
        ├── mv_0000002.txt
        └── mv_0000003.txt
```

## Results

The system generates personalized movie recommendations for each user with predicted ratings. Movies the user has not yet seen are ranked by predicted preference score, enabling content discovery based on collaborative patterns across the entire user base.

**Key Outputs:**
- Predicted ratings for unseen movies per user
- Ranked recommendation lists sorted by predicted score
- Co-occurrence statistics revealing movie relationships

## Technical Details

**Algorithm:** Item-based collaborative filtering computes similarity between items (movies) based on user behavior patterns, rather than computing user-user similarity. This approach scales better with large user bases.

**Scalability:** Each MapReduce stage is independently parallelizable across the Hadoop cluster. The pipeline handles datasets with millions of ratings by distributing computation across nodes.

**Normalization:** Raw co-occurrence counts are converted to conditional probabilities P(MovieA | MovieB), ensuring that popular movies don't dominate recommendations purely due to frequency.

## Limitations & Future Work

**Current Limitations:**
- Cold-start problem for new users with no rating history
- No temporal weighting (older ratings treated same as recent)
- Binary co-occurrence doesn't capture rating magnitude

**Future Improvements:**
- Incorporate rating values into co-occurrence weights
- Add Spark implementation for faster iterative processing
- Implement hybrid filtering (collaborative + content-based)
- Add real-time recommendation updates via Spark Streaming

## Author

**Praneet Chinthala**
- M.S. Data Analytics Engineering, George Mason University
- Email: praneetreddy66@gmail.com
- GitHub: [@praneetreddy3](https://github.com/praneetreddy3)

## License

This project is for educational purposes.

---

**Last Updated:** April 2026
