# Setup
All the code is located in `main_notebook.ipynb`.
1. Install the packages into a new environment using `conda env create -f environment.yml -n antipodes_agent`.
2. There are 2 configurable variables in Cell 2 of the notebook.
    - BACKTEST_DATE: The as of date for the backtest.
    - LOAD_STOCKDATA: Whether to load stock data from CSV or API. Default is to load via API.
3. Run all cells in order to run the agents and compute the backtest.
# Design
## Valuation Agent
- The valuation agent computes the 5 day momentum (weekly) and volatility which is then annualised.
- From the annualised values, a sharpe ratio is calculated and a buy hold sell signal is generated with the mapping: x < -1: sell, -1 <= x <= 1: hold, x > 1: buy.
## Sentiment Agent
- The sentiment agent utilises the `ProsusAI/finbert` model from HuggingFace (https://huggingface.co/ProsusAI/finbert).
- The labels that are generated when the model is run over the summary are used as the signals with the mapping: Negative = sell, Neutral = hold, Positive = Buy.
## Fundamental Agent
- 4 Metrics are used for the Fundamental Agent.
    1. Operating Income Growth - Tracks the trajectory of profit growth.
    2. Operating Margin Pct Change - Tracks how efficiently the company generates income.
    3. Operating Cash Flow Pct Change - Tracks the change of cash flow for the company.
    4. Receivables Turnover Pct Change - Tracks the changes in cash collection for the company.
- The raw percentage values are passed into a sigmoid function to compress down the range of values to between 0 and 1. A range of k values are used to moderate how aggressively the function is used to compress down the range.
- They are then equally weighted to form the final signal where: x < 0.33 (sell), 0.33 <= x <= 0.67 (hold), x > 0.67 (buy).

## Co-ordinator Rules
- The co-ordinator maps the buy/hold/sell recommendations to +1/0/-1 and sums them together.
- The final signal is buy (x > 0), hold (x = 0), sell (x < 0).
# Backtest
- The backtest is from the as of date to `2025-09-12`.
- Price data is only loaded from `2025-06-01` to `2025-09-12` so if you require to run a backtest prior to this date, please adjust the `start_date` in cell 3.
- Total Cumulative Return and Sharpe Ratio (average return/standard deviation) is output into `Output/performance.csv`. 
# Assumptions & Limitations
- There is at least one buy recommendation for the backtest date
- Number of trading days in a year is 252
# Time Accounting
1. Data Load: 1 hour
2. Creating Sample Data: 1 hour
3. Valuation Agent: 1 hour
4. Sentiment Agent: 30 minutes
5. Fundamental Agent: 1 hour 
6. Co-ordinator: 30 minutes
7. Backtest: 1 hour

Total: 6 - 7 hours
# AI-tool Usage
Extensive use of GitHub Copilot and ChatGPT was used to generate the code. Typical usage would be to write out the code to implement an idea that I thought of.
ChatGPT was also used to create some of the 2 line summaries that were fed into the Sentiment agent.

# Tests
2 Tests are located at the end of `main_notebook.ipynb`
1. Test `get_portfolio_weights` calculating the portfolio weights correctly
2. Test `get_portfolio_weights` reutrning no rows if there are no buys
