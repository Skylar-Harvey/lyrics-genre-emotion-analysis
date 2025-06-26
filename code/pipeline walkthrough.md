# Pipeline Walkthrough
this is a walkthrough for the analysis pipeline notebook [here](link). 

### Step 1
This cell's function is importing libraries and modules needed for running the pipeline. These are:
- `pandas` for data manipulation and handling
- `matplotlib.pyplot` for creating basic plots and visualizations
- `nltk` (Natural Language Toolkit) used for filtering while preprocessing text
- `seaborn` for creating visualizations
- `counter` and `defaultdict` from python's built-in `collections` module, used for handling dictionaries in the project and for tallying word frequencies.

```
import pandas as pd
import matplotlib.pyplot as plt
import re
import nltk
from nltk.corpus import stopwords
nltk.download("stopwords")
from collections import Counter
from collections import defaultdict
import seaborn as sns
```

### Step 2
This cell's function is to load all of the CSV files containing the data used in this project.
Each CSV file is loaded into the project as a pandas dataframe:
```
df_artist = pd.read_csv("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Kaggle Datasets\\DS_1\\artists-data.csv")
df_lyrics = pd.read_csv("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Kaggle Datasets\\DS_1\\lyrics-data.csv")
df_spot60k = pd.read_csv("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Kaggle Datasets\\DS_2\\Spotify Million Song Dataset_exported.csv")
df_genius = pd.read_csv("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Kaggle Datasets\\DS_3\\song_lyrics.csv")
```

and then these dataframes are collected in a list for easy accesss throughout the rest of the code:
```
df_list = [df_artist,df_lyrics,df_spot60k,df_genius] #list for easy access to dataframes
```

