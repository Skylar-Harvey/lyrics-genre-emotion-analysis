# Hello
The purpose of this "datasets" folder is to store copies of the datasets used within this project, as well as the various saved and cleaned dataframes made along the way.
Due to the size of the datasets used, they have been stored externally instead of here. To access them, use the folllowing links:
[Click here to download the dataframes made along the way as a zip file](link)
[Click here to be rerouted back to this repository's main page. Links to the datasets used in this project can be found in the primary readme](https://github.com/Skylar-Harvey/lyrics-genre-emotion-analysis)



## data_zip contents:
The file contains four different dataframes each stored two ways. A CSV file for easy viewing or implementation in other projects, and as a .pkl file for easy implementation into coding environments.

### Dataframe I: df_master
This dataframe is the result of "step 13" of the code outlined [here](link). It contains 4 columns and 3,482,119 rows.
The columns are as follows:
- Song_Name - The ttile of the songs represented in that row
- Lyrics - A string containing the lyrics of the song
- Language - A two-letter code that insicates the language of the lyrics data
- Genre - The genres associated with the song

### Dataframe II: df_master_cln
This is a cleaned version of the above dataframe dervived from "step 14" of the code. It contains 3 columns and 3,482,119
The columns are as follows:
- Lyrics_Clean - Cleaned lyrical data. Punctuation, filler words, common words ("a", "and", "the", etc.), and other non-semantic lyric data removed from lyric strings
- Genre_Clean - Cleaned genre data. Each song, if it had more than one genre association, now has only one genre association based on first match to the list of genres analyzed in the project
- Row_ID - A unique identifier for each row. Used to run mergers with other dataframes later.

### Dataframe III: df_scores
This dataframe contains the raw scores obtained by "step 17.2". It consit of 11 columns and 3,482,119 rows.
The first column, "Row_ID" contains a uniqu identifier for that row. This is to make merging a final version of this scores dataframe to the cleaned master dataset easier.
The other 10 columns each represent one of the emotional relationships each lyrical token was scored for. These 10 emotional catagories reflect those used in the EMOlex tool used as a part of this project. They are in order: Anger, Anticipation, Disgust, Fear, Joy, Negative, Positive, Sadness, Surprise, and Trust.

### Dataframe IV: df_scores_normalized
This dataframe is exactly like the one above in its structure. I.E., 11 columns and 3,482,119 rows all labeled exactly the same as the previous.
This dataframe is different in that it contains the scores normalized accross each row. this is to ensure that each emotional category scored for for each song analyzed are all scored somewhere between 0 and 1. 
