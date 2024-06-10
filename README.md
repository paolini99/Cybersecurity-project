# Tweet_popularity
Prediction of the popularity of a tweet about food recipes 

## Disclaimer
We will use the following features (extracted for each tweet) to train the models:
  - POS tagging (counting for each sentence the most significative pos tags)
  - number of hastags
  - number of quotation marks
  - number of exclamation marks
  - length of plain text
  - number of urls
  - number of emojis
  - number of tags

## Credentials
We will use secret tokens and secret api keys (stored in a credentials.txt file).
Get your own if you want to run the find_data.ipynb script

## How to prepare dataset
- with find_data.ipynb retrieve tweets from desired pages.
- with features_extractor.ipynb extract relevant features from raw dataset.
- note that ther is a pre-retrieved dataset!

## How to train the models
- just run the notebooks/training.ipynb script and see the results
