import re
import tweepy
from tweepy import OAuthHandler
from textblob import TextBlob

class TwitterClient(object):
    '''
    Generic Twitter Class for sentiment analysis.
    '''
    def __init__(self):
        '''
        Class constructor or initialization method.
        '''
        
        consumer_key = 'XXXXXXXXXXXXXXXXXXXXXXXX'
        consumer_secret = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXX'
        access_token = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXX'
        access_token_secret = 'XXXXXXXXXXXXXXXXXXXXXXXXX'

        
        try:
            
            self.auth = OAuthHandler(consumer_key, consumer_secret)
            
            self.auth.set_access_token(access_token, access_token_secret)
            
            self.api = tweepy.API(self.auth)
        except:
            print(&quot;Error: Authentication Failed&quot;)

    def clean_tweet(self, tweet):
        '''
        Utility function to clean tweet text by removing links, special characters
        using simple regex statements.
        '''
        return ' '.join(re.sub(&quot;(@[A-Za-z0-9]+)|([^0-9A-Za-z \t])
                                    |(\w+:\/\/\S+)&quot;, &quot; &quot;, tweet).split())

    def get_tweet_sentiment(self, tweet):
        '''
        Utility function to classify sentiment of passed tweet
        using textblob's sentiment method
        '''
        
        analysis = TextBlob(self.clean_tweet(tweet))
        
        if analysis.sentiment.polarity &gt; 0:
            return 'positive'
        elif analysis.sentiment.polarity == 0:
            return 'neutral'
        else:
            return 'negative'

    def get_tweets(self, query, count = 10):
        '''
        Main function to fetch tweets and parse them.
        '''
        
        tweets = []

        try:
            
            fetched_tweets = self.api.search(q = query, count = count)

            
            for tweet in fetched_tweets:
                
                parsed_tweet = {}

              
                parsed_tweet['text'] = tweet.text
                
                parsed_tweet['sentiment'] = self.get_tweet_sentiment(tweet.text)

                
                if tweet.retweet_count &gt; 0:
                    
                    if parsed_tweet not in tweets:
                        tweets.append(parsed_tweet)
                else:
                    tweets.append(parsed_tweet)

            
            return tweets

        except tweepy.TweepError as e:
            
            print(&quot;Error : &quot; + str(e))

def main():
    
    api = TwitterClient()
    
    tweets = api.get_tweets(query = 'Donald Trump', count = 200)

    
    ptweets = [tweet for tweet in tweets if tweet['sentiment'] == 'positive']
    
    print(&quot;Positive tweets percentage: {} %&quot;.format(100*len(ptweets)/len(tweets)))
    
    ntweets = [tweet for tweet in tweets if tweet['sentiment'] == 'negative']
    
    print(&quot;Negative tweets percentage: {} %&quot;.format(100*len(ntweets)/len(tweets)))
    
    print(&quot;Neutral tweets percentage: {} % \
        &quot;.format(100*(len(tweets) -(len( ntweets )+len( ptweets)))/len(tweets)))

    
    print(&quot;\n\nPositive tweets:&quot;)
    for tweet in ptweets[:10]:
        print(tweet['text'])

    
    print(&quot;\n\nNegative tweets:&quot;)
    for tweet in ntweets[:10]:
        print(tweet['text'])

if __name__ == &quot;__main__&quot;:
  
    main()
