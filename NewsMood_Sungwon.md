
# News Mood
##### Sungwon Byun

### Observations
1. CBS and BBC showed the most positive sentiment score of 0.34 and 0.13, respectively.
2. CNN, Fox, and New York Times showed negative sentiment scores, which suggests their aggressive reputations.
3. Fox and CNN shows a fairly neutral distribution, leaning towards negative. 


```python
import tweepy
import json
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from datetime import datetime
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
from textblob import TextBlob
%matplotlib inline

analyzer = SentimentIntensityAnalyzer()

# Twitter API Keys
consumer_key = "Ed4RNulN1lp7AbOooHa9STCoU"
consumer_secret = "P7cUJlmJZq0VaCY0Jg7COliwQqzK0qYEyUF9Y0idx4ujb3ZlW5"
access_token = "839621358724198402-dzdOsx2WWHrSuBwyNUiqSEnTivHozAZ"
access_token_secret = "dCZ80uNRbFDjxdU2EckmNiSckdoATach6Q8zb7YYYE5ER"

# Setup Tweepy API Authentication
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())
```


```python
# Assign twitter accounts
#Pull into a DataFrame the tweet's source acount, its text, its date, and its 
# compound, positive, neutral, and negative sentiment scores.
accounts = ['BBC','CBS','CNN','FoxNews','nytimes']
news = ['BBC','CBS','CNN','Fox','New York Times']
colors = ['lightblue','green','red','blue','yellow']
columns = ['account','date','text','compound','positive','neutral','negative']

tweets = {}
compound = {}
overall_compound = []
positive = {}
neutral = {}
negative = {}
tweets = {}
dates = {}

df = pd.DataFrame(columns=[columns])
```


```python
for account in accounts:
    for x in range(5):
        public_tweets = api.user_timeline(account,page=x)
        
        for tweet in public_tweets:
            comp = analyzer.polarity_scores(tweet['text'])['compound']
            pos = analyzer.polarity_scores(tweet['text'])['pos']
            neu = analyzer.polarity_scores(tweet['text'])['neu']
            neg = analyzer.polarity_scores(tweet['text'])['neg']
            time = tweet['created_at']
            text = tweet['text']
            
            if account in compound:
                compound[account].append(analyzer.polarity_scores(tweet['text'])['compound'])
                positive[account].append(analyzer.polarity_scores(tweet['text'])['pos'])
                neutral[account].append(analyzer.polarity_scores(tweet['text'])['neu'])
                negative[account].append(analyzer.polarity_scores(tweet['text'])['neg'])
                dates[account].append(tweet['created_at'])
                tweets[account].append(tweet['text'])
            else:
                compound[account] = [analyzer.polarity_scores(tweet['text'])['compound']]
                positive[account] = [analyzer.polarity_scores(tweet['text'])['pos']]
                neutral[account] = [analyzer.polarity_scores(tweet['text'])['neu']]
                negative[account] = [analyzer.polarity_scores(tweet['text'])['neg']]
                dates[account] = [tweet['created_at']]
                tweets[account] = [tweet['text']]
            
            current_df = pd.DataFrame([[account,time,text,comp,pos,neu,neg]],columns=columns)
            df = df.append(current_df,ignore_index=True)
    overall_compound.append(np.mean(compound[account]))

df.to_csv('NewsMood.csv')
```


```python
now = datetime.now()
todays_date = now.strftime('%m/%d/%y')
todays_date
```




    '01/14/18'




```python
xaxis = range(1,len(compound[account])+1)
sns.set()
for x in range(len(accounts)):
    plt.scatter(xaxis,compound[accounts[x]],alpha=0.75,edgecolor='black',linewidth=1,facecolor=colors[x],s=85,)

plt.legend(news,title='Media Sources',loc='upper left',bbox_to_anchor=(1,1))
plt.title('Sentiment Analysis of Media Tweets ('+todays_date+')')
plt.ylabel('Tweet Polarity')
plt.xlabel('Tweets Ago')
plt.yticks(np.arange(-1.00,1.01,0.5))
plt.gca().invert_xaxis()
plt.savefig('Sentiment Analysis of Media Tweets.png')
plt.show()
```


![png](output_5_0.png)



```python
plt.close()
sns.set_style('dark')

bars = plt.bar(news,overall_compound,width=1,color=colors,linewidth=1)
for k in range(5):
    bars[k].set_edgecolor('black')

for bar in bars:
    if bar.get_height() < 0:
        plt.text(bar.get_x()+bar.get_width()/3,bar.get_height()-0.023,str('{:.2f}'.format(bar.get_height())),color='black')
    else:
        plt.text(bar.get_x()+bar.get_width()/3,bar.get_height()+0.01,str('{:.2f}'.format(bar.get_height())),color='black')

plt.xlim(-0.5,4.55)
plt.ylim(-.15,0.4)
plt.title('Overall Media Sentiment on Twitter ('+todays_date+')')
plt.ylabel('Tweet Polarity')
plt.savefig('Overall Media Sentiment on Twitter.png')
plt.show()
```


![png](output_6_0.png)



```python

```




    range(0, 100)


