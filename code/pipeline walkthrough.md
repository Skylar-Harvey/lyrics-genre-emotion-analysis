# Pipeline Walkthrough
this is a walkthrough for the analysis pipeline notebook [here](https://github.com/Skylar-Harvey/lyrics-genre-emotion-analysis/blob/main/code/Analysis%20Pipeline.ipynb). 

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
This cell establishes a new dataframe formed by merging the now cleaned "df_artist" and "df_lyrics" dataframes. This effectivly brings togther some of the data that was originally downloaded, which was kept as two seperate tables, into a single dataframe that can be used within the project. The dataframe "df_artist" contained, amoung other data, information about each song's genre association, and the "df_lyrics" dataframe contained information about each song's lyrics and language. These two dataframes are left-merged (df_lyrics to df_artist) on the "Artist_link" column that they share. Following this, some checks are made on the new merged dataframe, "df_merged", to ensure that the meger was sucessful:
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
This step runs a few cleaning opperations on the new merged dataframe (df_merged). In order, the following cell of the notebook:
- Drops rows with null values from the dataframe
- Removes the "Songs" column.
- removes all non-english data from the table.
```
df_list[0] = df_list[0].dropna(subset=["Artist","Genres","Songs"])
df_list[0] = df_list[0].drop(columns=["Songs"])
df_list[0] = df_list[0][df_list[0]["Language"]=="en"]
```

### Step 12
This step runs some final steps for cleaning and normalizing the data in the other tables. These steps include:
- droping all non-english data from the "df_genius" dataframe.
- adding a language column to the "df_spot60k" dataframe (all marked "unkown").
- droping the "Artist_Link" column from "df_merged".
- Normalizing column names of "df_merged" and "df_spot60k"
```
df_list[2] = df_list[2][df_list[2]["Language"]=="en"]
df_list[1]["Language"]="en"
df_list[0] = df_list[0].drop(columns=["Artist_Link","Artist"])
df_list[0] = df_list[0].rename(columns={"Genres":"Genre"})
df_list[1] = df_list[1].rename(columns={"genre":"Genre"})
```

### Step 13
This step creates a new dataframe, "df_master" by concatenating the now cleaned other three dataframes. I.E., "df_merged", "df_spot60k", and "df_genius". After doing this, the cell runs a handfull of checks to varify that the new master dataframe is in order and ready to be used. This includes things like checking for null-values in the new dataframe, checking the dimensions of it, checking for duplicates, and looking at the value counts for a few of the columns.
```
df_master = pd.concat([df_list[0],df_list[1],df_list[2]], ignore_index=True)
print("COLUMNS IN DF_MASTER:")
print(df_master.columns)
print("NUMBER OF NULL VALUES IN DF_MASTER:")
print(df_master.isnull().sum())
print("INFORMATION FOR DF_MASTER:")
print(df_master.info())
print()
print("Duplicates in 'Song_Name' column: ", df_master.duplicated(subset="Song_Name").sum())
print("Duplicates in 'Lyrics' column: ", df_master.duplicated(subset="Lyrics").sum())
print("Duplicates in 'Song_Name' and 'Lyrics' column: ", df_master.duplicated(subset=["Song_Name","Lyrics"]).sum())
print("VALUE COUNTS PER COLUMN OF DF_MASTER:")
print("Value counts for 'Language' column:", df_master["Language"].value_counts())
print("Value counts for 'Genre' column:", df_master["Genre"].value_counts().to_string())
```

### Step 14
This cell does a number of things to both clean up the master dataframe and to preprocess the lyrical data contained within it. To begin this, we define two functions called "preprocess_lyrics" and "clean_genre":
```
# Function for preprocessing lyrics for analysis
stop_words = set(stopwords.words("english")).union({"im","ive","youre","youve","yeah","oh","get","gonna","aint","uh","ha","wanna","la","hey","woah","whoa","ooh","mmm"})
def preprocess_lyrics(text):
	if isinstance(text, str):
		text = text.lower()
		text = re.sub(r"[^\w\s]", "", text)
		text = re.sub(r"\d+", "", text)
		words = text.split()
		words = [word for word in words if word not in stop_words]
		return " ".join(words)
	return text
```

```
# Function for processing and cleaning entries in the 'genre' column of the master dataframe
genre_list = ['rock','pop','indie','rap','folk','blues','country','metal','electronic']
def clean_genre(genre_string):
	if isinstance(genre_string, str):
		genres = [g.strip().lower() for g in genre_string.split(';')]
		for g in genres:
			if g in genre_list:
				return g
	return 'other'
```
These functions are then applied to the concatenated master dataframe:
```
df_master["Lyrics_Clean"] = df_master["Lyrics"].apply(preprocess_lyrics) #cleaning up lyric data and inserting clean lyric data into new column
df_master["Genre_Clean"] = df_master["Genre"].apply(clean_genre) #normalizes genre lables, and reasigns song entries with multiple lables a single genre lable based on first match to a desired list (genre_list)
```
And finally, a new, cleaned version of the master dataframe is saved (df_master_cln), and the column "Song_Name" is droped from this new dataframe:
```
df_master_cln = df_master[["Song_Name","Lyrics_Clean","Genre_Clean"]].copy() #creates a copy of the master data frame containing only the cleaned data and song names.
df_master_cln = df_master_cln.drop(columns=["Song_Name"]) #drops song_name data column from clean copy of master dataframe
```

### Step 15
This cell represents a somewhat optional step in that it isn't completely needed to run the analysis. But it is still usefull as a way to get a quick glance at the nature of the lyric data contained in the master dataframe. This can be used as a point in the project to adjust the "stop_words" set defined in the previous cell's function for preprocessing the lyrical data. The cell begins similarly to the previous cell in that it defines two funtions, "word_frequency_by_genre" and "freq_by_genre", the former of which is used by the later:
```
# Function for preformance of word frequency anlaysis of words in different genres
def word_frequency_by_genre(df,genre,top_n=25):
	genre_lyrics = " ".join(df[df["Genre_Clean"] == genre]["Lyrics_Clean"]) #Gather all the lyrics of the specified genre and set them into on large string object
	words = genre_lyrics.split() #convert string into list of words
	word_counts = Counter(words) #count word frequency in genre
	return word_counts.most_common(top_n) #return 20 most common words in genre
```

```
# Function that performs frequency analysis for each genre
def freq_by_genre(df,genre_list,top_n=25):
	result ={}
	for genre in genre_list:
		result[genre] = word_frequency_by_genre(df, genre, top_n)
	return result
```
After defining these two functions, the cell then builds a collection of simple visualizations that show the top 25 most common words used by each genre:
```
genre_list_2 = ['rock','pop','indie','rap','folk','blues','country','metal','electronic','other'] #redefine genre_list to include "other"
freq_dict = freq_by_genre(df_master_cln,genre_list_2) #run frequency analysis on each genre in dataframe, returns top 25 results, stores results in dictionary
for genre in genre_list_2:
	genre_words = freq_dict[genre]
	words, counts = zip(*genre_words)
	plt.figure(figsize=(10,10))
	plt.barh(words,counts)
	plt.gca().invert_yaxis()
	plt.xlabel("Occurences in Lyric Data")
	plt.ylabel("Top 25 Words")
	plt.title(f"Top 25 Words for {genre.capitalize()}")
	plt.tight_layout()
	plt.show()
```

### Step 16
This cell's ofunction is to import the EMOlex tool, used to see assosiations of individual words to different emotional categories, into the project. This is done by importing it's data as a pandas dataframe, and then using that to create a dictionary object that stores each word in the EMOlex dataset as a key who's value pairing(s) are the emotional associations that that word is marked as having. For example, the word "abandoned" in the EMOlex dataset has its associations marked as "anger", "fear", "negative", and "sadness". This is represented in the created dictionary object as `{"abandoned":["anger", "fear", "negative", and "sadness"], ... }`

```
df_emolex = pd.read_csv("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\NRC-Emotion-Lexicon\\NRC-Emotion-Lexicon\\NRC-Emotion-Lexicon-Wordlevel-v0.92.txt",
	sep="\t", engine="python", header=None, names=["word","emotion","value"])

word_associations = defaultdict(list)
for _, row in df_emolex.iterrows():
	if row["value"]==1:
		word_associations[row["word"]].append(row["emotion"])
```

### Step 17
These next few cells (17.1 - 17.4) are where the actual analysis is run. This begins with "step 17.1", in which some intial prep for the analysis is run. Specifically, before the analysis step can be run, we:
- add a column "Row_ID" to the cleaned master dataframe, and give each row of the table a unique identifier number.
- create a new empty dataframe, "df_scores", that has the same amount of rows as does the cleaned master dataframe, as well as an identical "Row_ID" column.
- expand the the "df_scores" dataframe to include a column for each emotional association counted for in the EMOlex dataset. All of these are initialized at a value of zero(0).
```
############# STEP 17.1 - Preparing for the analysis

df_master_cln["Row_ID"] = df_master_cln.index #creates a column of unique identifiers for each entry in the dataframe
df_scores = pd.DataFrame(df_master_cln["Row_ID"].copy()) #generates a new dataframe with the same amout rows as the cleaned master dataframe. both dataframes can be linked by the "Row_ID" key
emo_tag_list = ["anger","anticipation","disgust","fear","joy","negative","positive","sadness","surprise","trust"] #list of unique emotional tags in EMOlex associations list
for tag in emo_tag_list: #expands df_scores to now have a column for each word in emo_tag_list. All are initialized to 0
	df_scores[tag] = 0
```

This bit of prep-work done, we move on to creating a scorring function that goes row-by-row through the cleaned master dataframe with it's preprocessed lyrics and genrates a score for each emotional assotiation based on how many times a word having certain associations is encountered in each string of lyrical data. Essentially, each song's lyrics are tokenized into individual words. Each of these words is then checked to see if it matches an entry in the dictionary genereated in "step 16". If a match is found, the column of "df_scores" that represents the respective emotional associations of that word is increased by a value of one. If a match isn't found, then the word token is passed over and no change to the "df_scores" dataframe is made. With this function in place, we can then run it and generate a table of scores that shows each set of lyric's collective emtional scores. (Note: This part of the code, as it is, takes awhile to run (>2hrs in my case). It, like much else with this project, can certainly be improved upon!)
```
def score_lyrics(row_n): #define new function "score_lyrics". Function takes in one variable, an row number
	token_list = df_master_cln.loc[row_n,"Lyrics_Clean"].split() #create a list of word tokens from the lyric string for row_n
	emotion_counter = {"anger":0,"anticipation":0,"disgust":0,"fear":0,"joy":0,"negative":0,"positive":0,"sadness":0,"surprise":0,"trust":0}
	for word in token_list: #iterate through created list of word tokens
		if word in word_associations: #if a token is in the word_associations dictionary defined in step 16:
			word_key_values = word_associations[word] #pull the value for that word. This value is a list of associated emotions for that word
			for emotion in word_key_values: #iterate through this list of emotions associated with that word
				emotion_counter[emotion] +=1
		else:
			pass
	for tag in emo_tag_list:
		df_scores.loc[row_n, tag] = emotion_counter[tag]

for r in range(len(df_master_cln)):
	score_lyrics(r)
```
Generating this table takes awhile, and so "step 17.3" is another optional (but recomended) step that simply ensures that all of the important dataframes (including the one just made!) are saved as objects that can be recalled later without having to rerun any of the code. I chose here to save everything as both a .csv file and a .pkl file. (Note: the file paths shown in the code are of course particualr to my device and where I chose to store the created files. They should be changed according to need if running this code on your own system.):
```
############# STEP 17.3 - Creating project checkpoint. Saving all created dataframes

save_path_pkl = "F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Saved Dataframes\\pickles\\"
save_path_csv = "F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Saved Dataframes\\CSVs\\"

#saving to .pkl
df_master.to_pickle(save_path_pkl + "df_master.pkl")
df_master_cln.to_pickle(save_path_pkl + "df_master_cln.pkl")
df_scores.to_pickle(save_path_pkl + "df_scores.pkl")
#saving to .csv
df_master.to_csv(save_path_csv + "df_master.csv")
df_master_cln.to_csv(save_path_csv + "df_master_cln.csv")
df_scores.to_csv(save_path_csv + "df_scores.csv")
```
Finally, "step 17.4" creates a whole new dataframe "df_scores_normalized" based off the data in the "df_scores" dataframe. Like the name of this new dataframe suggests, it is a version of the "df_scores" dataframe, but with the values contained therein having been normalized (in this case, row-wise) so that each entry in the dataframe (each set of lyrics) has now a collection of scores (one for each of the emotional associations looked for) that are between the values of zero (0) and one (1).
```
############# STEP 17.4 - Normalizing the values across the rows in df_scores

df_scores_normalized = pd.DataFrame(df_scores.copy())
print("New dataframe created: 'df_scores_normalized'")
print("Data normalization initiated:")
def normalize_scores(row_n):
	scores_list = []
	normal_scores = {"anger":0,"anticipation":0,"disgust":0,"fear":0,"joy":0,"negative":0,"positive":0,"sadness":0,"surprise":0,"trust":0}
	for tag in emo_tag_list:
		scores_list.append(df_scores_normalized.loc[row_n,tag])
	score_range = (max(scores_list)-min(scores_list))
	if score_range == 0:
		pass
	else:
		for tag in emo_tag_list:
			normal_scores[tag] = ((df_scores_normalized.loc[row_n,tag] - min(scores_list)) / score_range)
	for tag in emo_tag_list:
		df_scores_normalized.loc[row_n, tag] = normal_scores[tag]

for r in range(len(df_scores)):
	if r % 100000 == 0:
		print(f"Rows processed : {r}")
	normalize_scores(r)
print("Normalization complete!")
print("Printing sample ...")
print(df_scores_normalized.head(20))

df_scores_normalized.to_pickle(save_path_pkl + "df_scores_normalized.pkl")
df_scores_normalized.to_csv(save_path_csv + "df_scores_normalized.csv")
print("This data has been stored as a .pkl file at:")
print("		" + save_path_pkl)
print("This data has been stored as a .csv file at:")
print("		" + save_path_csv)
```

### Optional Step
This next code block is an entirley optional step I took to reload the "df_scores_normalized" and "df_master_cln" for use in the final steps. In my case here I chose to do both so I could verify that the files had been saved to the right place on my computer successfully.
```
NormScore_reloaded = pd.read_pickle("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Saved Dataframes\\pickles\\df_scores_normalized.pkl")
Master_cln_reloaded = pd.read_pickle("F:\\Docs\\personal\\projects\\Sentiment Analysis Project\\Saved Dataframes\\pickles\\df_master_cln.pkl")
```

### Step 18
Step 18 takes the dataframes that contain the genre information for each set of lyrics (the cleaned master dataframe) and the one that contains the normalized scores (each called "master_cln_reloaded" and "NormScore_reloaded respectivley after having been reloaded in the previous option step), and merges them on thier shared "Row_ID" columns. This new dataframe is the second to final one generated in the course of this project and it is called "df_final". It contains 12 columns (the "Row_ID", a column for the genre lable, and a column for each of the normalized emotion scores).
`df_final = NormScore_reloaded.merge(Master_cln_reloaded[["Row_ID","Genre_Clean"]], on="Row_ID", how="left")`

### Step 19
This penultimate is a simple step to group the entries in the final dataframe just created by genre, producing the dataframe that is used to build the heatmap in the next step. This is saved as a dataframe called "df_heatmap" and it contains a row for each genre looked at in this project, and the mean score of each emotional category.
`df_heatmap = df_final.groupby("Genre_Clean")[emo_tag_list].mean()`

### Step 20
This final step uses seaborn to build a heatmap that visualizes the data in the "df_heatmap" dataframe generated in the last step.
```
plt.figure(figsize=(12, 6))
sns.heatmap(
	df_heatmap,
	cmap = "YlGnBu",
	annot = True,
	fmt = ".2f",
	cbar = "True")

plt.title("Average Emotional Profile of Genre")
plt.xlabel("Emotion")
plt.ylabel("Genre")
plt.tight_layout()
plt.show()
```
