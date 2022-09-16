+++
title = "Introduction to Natural Language Processing(NLP): Jobs in Kenya Project"
date = "2020-11-14 16:23:13"
tags = [
    "data",
    "nlp",
]
+++

Twitter Analysis of Jobs posted and its legitimacy.
<!--more-->

2020 was a tough year where due to the ongoing COVID-19 crisis, most individuals have lost their jobs. Twitter is one of the platforms individuals go to seek job opportunities, hence I decided to create a model to filter relevant job opportunities.
 
Find the code at_ [https://github.com/BethanyJep/JobsInKE](https://github.com/BethanyJep/JobsInKE)

## Prequisites

Here is what is used for the project:

- Twitter Developer Account
- Python: some of the libraries I used include sci-kit-learn, pandas, NumPy, matplotlib, seaborn, regex, NLTK, Tweepy and regex
- Notebook: I used [Google Colab](http://colab.research.google.com/) for this project.

## Getting the Data

First, I extracted jobs posted on Twitter using specific keywords, hashtags and accounts. You need to create a developer account on Twitter at [http://developers.twitter.com/](http://developers.twitter.com/). It took me roughly one hour to get my request approved back in 2019, this might have changed. After creating an account, you can then create an app, which will give you your API keys and tokens.

Now let's get to the code. First we fetch data from Twitter:

```python
#Getting the necessary libraries (if tweepy is not installed, you can install it using: pip install tweepy)
import tweepy as tweepy
from tweepy import Stream
from tweepy import OAuthHandler

#consumer keys
consumerKey = "xxxxxxxxxx"
consumerSecret = "xxxxxxx"
accessKey = "xxxxxxxxxx"
accessSecret = "xxxxxxxxx"

auth = OAuthHandler(consumerKey, consumerSecret)
auth.set_access_token(accessKey, accessSecret)

api = tweepy.API(auth)
```

In scraping data, I did it in two ways:

- data from specific [hashtags and keywords](https://github.com/BethanyJep/JobsInKE/blob/main/Jobs%20in%20Ke%20Twitter%20Data%20Scraping.ipynb): '#ikokazi', 'Iko Kazi KE', '#ikokazike', '#IkoKaziKE', '#ikokaziKE',' #ajiraKE' '#PataKaziKE'

```python
search_words = [ '#ikokazi', 'Iko Kazi KE', '#ikokazike', '#IkoKaziKE', '#ikokaziKE',' #ajiraKE' '#PataKaziKE']
date_since = "2020-7-10"

Our_tweets = []
for search_word in search_words:
  # filters out retweets to remove duplicates
  new_search = search_word + " -filter:retweets"
  tweets = tweepy.Cursor(api.search,
                  q=new_search, since=date_since,   tweet_mode = 'extended' ).items(300)
  Our_tweets.append(tweets)
Our_tweets
```

Twitter has a restriction has to how far you can scrape data using keywords hence the date since. Consequently, to avoid duplicates, I made sure to filter retweets.

- data from [specific accounts](https://github.com/BethanyJep/JobsInKE/blob/main/Influencers%20of%20Jobs%20in%20Ke.ipynb): 'ikokaziKE, ikokaziKenya, KaziQuest and AjiraKE'

```python
tweets = api.user_timeline(screen_name='ikokaziKenya',   count=500,   include_rts = True,  tweet_mode = 'extended' )
tweets
```

A short explanation on the above: count is how many tweets I hope to retrieve from the specific user, I opted to include retweets as I noticed most of these users rarely tweet about job opportunities but rather retweet from other accounts and tweet mode extended ensures the tweets are not truncated.

Something worth noting is am Kenyan and I wanted the tweets from my specific location therefore opted to Using Swahili keywords and hashtags as a filter to what tweets I wanted to extract.

## Data Cleaning

First, I created a function to remove URLs.

```python
import re #regular expression
def remove_url(txt):
  return " ".join(re.sub("([^0-9A-Za-z \t])|(\w+:\/\/\S+)", "", txt).split())
```

I filtered the data collected to the tweets, username and specific location.

```python
users_locs=[]
all_split_tweets=[]
for tweets in Our_tweets:
  for tweet in tweets:
      tweets_no_url = remove_url(tweet.full_text)
      all_split_tweets.append(tweets_no_url.lower())
      users_locs.append([tweet.user.screen_name, tweet.user.location, tweets_no_url])

users_locs
```

With a somewhat clean data, I went ahead and merged the different data sets together.

```python
#exporting data in colab: adding google drive
from google.colab import drive
drive.mount('/content/drive')
#exporting data from colab: exporting
df.to_csv('datafinl2.csv')
!cp datafinl2.csv "drive/My Drive/"

#Importing data from colab (ensure you add google drive)
df = pd.read_csv('/content/drive/My Drive/tweets.csv') #I did this for all the data sets

#Merging the datasets
dtframes = [df, df2, df1, df3, df4, df5, df6, df7, df8, df9, df10, df11]
tweets_df = pd.concat(dtframes)
tweets_df
```

As I scraped using both Keywords and Usernames, there were bound to be many duplicates within the data set, here is how I resolved that:

```python
#removing tweets that have been duplicated
tweets_df = tweets_df.drop_duplicates(subset=['Tweet'])
tweets_df
```

Then, I pre-processed the data, emoticons, stopwords and ensuring the tweets were in English. While cleaning data, I used the tweet-preprocessor, which I recommend 10/10, library to remove URLs, emojis, and numbers.

```python
pip install tweet-preprocessor #install tweet-processor library
#using text preprocessor to clean data

def pre_proc(text):
    text = p.clean(text)
    return text

tweets_df['Tweet_clean'] = tweets_df['Tweet'].apply(lambda x: pre_proc(x))
tweets_df.head()
```

Consequently, the NLTK library came in handy in removing non-English words.

```python
words = set(nltk.corpus.words.words())

def retain_eng(text):
    text  = " ".join(w for w in nltk.wordpunct_tokenize(text) if w.lower() in words or not w.isalpha())
    return text

tweets_df['Tweet'] = tweets_df['Tweet'].apply(lambda x: retain_eng(x))
tweets_df.head(10)
```

Removing stop words came next. I had to tokenize the data, which is splitting the tweets to words so as to easily remove the stopwords:

```python
#setting stopwords
stopword = nltk.corpus.stopwords.words('english')
#splitting text to tokens
def tokenization(text):
    text = re.split('\W+', text)
    return text

tweets_df['Tweet_token'] = tweets_df['Tweet_clean'].apply(lambda x: tokenization(x.lower()))

#removing stopwords
def remove_stopwords(text):
    text = [word for word in text if word not in stopword]
    return text

tweets_df['Tweet_nonstop'] = tweets_df['Tweet_token'].apply(lambda x: remove_stopwords(x))
tweets_df.head()
```

## Data Visualization

Using the clean data, I analyzed major keywords and patterns used, using exploratory visualizations including WordClouds, factor plots and bar graph counter of top words and patterns.

### WordClouds

Visualize frequency of how words appear using a word cloud.

```
#visualizing tweet data; most common wordsstopwords = set(STOPWORDS)
tweets_combo = ' '
# instantiate a word cloud objectfor val in tweets_df['Tweet']:
          # typecaste each val to string val = str(val)
      # split the value tokens = val.split()
          # Converts each token into lowercase æææfor i in range(len(tokens)):
        tokens[i] = tokens[i].lower()
    tweets_combo += " ".join(tokens)+" "

df_wc = WordCloud(
    background_color='white',
    max_words=10000,
    stopwords=stopwords
)
# tweets_combo
# generate the word cloud
df_wc.generate(tweets_combo)

plt.figure(figsize = (8, 8), facecolor = None)
plt.imshow(df_wc)
plt.axis("off")
plt.tight_layout(pad = 0)

plt.show()
```

![Introduction%20to%20NLP%20Jobs%20in%20Kenya%20project%20b65a8eabed4340b8a8647feb9043f31a/Untitled.png](<https://github.com/BethanyJep/RandomDesignRepo/blob/master/Untitled%20(28).png?raw=true>)

### Countplot

Counting the most common words in the tweets.

```python
# Helper function
def plot_10_most_common_words(count_data, count_vectorizer):
    import matplotlib.pyplot as plt
    words = count_vectorizer.get_feature_names()
    total_counts = np.zeros(len(words))
    for t in count_data:
        total_counts+=t.toarray()[0]

    count_dict = (zip(words, total_counts))
    count_dict = sorted(count_dict, key=lambda x:x[1], reverse=True)[0:20]
    words = [w[0] for w in count_dict]
    counts = [w[1] for w in count_dict]
    x_pos = np.arange(len(words))

    plt.figure(2, figsize=(15, 15/1.6180))
    plt.subplot(title='20 most common words')
    sns.set_context("notebook", font_scale=1.25, rc={"lines.linewidth": 2.5})
    sns.barplot(x_pos, counts, palette='husl')
    plt.xticks(x_pos, words, rotation=90)
    plt.xlabel('words')
    plt.ylabel('counts')
    plt.show()
# Initialise the count vectorizer with the English stop words
count_vectorizer = CountVectorizer(stop_words='english')
# Fit and transform the tweets
count_data = count_vectorizer.fit_transform(tweets_df['Tweet'])
# Visualise the 10 most common words
plot_10_most_common_words(count_data, count_vectorizer)
```

result:

![Introduction%20to%20NLP%20Jobs%20in%20Kenya%20project%20b65a8eabed4340b8a8647feb9043f31a/Untitled%201.png](<https://github.com/BethanyJep/RandomDesignRepo/blob/master/Untitled%20(27).png?raw=true>)

### Counter

I used this to find the most common patterns on how words occur in the tweets.

```python
#Finding the most common patternsCounter = Counter(tweets_df['Tweet_nonstop'])
most_occur = Counter.most_common(30)
most_occur
```

![Introduction%20to%20NLP%20Jobs%20in%20Kenya%20project%20b65a8eabed4340b8a8647feb9043f31a/Untitled%202.png](<https://github.com/BethanyJep/RandomDesignRepo/blob/master/Untitled%20(26).png?raw=true>)

## Data Modelling

From there, I filtered data using regex, then created a model that predicts whether the tweet extracted was a job posted or just some random tweet using a trending hashtag.

```python
#Finding patterns in tweets using regex
def map_freq(text):
    freq = "qualified|looking|inspired|people|within|work|new|company|interested|comment|ready|open|job|help|application|check|apply|post|vacancy"
    x = re.findall(freq, text)
    if x:
        return 1
    else:
        return 0


tweets_df['Tweet_freq'] =[ map_freq(text) for text in tweets_df['Tweet_nonstop']]
tweets_df
```

From the results, I visualised the spam and not spam tweets occurrence

```python
#visualizing the spam vs not spam tweets
sns.factorplot(x="Tweet_freq", data=tweets_df, kind="count", size=6, aspect=1.5, palette="PuBuGn_d")
plt.show();
```

![Introduction%20to%20NLP%20Jobs%20in%20Kenya%20project%20b65a8eabed4340b8a8647feb9043f31a/Untitled%203.png](<https://github.com/BethanyJep/RandomDesignRepo/blob/master/Untitled%20(25).png?raw=true>)

I then split my data to test and training sets:

```python
#identifying the label
target = tweets_df['Tweet_freq']
#splitting data to test and training sets
X_train, X_test, y_train, y_test = train_test_split(tweets_df['Tweet_nonstop'], target, test_size=0.20, random_state=100)

print(df.shape); print(X_train.shape); print(X_test.shape)
```

Prior to modelling, I used the term frequency-inverse document frequency to give a statistical measure to the tweets.

```python
#converting text with tfidf vectorizer
vectorizer_tfidf = TfidfVectorizer(stop_words='english', max_df=0.7)

train_tfIdf = vectorizer_tfidf.fit_transform(X_train.values.astype('U'))

test_tfIdf = vectorizer_tfidf.transform(X_test.values.astype('U'))

print(vectorizer_tfidf.get_feature_names()[:10])
```

The performance of the models are as follows: Naive Bayes Classifier Accuracy - 73.32% Random Forest Classifier Accuracy - 97.65%.

```python
#Naive bayes classifier
nb_classifier = MultinomialNB()

nb_classifier.fit(train_tfIdf, y_train)

pred2 = nb_classifier.predict(test_tfIdf)
print(pred2[:30])

# Evaluating the model
# Calculate the accuracy score: score
accuracy_tfidf = metrics.accuracy_score(y_test, pred2)
print(accuracy_tfidf)

Conf_metrics_tfidf = metrics.confusion_matrix(y_test, pred2, labels=[1, 2])
print(Conf_metrics_tfidf)
```

```python
#Random forest classifier
classifier = RandomForestClassifier(n_estimators = 10, criterion = 'entropy', random_state = 100)

classifier.fit(train_tfIdf, y_train)

predRF = classifier.predict(test_tfIdf)
print(predRF[:10])

# Calculate the accuracy score
accuracy_RF = metrics.accuracy_score(y_test, predRF)
print(accuracy_RF)

Conf_metrics_RF = metrics.confusion_matrix(y_test, predRF, labels=[1, 0])
print(Conf_metrics_RF)
```

This model could be useful for one to filter out job postings, to only view relevant posts.

## Future of the Project

Having not previously completed a Data Science project alone, I was on the verge of quitting Data Science as imposter syndrome was slowly creeping up inside me. However, working on this project reignited my passion. From here I hope to implement my project using a web application that will show all opportunities.

## Resources

Google Colab: [https://colab.research.google.com/](https://colab.research.google.com/)

Twitter Developers: [http://developers.twitter.com/](http://developers.twitter.com/)

Tweet Preprocessor Library: [https://pypi.org/project/tweet-preprocessor/](https://pypi.org/project/tweet-preprocessor/)
