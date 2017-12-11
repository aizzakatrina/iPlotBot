# social_analytics-PlotBot
see the Twitter bot at @iPlotBot [https://twitter.com/iPlotBot](https://twitter.com/iPlotBot) 
```
# dependencies 
import pandas as pd 
import matplotlib.pyplot as plt 
import json
import numpy as np
import time 
import datetime
import tweepy
import apikeys

# Import and Initialize Sentiment Analyzer
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

# Twitter API keys
consumer_key = apikeys.consumer_key
consumer_secret = apikeys.consumer_secret
access_token = apikeys.access_token 
access_secret = apikeys.access_secret

# Twitter credentials
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())

# target term to search 
search_term = "iPlotBot"

# define function
def iPlotBot_analyzer():

    # list of accounts already analyzed to prevent abuse
    previously_analyzed = ['iPlotBot']

    # loop through 5 pages
    for pages in range(5):

        # retreive tweets
        public_tweets = api.search(search_term, rpp=100, count=100, result_type="recent")

        # loop through all tweets
        for tweet in public_tweets["statuses"]:

            # get user account to anlyze
            to_analyze = tweet['entities']['user_mentions'][1]['screen_name']

            # get user account who submitted request
            requested_by = tweet['user']['screen_name']

            # if user to analyze is not private
            try: 

                # if target user not previously analyzed
                if to_analyze not in previously_analyzed:

                    # if user to analyze has 500+ texts
                    try:

                        # add user to previously analyzed list
                        previously_analyzed.append(to_analyze)

                        # Variables for holding sentiments
                        sentiments = []

                        # loop through 25 pages of tweets (total 500 tweets)
                        for page_number in range(25):

                            # get all tweets from user
                            user_tweets = api.user_timeline(to_analyze, page=page_number, result_type="recent")

                            # loop through all tweets by user
                            for user_tweet in user_tweets:

                                # get status text
                                text = user_tweet['text']

                                # Run Vader Analysis on each tweet
                                scores = analyzer.polarity_scores(text)

                                # Add each value to the appropriate array
                                sentiments.append(scores)

                        # store analysis in dataframe
                        df = pd.DataFrame(sentiments)

                        # plot analysis 
                        x = np.arange(-500,0)
                        y1 = df['compound']
                        fig, ax1 = plt.subplots()
                        ax1.plot(x, y1, alpha=0.5, linewidth=1,marker='o', label='@' + to_analyze)
                        ax1.set_xlabel('Tweets Ago')
                        ax1.set_ylabel('Tweet Polarity')
                        plt.title('Sentiment Analysis of Tweets by @' + to_analyze + '\n (as of %s' % datetime.date.today() + ')')
                        box = ax1.get_position()
                        ax1.set_position([box.x0, box.y0, box.width * 0.9, box.height])
                        ax1.legend(title="Tweets", loc='upper right', bbox_to_anchor=(1.25, 1))
                        plt.savefig('results.png')

                        # update status
                        api.update_with_media('results.png', 'New tweet analysis: @' + to_analyze + ' (Thanks @' + requested_by + ' for the submission!)')

                    except ValueError:
                        pass

            except tweepy.error.TweepError:
                pass

# run function                       
while True:
    iPlotBot_analyzer()
    time.sleep(300)
```   
