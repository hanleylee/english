# Building a Robust Backtesting Framework — Event-Driven Architecture

Did you conduct a backtest to confirm the potential profitability of your strategies or ideas?

Backtesting isn't simply about comparing it with actual results. It involves a series of decisions that can greatly influence the outcome and decide whether you make a profit or incur a loss.

An important question to consider is: What kind of live performance can we expect based on a backtest?

![img](https://miro.medium.com/v2/resize:fit:1400/0*JEcjaXo5gGRpOcQa.png)

"Many sources recommend easy backtesting. But if it were that easy, we'd all be rich.

There are different methods to conduct backtesting. Here are the main factors that typically affect backtested performance:

- **Backtest methodology** — with all methods addressing overfitting, data mining and data snooping;
- **Model drag** — for training vs live differences, any regime and behavioral effects;
- **Trading costs** — mostly for accounting for bid-ask spread, liquidity and market constrains;
- **Structural issues** — with design of strategy and portfolio selection / optimisation and any market access.

While coding our backtest framework, we will explore all these points. For now, as an example let me explain how we are managing transaction costs, with a goal to reduce these expenses:

- **rebalancing frequency** — reduce trading through less frequent rebalancing e.g. from daily to weekly, hence it may increase our risk and efficiency;
- **threshold / inertia trading** — to decide to trade if some parameters are above some threshold, hence it may limit our positions in some cases;
- **liquidity focus** — to trade only liquid assets to avoid any less liquid markets / sectors;
- **screening / filter** — to use better filtering scheme to use better trades, hence it can limit our positions and trades.

## Other approach

It's evident that many backtested strategies appear promising and ready-to-invest because they are often presented in a positive light in numerous articles. Developing our own backtest frameworks, as opposed to using existing ones, grants full control over the code and its logic. However, running a basic backtest without considering factors such as market trends, significant events, company fundamentals, or implementing constraints like stop-loss/take-profit or time events, can lead to skewed results.

## Get Jakub Polec's stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

Many backtesting frameworks have a steep learning curve and limited flexibility. I've used several of these frameworks and scripts, and here are my impressions:

- **Backtrader** is a flexible, powerful Python library for backtesting trading strategies, although challenging to learn due to complex syntax. Its development has slowed, resulting in fewer updates and new features.
- **QuantConnect** is an algorithmic trading platform offering backtesting and live trading. Its complexity and coding requirements can pose a learning challenge. While backtesting is free, live trading and some advanced features may incur costs. And the more you want, the more you have to pay.
- **Zipline** — an algorithmic trading library built by Quantopian. Setting it today is challenging, especially on Windows, due to its dependencies and installation process. Similar to Backtrader, Zipline is no longer actively maintained, leading to potential issues with compatibility and missing out on recent market features and data integrations.
- **MetaTrader 5** — a popular trading platform with built-in backtesting capabilities for automated trading systems. It uses its own scripting language (MQL), which can be a hurdle for those accustomed to more common languages like Python.
- **NinjaTrader** -a trading platform that provides backtesting tools and supports automated trading systems. While NinjaTrader offers a free version, accessing advanced features and live trading capabilities can be expensive. Similar to QuantConnect.
- **Quantlib** — On the other hand, this is a free/open-source library that offers tools for modeling, trading, and risk management in real-life scenarios. However, the downside is that QuantLib requires an extensive understanding of quantitative finance and primarily C++ programming.

## Main Framework

To build a reliable backtest engine, we focus on several reality framework checklists. These are often overlooked by simplified backtest scripts, resulting in unrealistic outcomes:

1. Strategy Rationale — What makes the strategy profitable?
2. Target Universe — Where do we plan to apply the strategy?
3. Data Acquisition — How do we obtain accurate data with the required detail?
4. Testing Methodology — What is our backtest procedure and what additional modules/engines are used?
5. Testing/Sample — When and in what situations is the strategy profitable?
6. Portfolio Construction/Optimisation — How are trades identified? How do we adjust based on different situations/events?
7. Transaction Costs — What costs are involved and how are they managed?
8. Execution — How is the strategy implemented in a live environment?

## BaseStrategy — main class for Backtest Framework

Below is the code for the main class, called BaseStrategy(), which loads the strategy from a config JSON file and initializes the strategy state:

```python
def _initialize_engines(self) -> None:
  config = self.config
  engine_params = [
     ('mdp', MarketDataProvider, 'market_data'),
     ('mra', MarketRegimeAnalyzer, 'market_regime'),
     ('ep', EventsProcessor, 'events'),
     ('fp', ForecastPredictor, 'forecast', self.risk_management),
     ('tcc', TradingCostCalculator, 'trading_cost'),
     ('vf', VolatilityForecaster, 'volatility'),
     ('rm', RiskManager, 'risk'),
     ('pa', PerformanceAnalyst, 'performance'),
     ('prg', PerformanceReportGenerator, 'performance_report'),
     ('plt', ProfitLossTracker, 'profitloss'),
     ('te', TradeExecutor, 'trade_execution'),
    ]
  for attr, cls, param_key, *extra_args in engine_params:
   setattr(self, attr, cls(config.get(param_key, {}), *extra_args))

  rules = config.get('event', {}).get('signal_events', {}).get('rules', {})
  self.strategies = list(rules.keys())
  self.po = PortfolioOptimizer(
   initial_capital=self.initial_capital,
   transaction_costs=self.tcc,
   strategies=self.strategies,
   rules=rules
  )
```

part of the config:

```json
"benchmark": "SPY",
   "data_manager_params": {
       "source": "yfinance",
       "granularity": "1d"
   },
   "frequency": {
       "frequency": "daily",
       "rebalance_frequency": "monthly",
       "rebalance_strategy": "periodic"
   },
```

BaseStrategy then initializes other required engines, which can be thought of as specialized modules that add specific functionalities. This modular approach makes it much easier to remove, pause, or modify any of them as needed:

- **MarketDataProvider** manages market data for trading strategies. It fetches data from sources like local MongoDB or EOD providers. It provides necessary data, such as historical prices and volumes, to the backtesting framework. It also handles data retrieval, preprocessing, and storage.
- **MarketRegimeAnalyzer** determines the current market regime based on factors like price trends and volatility. It aids in making informed trading decisions and adapting strategies according to the prevailing market conditions, thereby optimizing performance.
- **EventsProcessor** generates trading events like market, order, and signal events. Market events reflect market changes, order events handle trade orders, and signal events are based on trading strategy and indicate position changes. The `EventsProcessor` manages these events, enabling the strategy to respond to market changes and execute trades.
- **RiskManager** manages risk, determines position sizes, and ensures the portfolio's overall risk remains within limits. It considers market volatility, portfolio diversification, and risk tolerance to define position sizes, thus protecting the portfolio from excessive losses and maintaining the trading strategy within set risk parameters.
- **ForecastPredictor** uses machine learning to forecast returns and volatility from historical market data, guiding trading decisions and optimizing strategy performance.
- **TradingCostCalculator** calculates all trade-related costs such as execution fees, commissions, swap fees, etc. It provides a realistic estimate of total trading expenses, ensuring performance metrics and profitability analysis are accurate.
- **VolatilityForecaster** focuses on modeling and forecasting volatility using various methods, for example historical volatility, realized volatility, Garman-Klass volatility, and Parkinson volatility. Uses some statistical techniques to anticipate future fluctuations in asset prices, which helps the trading strategy in adapting to shifting market conditions and decisions regarding risk exposure.
- **PerformanceAnalyst** executes in-depth analysis of the trading strategy, calculating metrics like ROI, Sharpe ratio, drawdown, and risk-adjusted returns. It offers insights into historical performance, helping identify improvement areas. By evaluating performance across various market conditions and time periods, it aids decision-making and strategy optimization.
- **PortfolioOptimizer** manages portfolio construction and optimizes asset allocation, considering risk preferences, investment objectives, and diversification needs. It employs advanced optimization techniques, such as mean-variance optimization or risk parity, to create efficient portfolios that maximize returns while minimizing risk.
- **PerformanceReportGenerator** produces final performance reports for the trading strategy.
- **ProfitLossTracker** aggregates trade results, calculating key metrics like gross profit and win/loss ratio for strategy evaluation, risk management, and decision-making on usage or improvements.
- **TradeExecutor** executes trades based on strategy signals. It works with the exchange API specifically for Interactive Brokers (IBKR) to manage trade positions, considering order types, price limits, etc. It oversees open positions and manages trade closure based on set rules or manual interventions.

## Event-Driven Architecture in Backtesting

I'll concentrate on EventProcessor and the event-driven architecture (EDA), a core pattern in backtesting frameworks. EDA is used by libraries like QuantConnect, Zipline, and VectorBT, and by many trading firms and hedge funds.

Event-driven architecture is ideal for backtesting trading strategies as it accurately reflects the nature of financial markets. Everything in finance, from price changes to order executions, is an event. By treating the backtesting process as a series of these events, the system can react to them in real-time, instead of adhering to a linear or procedural flow.

**Key Components of an Event-Driven Backtesting System**

1. **Events**: In an event-driven backtesting system, events are the fundamental building blocks that represent various occurrences during the trading process. Common event types include:

- **Market Events**: Represent the arrival of new market data, such as price updates, volume changes, or economic news releases.
- **Signal Events**: Represent trading signals generated by a strategy, such as buy or sell signals based on technical indicators or other rules.
- **Order Events**: Represent orders to be placed in the market, such as stop-loss orders, take-profit orders, or more complex order types like bracket orders or iceberg orders.

1. **Event Loop**: The event loop is the central component that continuously checks for and processes events as they occur. This loop ensures that events are handled in a timely and efficient manner, mimicking the real-time nature of financial markets.
2. **Event Handlers**: Event handlers are responsible for responding to specific types of events. For example, a market data handler might process market events and update the historical data, while an order execution handler might process order events and simulate order execution based on predefined rules or models.
3. **Event Producers**: Event producers are components that generate events based on specific conditions or rules. Strategies, data feeds, and other components can act as event producers, generating signal events, market events, or order events based on their logic.
