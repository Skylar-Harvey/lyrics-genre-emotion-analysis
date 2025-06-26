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
df_list = [df_artist,df_lyrics,df_spot60k,df_genius]
```
Note: In this case I used the file paths to where the data was located on my computer. These paths would of course need to be changed if you download the data to some place else.

### Step 3
This cell is simply to check that step 2 was successfully executed, and that there were no errors in the data being loaded in as pandas dataframes:
```
for df in df_list:
	print('Dataframe Information:')
	print(df.info())
	print(df.shape)
	print(df.columns)
	print('Dataframe Samples:')
	print(df.head())
	print()
	print()
	print("END OF DATASET INFORMATON AND SAMPLE")
	print()
	print()
```
### Step 4
This cell is another check on each of the newly created dataframes to see how many, if any, null values are found in the dataframes:
```
slice = 0
for df in df_list:
	df_names_list = ["df_artist","df_lyrics","df_spot60k","df_genius"]
	print(f'Missing values in {df_names_list[slice]}:')
	print(df.isnull().sum())
	slice +=1
```
### Step 5
This cell is used to adjust and standardize column names across the dataframes. Taking it piecemeal:

`df_list[0] = df_artist.rename(columns={"Link":"Artist_Link"})`
This renames the "Link" column of "df_artist" to now be "Artist_Link".

```
df_list[1] = df_lyrics.rename(columns={
	"ALink":"Artist_Link",
	"SName":"Song_Name",
	"SLink":"Song_Link",
	"Lyric":"Lyrics",
	"language":"Language"
	})
```
This renames several columns of the "df_lyrics" dataframe:
- "ALink" --> "Artist_Link"
- "SName" --> "Song_Name"
- "SLink" --> "Song_Link"
- "Lyric" --> "Lyrics"
- "language" --> "Language"

`df_list[2] = df_spot60k.rename(columns={"song":"Song_Name","text":'Lyrics'})`
This renames the "song" and "text" columns of the "df_spot60k" dataframe to be "Song_Name" and "Lyrics" respectivly.

```
df_list[3] = df_genius.rename(columns={
	"title":"Song_Name",
	"lyrics":"Lyrics",
	"language":"Language",
	"tag":"Genre"
	})
```
This renames several columns of the "df_genius" dataframe:
- "title" --> "Song_Name"
- "lyrics" --> "Lyrics"
- "language" --> "Language"
- "tag" --> "Genre"

And finally, these next few lines just give us a check to see if the renaming of the columns went successfully:
```
print("New column names for df_artist:")
print(df_list[0].columns)
print("New column names for df_lyrics:")
print(df_list[1].columns)
print("New column names for df_spot60k:")
print(df_list[2].columns)
print("New column names for df_genius:")
print(df_list[3].columns)
```

### Step 6
This cell simply adds a new column ("Genre") to the "df_spot60k" dataframe. This was done because the data in this datafram did not come marked with genre labels. Rather than remove it altogether from the project, I elected to keep it and assign the songs it contains a genre lable of "unknown". In the scope of this project, these songs were then lumped in with any other songs in the datasets that lacked any know genre associations . In future endevours, it would be better to find a way to find accurate genre lables for each of these songs.

`df_list[2]["genre"]="Unknown"`

### Step 7
This cell drops columns not needed for the analysis from each of the dataframes. This are:

**From "df_artist":**
- "Popularity"

**From "df_lyrics":**
- "Song_Link"

**From "df_spot60k":**
- "Link"
- "artist"

**From "df_genius":**
- "artist"
- "year"
- "views"
- "features"
- "id"
- "language_cld3"
- "language_ft"

```
df_list[0] = df_list[0].drop(columns=["Popularity"])
df_list[1] = df_list[1].drop(columns=["Song_Link"])
df_list[2] = df_list[2].drop(columns=["link","artist"])
df_list[3] = df_list[3].drop(columns=["artist","year","views","features","id","language_cld3","language_ft"])
```

### Step 8
This cell drops rows in the various dataframes that have null values in the columns that will be relevant to the anaysis being done later. These columns are:

**From "df_artist":**
- "Artist"
- "Genres"
- "Songs"
- "Artist_Link"

**From "df_lyrics":**
- "Artist_Link"
- "Song_Name"
- "Lyrics"
- "Language"

**From "df_genius":**
- "Song_Name"
- "Language"

```
df_list[0] = df_list[0].dropna(subset=["Artist","Genres","Songs","Artist_Link"])
df_list[1] = df_list[1].dropna(subset=["Artist_Link","Song_Name","Lyrics","Language"])
df_list[3] = df_list[3].dropna(subset=["Song_Name","Language"])
```

### Step 9
This cell cleans out non-music entries that were part of the "df_genius" dataframe.
`df_list[3] = df_list[3][df_list[3]["Genre"] != "misc"]`

### Step 10
This cell establishes a new dataframe formed by merging the now cleaned "df_artist" and "df_lyrics" dataframes. This effectivly brings togther the data that was originally downloaded kept as two seperate tables into a single dataframe that can be used within the project. The dataframe "df_artist" contained, amoung other data, information about each song's genre association, and the "df_lyrics" dataframe contained information about each song's lyrics and language. These two dataframes are left merged (df_lyrics to df_artist) on the "Artist_link" column that they share. Following this, some checks are made on the new merged dataframe, "df_merged", to ensure that the meger was sucessful:
```
df_merged = df_list[1].merge(df_list[0], on="Artist_Link", how="left")
df_list = [df_merged, df_list[2], df_list[3]]
print(df_list[0].info())
print(df_list[0].head())
print(df_list[0].isnull().sum())
print(df_list[0].shape)
print(df_list[0]["Language"].value_counts())
```

### Step 11
