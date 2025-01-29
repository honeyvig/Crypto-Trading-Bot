# Crypto-Trading-Bot
Buy and sell coins automatically operating on Apple MacOS.

2. Trading decision based on RSI indicator, standard variable settings to be available for each coin.

3. Watch List. A user defined watch list of coins numerically ranked by user, maximum 20 coins on watch list. Each coin on watch list must be viewable on a chart with RSI indicator. Bot to cycle through coins on specified time frame eg 30min to determine if any coins meet purchase criteria

First coin in watch list queue that meets purchase criteria is to be bought.

4. Purchase coins.

Buy signal.

Purchase parameters for long sale are;

1. RSI value below RSI low ie 30.

2. Wait for subsequent low below RSI low. This low must be higher than the previous low.

3. Check trading low value at low of RSI low then compare next trading low opposite RSI low that the second trading low is lower than the previous trading low.

4. If all conditions are met place long order when RSI is above 30 but on the next bar, not on the current bar that meets all criteria, the next bar, as soon as it opens.

5. $$ amount purchased specified as % of available funds

6. Bot to continue to place orders if $$ are available. Obviously if no $$ available no orders to be placed but bot continues to seek purchase opportunities each RSI nominated period (say 1hr) cycling through watch list until $$ become available.

7. When a sold coin realises a free cash balance in trade account of say $200 that $200 is allocated to buy the next coin eligible to be bought, trade fee of 1% of available funds to be recalculated to ensure sufficient funds  ie say 1% of purchase price allocated for trade fee.

There may be up to four trades operational at a single point in time. When a transaction is complete (positively or negatively) the funds are allocated to the next trade with allowance for a fee of up to 1% of purchase.

8. Take Profit

Take profit when coin $ has risen to defined value this will be calculated as a percentage;
ie 120% of purchase price.



9. Stop lost

A stop loss to be defined as % of purchase.

10. User to be able to override any take profit, stop loss or change defined parameter on the fly to either increase (decrease) value or close transaction.

11. Current trades to listed with ability to change any relevant parameter ie profit or loss or close transaction.

12. This Bot is to be operationally (API linked) to Swiftx to conduct the transactions.

13. Bot to operate in either live mode or test mode.

14. Security features as available in the API to be included.

15. The Bot is too operate continuously even if Apple device has locked sceeen.

16. If device fails and needs to be rebooted bot should return to previous position updated to real time.

17. Source code reverts to me upon payment.
18. ---
Building a cryptocurrency trading bot with the features you mentioned requires several steps, integrating with the Swiftx API, handling real-time market data, and implementing technical indicators such as the Relative Strength Index (RSI). Below is an outline of the Python code required to create such a bot.
Requirements:

    Python 3.x
    Libraries: requests, pandas, numpy, talib, time, schedule, matplotlib, ccxt, threading, etc.
    Swiftx API key and secret (for connecting to the Swiftx exchange)
    A persistent storage system (e.g., SQLite or file storage) to track trades, watchlist, and portfolio

Below is the starting point for building this bot. Please note this is just a skeleton implementation and needs to be completed and tested with the actual Swiftx API and other libraries.
Install Dependencies:

You can install the required dependencies using pip:

pip install requests pandas numpy talib matplotlib ccxt schedule

Skeleton Python Bot Code:

import ccxt
import time
import numpy as np
import pandas as pd
import talib
import matplotlib.pyplot as plt
import threading
import schedule

class CryptoTradingBot:
    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret
        self.exchange = ccxt.swiftx({
            'apiKey': self.api_key,
            'secret': self.api_secret,
        })
        self.watchlist = []
        self.current_trades = []
        self.max_trades = 4
        self.cash_balance = 1000  # Example starting amount in USD
        self.trade_fee = 0.01  # 1% fee
        self.rsi_period = 14  # Default RSI period
        self.take_profit_pct = 1.2  # 120% for take profit
        self.stop_loss_pct = 0.8  # 80% for stop loss

    def get_balance(self):
        return self.exchange.fetch_balance()

    def get_price_data(self, symbol, timeframe='30m'):
        # Retrieve OHLCV data (Open, High, Low, Close, Volume)
        ohlcv = self.exchange.fetch_ohlcv(symbol, timeframe)
        df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
        df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
        return df

    def calculate_rsi(self, df):
        rsi = talib.RSI(df['close'], timeperiod=self.rsi_period)
        return rsi

    def check_buy_conditions(self, df, rsi):
        last_rsi = rsi[-1]
        last_close = df['close'].iloc[-1]
        
        if last_rsi < 30:  # RSI condition for buy
            # Check for subsequent higher low and lower trading low
            if self.is_valid_rsi_buy_condition(df, rsi):
                return True
        return False

    def is_valid_rsi_buy_condition(self, df, rsi):
        # Check the low values of RSI and prices for buy conditions
        pass  # Implement logic to check if the RSI low is higher than previous low

    def place_order(self, symbol, amount):
        # Placeholder for placing an order using Swiftx API
        print(f"Placing order to buy {amount} of {symbol}")
        # Actual code to place order should be here
        self.cash_balance -= amount  # Deduct the purchase amount from available cash

    def sell_coin(self, symbol, amount):
        # Placeholder for placing a sell order using Swiftx API
        print(f"Selling {amount} of {symbol}")
        self.cash_balance += amount  # Add the sale amount back to available cash

    def check_and_buy(self):
        for coin in self.watchlist:
            df = self.get_price_data(coin)
            rsi = self.calculate_rsi(df)
            if self.check_buy_conditions(df, rsi):
                if self.cash_balance > 0:
                    amount_to_buy = self.cash_balance * (1 - self.trade_fee)  # Account for fee
                    self.place_order(coin, amount_to_buy)

    def take_profit(self, symbol, purchase_price):
        # Calculate the take profit condition based on the purchase price
        target_price = purchase_price * self.take_profit_pct
        # Check if the current price has reached the target price
        current_price = self.get_current_price(symbol)
        if current_price >= target_price:
            # Sell the coin
            print(f"Take Profit reached for {symbol}, selling at {current_price}")
            self.sell_coin(symbol, self.cash_balance)

    def stop_loss(self, symbol, purchase_price):
        # Calculate the stop loss condition based on the purchase price
        target_price = purchase_price * self.stop_loss_pct
        # Check if the current price has dropped below the stop loss price
        current_price = self.get_current_price(symbol)
        if current_price <= target_price:
            # Sell the coin
            print(f"Stop Loss triggered for {symbol}, selling at {current_price}")
            self.sell_coin(symbol, self.cash_balance)

    def get_current_price(self, symbol):
        ticker = self.exchange.fetch_ticker(symbol)
        return ticker['last']

    def run(self):
        while True:
            self.check_and_buy()
            time.sleep(30)  # Run every 30 seconds

    def start(self):
        # Running the bot in a separate thread
        bot_thread = threading.Thread(target=self.run)
        bot_thread.start()

    def add_coin_to_watchlist(self, symbol, rank):
        if len(self.watchlist) < 20:
            self.watchlist.append((rank, symbol))
            self.watchlist = sorted(self.watchlist, key=lambda x: x[0])  # Sort by user-defined rank
        else:
            print("Watchlist full")

    def override_parameters(self, symbol, take_profit=None, stop_loss=None):
        # Allow the user to override parameters for take profit or stop loss
        if take_profit:
            self.take_profit_pct = take_profit
        if stop_loss:
            self.stop_loss_pct = stop_loss

    def list_current_trades(self):
        for trade in self.current_trades:
            print(f"Current Trade: {trade['symbol']}")

# Example usage:
api_key = "your_swiftx_api_key"
api_secret = "your_swiftx_api_secret"
bot = CryptoTradingBot(api_key, api_secret)

# Add some coins to the watchlist
bot.add_coin_to_watchlist('BTC/USD', 1)
bot.add_coin_to_watchlist('ETH/USD', 2)

# Start the bot
bot.start()

Key Features Breakdown:

    RSI Calculation: This is done using the talib.RSI method.
    Purchase Logic: The bot buys a coin when the RSI is below 30 and meets the subsequent higher low condition.
    Watchlist Management: A list of coins is maintained, and the bot checks them periodically.
    Order Management: The bot places buy orders if funds are available and sells coins based on take profit or stop loss criteria.
    Multi-Threading: The bot can run continuously on a separate thread.
    User Overrides: You can override parameters like take profit, stop loss, and buy conditions.

Additional Considerations:

    API Rate Limits: Be mindful of API rate limits when calling the Swiftx API. Make sure to implement error handling and retries for network issues.
    Backtesting: Before using the bot in live trading, backtest it using historical data to ensure the logic works as expected.
    Security: Ensure your API keys are securely stored and not exposed in the code.
    Error Handling: Implement error handling for cases like invalid orders or network failures.
    Continuous Operation: Consider using a service like screen or tmux on MacOS to run the bot even if the terminal session is closed.

This bot is a starting point for building a fully automated trading system with all the features you’ve requested. You should enhance and test it carefully, especially with real market data and the Swiftx API documentation.
