# BuyBitcoinElonMusk
This bot is designed to buy Bitcoin when Elon Musk tweets about it.

http://chatalistic.com/fact-1528-program-a-trading-bot-to-buy-bitcoin-when-elon-musk-tweets-about-it.html



Program a trading bot to buy Bitcoin when Musk Tweets about it
Why would someone do this? 
The answer is a simple one – the Techno King of Tesla has a history of influencing crypto markets whenever he tweets about them, to the point where a movement in the market is almost expected when he picks up is phone and starts expressing his opinions on the blockchain technology over twitter.

By creating a crypto trading bot that buys bitcoin every time the Tesla boss tweets about it you can rest assured that you are going to catch a VIP seat on the rocket that will slingshot right past the moon and make its way directly to Mars, where Elon spends most of the summer months due to its cold weather and dry climate.

Will this actually work? 
The quick answer is “not sure” – as no one tested this strategy before. The longer answer is probably – as long as we’re talking about Bitcoin. Statistically speaking, regardless of the time you bought your bitcoin, you are most likely in profit (excluding the recent all time high around the time of writing). 

So if nothing else, you will at least spice up your BTC HODLing strategy with a bit of help from Elon. This article won’t go into a detailed analysis to show whether this strategy actually works or not. This article is about building it for fun, but it does have serve as a powerful reminder of just how many resources we have at our disposal and that you can build just about any crypto trading bot you can think of. 

You will also be able to see and use the code so you can test it or improve it.

How to set your bitcoin bot up
What this article is focused on is the actual technical building of the bitcoin trading bot, and how to set it up in a safe test environment, so let’s get to it. 

You will need the following resources:
A MetaTrader5 account
A demo account with XBTFX so you can safely test your strategy
A Twitter Dev Account
A Tweepy API account
Setting up MetaTrader5 and XBTFX
As the name suggests, MT5 is a platform which supports multiple brokers along with detailed technical analysis – the main reason to start your crypto bot building journey with MT5 is due to it’s easy integration with Python and out-of-the-box support for a demo or virtual account so that you can test in a safe demo environment.

There are detailed instructions on how to install and configure MetaTrader5 as well as the XBTFX crypto broker  in the previous post that covers how to build a crypto trading bot in python, so we’ll only briefly going over these steps in this article. If you need more information on how to do it, as well as why those two platforms were chosen, please refer back to the linked article above.

Start by downloading and installing MetaTrader5 and create an account on their platform. The next thing you need is a broker that you can place your trades with – I recommend XBTFX as they offer the most crypto-pairs of all brokers that work with the MT5 terminal. Register with XBTFX and create a demo account.

You can now connect to your demo account via MT5 by navigating over to File > Open an Account and searching for XBTFX. If you have registered using the referral link above you will need to select “Connect to Existing Account”, otherwise proceed to create a new account.

Apply for a Dev Account with Twitter
Before you can use Twitter’s API or the Tweepy Python module, you need a developer account with Twitter. Luckily the application process is quick and easy, and you will probably be accepted as long as you describe why you need the access to the Twitter API.

Nativate over to twitter’s dev platform and click Apply in the top right corner of the navigation menu.



On the next page click Apply for a Developer account and you will be prompted to sign in with your twitter account.



Follow the registration process and explain your intentions with the API 



After you have completed all the necessary information, it may take anywhere between a couple of hours to a couple of days before you can get access to the platform. In my experience it was only a few hours.

Once your dev account is ready navigate over to the Projects & Apps tab open Project 1, if this is not available go ahead and create one. Under your project go to Keys and Tokens and generate the following (make sure to save them or you will need to regenerate the keys!):



Defining bot parameters
The bot will open a buy position on bitcoin every time Elon mentions bitcoin in his tweet
Take profit is set to 10% and stop loss to 5%
The bitcoin bot will not place another trade if there is already an active trade (can be adjusted) 
Coding for your bitcoin trading bot
Preliminary set-up
First off you need to import the MetaTrader5 and Tweepy modules using PyPi.

pip install tweepy
pip install MetaTrader5
pip install --upgrade MetaTrader5
The next step is to import these modules along with a few others into your Python interpreter.

#Twitter Scraper module
import tweepy
from tweepy import OAuthHandler

#dates module
from datetime import datetime, date
from itertools import count
import time
import re


#trading terminal
import MetaTrader5 as mt5
We now need to store the secret keys and tokens that you generated using the Twitter Dev platform in order to use them with Tweepy.

# Store Twitter credentials from dev account
consumer_key = "CONSUMER_KEY"
consumer_secret = "CONSUMER_SECRET"
access_key = "API_KEY"
access_secret = "API_SECRET"

# Pass twitter credentials to tweepy via its OAuthHandler
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_key, access_secret)
api = tweepy.API(auth)
In the last part of the preliminary set up you need to connect to the MT5 terminal, store your account’s equity and define the trading instrument that we will be working with – in this case it’s Bitcoin. We will also create a short list of keywords to query Elon’s last tweet against.

# connect to the trade account without specifying a password and a server
mt5.initialize()

# account number in the top left corner of the MT5 terminal window
# the terminal database password is applied if connection data is set to be remembered
account_number = 555
authorized = mt5.login(account_number)

if authorized:
    print(f'connected to account #{account_number}')
else:
    print(f'failed to connect at account #{account_number}, error code: {mt5.last_error()}')

# store the equity of your account
account_info = mt5.account_info()
if account_info is None:
    raise RuntimeError('Could not load the account equity level.')
else:
    equity = float(account_info[10])
#crypto sign and keywords
CRYPTO = 'BTCUSD'
keywords = ['Bitcoin', 'bitcoin', 'BITCOIN', 'btc', 'BTC']
Getting Elon’s latest tweet
With all the preliminary stuff out of the way, it’s time to focus on the cool parts of this bot. Let’s start by getting Elon’s last tweet with Tweepy as shown below in the get_elons_tweet() function.

During testing, emojis and other invalid characters would break the script, so each tweet is re formatted to only contain alpha-numeric characters.

#Get Technoking's latest tweet
def get_elons_tweet():
    """Get Elon's last tweet by user ID - retry until tweepy returns tweet"""
    tweets = tweepy.Cursor(api.user_timeline,id="44196397", since=date.today(), tweet_mode='extended').items(1)

    #remove all invalid characters
    elons_last_tweet = [re.sub('[^A-Za-z0-9]+', ' ', tweet.full_text) for tweet in tweets]

    #re-try until it returns a value - tweepy API fails to return the tweet sometimes
    while not elons_last_tweet:
        tweets = tweepy.Cursor(api.user_timeline,id="44196397", since=date.today(), tweet_mode='extended').items(1)
        elons_last_tweet = [re.sub('[^A-Za-z0-9]+', ' ', tweet.full_text) for tweet in tweets]
    return elons_last_tweet[0]
Logic check and preparing the trading request
Now that we have Elon’s last tweet we can start preparing the logic and the trading request in function trade(). For more information regarding the format of the trade request, have a look at the MT 5 documentation.

what_musk_said contains the last tweet and the logic will check whether any of the keywords defined in our keywords variable above are present in Elon’s tweet. If that is true, the bitcoin trading bot will place a buy order on bitcoin with instant execution. In case it’s false it will simply return to us the tweet.

#buy bitcoin
def trade():
    """Check if Musk mentioned bitcoin and open a buy position if so"""
    what_musk_said = get_elons_tweet()

    # used to check if a position has already been placed
    positions = mt5.positions_get(symbol=CRYPTO)
    orders = mt5.orders_get(symbol=CRYPTO)
    symbol_info = mt5.symbol_info(CRYPTO)
    price = mt5.symbol_info_tick(CRYPTO).bid

    # perform logic check
    if any(keyword in what_musk_said for keyword in keywords):
        print(f'the madlad said it - buying some!')

        # prepare the trade request
        if not mt5.initialize():
            raise RuntimeError(f'MT5 initialize() failed with error code {mt5.last_error()}')

        # check that there are no open positions or orders
        if len(positions) == 0 and len(orders) < 1:
            if symbol_info is None:
                print(f'{CRYPTO} not found, can not call order_check()')
                mt5.shutdown()

            # if the symbol is unavailable in MarketWatch, add it
            if not symbol_info.visible:
                print(f'{CRYPTO} is not visible, trying to switch on')
                if not mt5.symbol_select(CRYPTO, True):
                    print('symbol_select({}}) failed, exit', CRYPTO)

            #this represents 5% Equity. Minimum order is 0.01 BTC. Increase equity share if retcode = 10014
            lot = float(round(((equity / 5) / price), 2))

            # define stop loss and take profit
            sl = price - (price * 5) / 100
            tp = price + (price * 10) / 100
            request = {
                'action': mt5.TRADE_ACTION_DEAL,
                'symbol': CRYPTO,
                'volume': lot,
                'type': mt5.ORDER_TYPE_BUY,
                'price': price,
                'sl': sl,
                'tp': tp,
                'magic': 66,
                'comment': 'python-buy',
                'type_time': mt5.ORDER_TIME_GTC,
                'type_filling': mt5.ORDER_FILLING_IOC,
            }

            # send a trading request
            result = mt5.order_send(request)

            # check the execution result
            print(f'1. order_send(): by {CRYPTO} {lot} lots at {price}')

            if result.retcode != mt5.TRADE_RETCODE_DONE:
                print(f'2. order_send failed, retcode={result.retcode}')

            #print the order result - anything else than retcode=10009 is an error in the trading request.
            print(f'2. order_send done, {result}')
            print(f'   opened position with POSITION_TICKET={result.order}')

        else:
            print(f'BUY signal detected, but {CRYPTO} has {len(positions)} active trade')

    else:
        print(f'He did not say it, he said: {what_musk_said}')
Putting it all together
We now need to decide how often we should be iterating through the code below. By default, it pull and analyse Elon’s last tweet once every 5 seconds, but this can be adjusted in the time.sleep function below.

#execute code every 5 seconds
if __name__ == '__main__':
    print('Press Ctrl-C / Ctrl-Q to stop.')
    for i in count():
        trade()
        print(f'Iteration {i}')
        time.sleep(5)
Additional Resources:
PyPi Installer
GitHub Repo
Tweepy Documentation
MetaTrader5 Python Documentation
It was a fun project work on and I hope that you enjoyed this article. Please subscribe to the newsletter for more awesome content delivered straight to your inbox!
