# Data Directory

## Movie Rating Files

**Format:** Netflix Prize-style text files (one file per movie)

### File Format

Each file is named `mv_XXXXXXX.txt` where XXXXXXX is the movie ID.

**Content Format (per line):**
```
CustomerID,Rating,Date
```

**Example (mv_0000001.txt):**
```
1488844,3,2005-09-06
822109,5,2005-11-10
885013,4,2005-10-28
```

### Sample Data

The `sample/` directory contains 3 sample movie files for testing:
- `mv_0000001.txt` — Movie 1 ratings
- `mv_0000002.txt` — Movie 2 ratings
- `mv_0000003.txt` — Movie 3 ratings

### Full Dataset

The complete dataset contains 100 movie files with 500K+ total ratings. Due to size constraints, only sample files are included in the repository. The full dataset can be obtained from the [Netflix Prize](https://www.kaggle.com/datasets/netflix-inc/netflix-prize-data) on Kaggle.

### Input Format for MapReduce Pipeline

The Driver class expects the input data directory path as the first argument. The pipeline processes all files in the directory.
