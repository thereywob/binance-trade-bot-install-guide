# binance-trade-bot-install-guide

> Installation and set-up guide for [edeng23/binance-trade-bot](https://github.com/edeng23/binance-trade-bot)'s automated cryptocurrency trading bot. There are lots of resources for debugging on the original repository. If you're having issues getting your bot running, click the link above or see the original README.md inserted at the bottom of this file.

## Step 1 - Create a Binance Account

- Go to [binance.us](https://www.binance.us) (USA users only) and create an account
- Enable Two-Factor Authentication
- Buy some of your preferred **_bridge coin_**
  - The bot will use the bridge coin to begin buying coins when we start. If you are unsure about what a bridge coin is, or which one to use, refer to the **HOW?** section in the original README.md [here](https://github.com/edeng23/binance-trade-bot), or at the bottom of this file. We'll use USDT in this example.

## Step 2 - Install Raspberry Pi OS

- Raspberry Pi Imager - [Download Here](https://www.raspberrypi.org/software/)

I recommend the Lite version as it only has the bare minimum amount of software pre-installed and keeps plenty of space on the microSD card.

## Step 3 - Set-up Raspberry Pi OS

1. Connect the Pi to the internet
2. Change the default login password
3. Run the following commands to ensure your OS is up to date:

```shell
sudo apt-get update
sudo apt-get upgrade
```

## Step 4 - Install necessary software

### Install Git and clone the repository

```shell
sudo apt install git
git clone https://github.com/edeng23/binance-trade-bot.git
```

### Install Python 3.8

- [How to install Python 3.8 on Raspberry Pi (Raspbian)](https://installvirtual.com/how-to-install-python-3-8-on-raspberry-pi-raspbian/) _[This might take a while]_

Enter `cd` after finishing the tutorial to return to the home directory

### Install pip for Python3

`sudo apt install python3-pip`

### Install Python dependencies

```shell
cd binance-trade-bot
pip3 install -r requirements.txt
```

## Step 5 - Configure the bot

### Create and edit user.cfg file

```shell
touch user.cfg
sudo nano user.cfg
```

Copy and paste the following block into the file we just created. Be sure to check **User.cfg Guidelines** below to make sure all the fields match your needs.

```shell
[binance_user_config]
api_key=
api_secret_key=
current_coin=
bridge=USDT
tld=us
hourToKeepScoutHistory=1
scout_multiplier=5
scout_sleep_time=5
strategy=default
```

#### User.cfg Guidelines:

-   **api_key** - Binance API key generated in the Binance account setup stage.
-   **api_secret_key** - Binance secret key generated in the Binance account setup stage.
-   **current_coin** - This is your starting coin of choice. This should be one of the coins from your supported coin list. If you want to start from your bridge currency, leave this field empty - the bot will select a random coin from your supported coin list and buy it.
-   **bridge** - Your bridge currency of choice. Notice that different bridges will allow different sets of supported coins. For example, there may be a Binance particular-coin/USDT pair but no particular-coin/BUSD pair.
-   **tld** - 'com' or 'us', depending on your region. Default is 'com'.
-   **hourToKeepScoutHistory** - Controls how many hours of scouting values are kept in the database. After the amount of time specified has passed, the information will be deleted.
-   **scout_multiplier** - Controls the value by which the difference between the current state of coin ratios and previous state of ratios is multiplied. For bigger values, the bot will wait for bigger margins to arrive before making a trade.
-   **strategy** - The trading strategy to use. See [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) for more information

### Create Binance API Key

Go to Binance and create a new API key under [API Management](https://www.binance.us/en/usercenter/settings/api-management). Insert the API key and secret key into the user.cfg file we just created. Leave the current_coin field blank (bot automatically assigns a coin).

Then enter CONTROL+X, then hit Y, and then ENTER to save and exit the user.cfg file

### Modify supported_coin_list

`sudo nano supported_coin_list`

Make sure the coins on this list are compatable with your chosen bridge coin. You can check [here](https://www.binance.us/en/markets). Since we're using USDT as the bridge coin, we're looking for coins that end with **/USDT**. For example **ADA/USDT** would work, but **ALGO/BUSD** would not work unless we change the bridge coin to BUSD.

Save changes and exit the file using CONTROL+X, then Y, then ENTER.

## Step 6 - Start the bot

We're now ready to start the bot for the first time. Make sure you're in the binance-trade-bot directory using:

```shell
cd
cd binance-trade-bot
```

To start the bot enter `python3 -m binance_trade_bot`

If all goes well you should have the following messages:

- *INFO - Starting*
- *INFO - Creating database schema if it doesn't already exist*
- *INFO - Setting initial coin to ____*
- *INFO - Purchasing <____> to begin trading*
- *INFO - BUY QTY ####*
- *INFO - Bought ____*
- *INFO - Ready to start trading*

## Step 7 - Notifications

Apprise allows the bot to send notifications to all of the most popular notification services available such as: Telegram, Discord, Slack, Amazon SNS, Gotify, etc... [Read more](https://github.com/edeng23/binance-trade-bot#notifications-with-apprise).

## Step 8 - Errors & Debugging

If your bot is throwing errors or otherwise not working properly, please reference the original repository [here](https://github.com/edeng23/binance-trade-bot). They have a Discord server with lots of helpful hints about how to fix common issues.

## Disclaimer

This project is a remix/installation guide for the original repository [edeng23/binance-trade-bot](https://github.com/edeng23/binance-trade-bot). The purpose of this project is to create an easier step-by-step guide to installing and configuring this software. I am not, under any circumstances, responsible for any of the actions you take whilst using this software or installation guide.

> -End of installation guide-

# Original README.md
From [edeng23/binance-trade-bot](https://github.com/edeng23/binance-trade-bot)
## Why?

This project was inspired by the observation that all cryptocurrencies pretty much behave in the same way. When one spikes, they all spike, and when one takes a dive, they all do. _Pretty much_. Moreover, all coins follow Bitcoin's lead; the difference is their phase offset.

So, if coins are basically oscillating with respect to each other, it seems smart to trade the rising coin for the falling coin, and then trade back when the ratio is reversed.

## How?

The trading is done in the Binance market platform, which of course, does not have markets for every altcoin pair. The workaround for this is to use a bridge currency that will complement missing pairs. The default bridge currency is Tether (USDT), which is stable by design and compatible with nearly every coin on the platform.

<p align="center">
  Coin A → USDT → Coin B
</p>

The way the bot takes advantage of the observed behaviour is to always downgrade from the "strong" coin to the "weak" coin, under the assumption that at some point the tables will turn. It will then return to the original coin, ultimately holding more of it than it did originally. This is done while taking into consideration the trading fees.

<div align="center">
  <p><b>Coin A</b> → USDT → Coin B</p>
  <p>Coin B → USDT → Coin C</p>
  <p>...</p>
  <p>Coin C → USDT → <b>Coin A</b></p>
</div>

The bot jumps between a configured set of coins on the condition that it does not return to a coin unless it is profitable in respect to the amount held last. This means that we will never end up having less of a certain coin. The risk is that one of the coins may freefall relative to the others all of a sudden, attracting our reverse greedy algorithm.

## Binance Setup

-   Create a [Binance account](https://accounts.binance.com/en/register).
-   Enable Two-factor Authentication.
-   Create a new API key.
-   Get a cryptocurrency. If its symbol is not in the default list, add it.

## Tool Setup

### Install Python dependencies

Run the following line in the terminal: `pip install -r requirements.txt`.

### Create user configuration

Create a .cfg file named `user.cfg` based off `.user.cfg.example`, then add your API keys and current coin.

**The configuration file consists of the following fields:**

-   **api_key** - Binance API key generated in the Binance account setup stage.
-   **api_secret_key** - Binance secret key generated in the Binance account setup stage.
-   **current_coin** - This is your starting coin of choice. This should be one of the coins from your supported coin list. If you want to start from your bridge currency, leave this field empty - the bot will select a random coin from your supported coin list and buy it.
-   **bridge** - Your bridge currency of choice. Notice that different bridges will allow different sets of supported coins. For example, there may be a Binance particular-coin/USDT pair but no particular-coin/BUSD pair.
-   **tld** - 'com' or 'us', depending on your region. Default is 'com'.
-   **hourToKeepScoutHistory** - Controls how many hours of scouting values are kept in the database. After the amount of time specified has passed, the information will be deleted.
-   **scout_multiplier** - Controls the value by which the difference between the current state of coin ratios and previous state of ratios is multiplied. For bigger values, the bot will wait for bigger margins to arrive before making a trade.
-   **strategy** - The trading strategy to use. See [`binance_trade_bot/strategies`](binance_trade_bot/strategies/README.md) for more information

#### Environment Variables

All of the options provided in `user.cfg` can also be configured using environment variables.

```
CURRENT_COIN_SYMBOL:
SUPPORTED_COIN_LIST: "XLM TRX ICX EOS IOTA ONT QTUM ETC ADA XMR DASH NEO ATOM DOGE VET BAT OMG BTT"
BRIDGE_SYMBOL: USDT
API_KEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
API_SECRET_KEY: NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j
SCOUT_MULTIPLIER: 5
SCOUT_SLEEP_TIME: 5
TLD: com
STRATEGY: default
```

### Paying Fees with BNB
You can [use BNB to pay for any fees on the Binance platform](https://www.binance.com/en/support/faq/115000583311-Using-BNB-to-Pay-for-Fees), which will reduce all fees by 25%. In order to support this benefit, the bot will always perform the following operations:
-   Automatically detect that you have BNB fee payment enabled.
-   Make sure that you have enough BNB in your account to pay the fee of the inspected trade.
-   Take into consideration the discount when calculating the trade threshold.

### Notifications with Apprise

Apprise allows the bot to send notifications to all of the most popular notification services available such as: Telegram, Discord, Slack, Amazon SNS, Gotify, etc.

To set this up you need to create a apprise.yml file in the config directory.

There is an example version of this file to get you started.

If you are interested in running a Telegram bot, more information can be found at [Telegram's official documentation](https://core.telegram.org/bots).

### Run

`python -m binance_trade_bot`

### Docker

The official image is available [here](https://hub.docker.com/r/edeng23/binance-trade-bot) and will update on every new change.

```shell
docker-compose up
```

If you only want to start the SQLite browser

```shell
docker-compose up -d sqlitebrowser
```

## Developing

To make sure your code is properly formatted before making a pull request,
remember to install [pre-commit](https://pre-commit.com/):

```shell
pip install pre-commit
pre-commit install
```

The scouting algorithm is unlikely to be changed. If you'd like to contribute an alternative
method, [add a new strategy](binance_trade_bot/strategies/README.md).

## Support the Project

<a href="https://www.buymeacoffee.com/edeng" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

## Join the Chat

-   **Discord**: [Invite Link](https://discord.gg/m4TNaxreCN)

## FAQ

A list of answers to what seem to be the most frequently asked questions can be found in our discord server, in the corresponding channel.

<p align="center">
  <img src = "https://usercontent2.hubstatic.com/6061829.jpg">
</p>

## Disclaimer

This project is for informational purposes only. You should not construe any
such information or other material as legal, tax, investment, financial, or
other advice. Nothing contained here constitutes a solicitation, recommendation,
endorsement, or offer by me or any third party service provider to buy or sell
any securities or other financial instruments in this or in any other
jurisdiction in which such solicitation or offer would be unlawful under the
securities laws of such jurisdiction.

If you plan to use real money, USE AT YOUR OWN RISK.

Under no circumstances will I be held responsible or liable in any way for any
claims, damages, losses, expenses, costs, or liabilities whatsoever, including,
without limitation, any direct or indirect damages for loss of profits.
# End of Original README.md
