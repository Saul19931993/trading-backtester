import streamlit as st
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

# App title
st.set_page_config(page_title="Trading Backtester", layout="centered")
st.title("📈 Trading Strategy Backtester")

# Input section
ticker = st.text_input("Ticker Symbol", "AAPL")
start_date = st.date_input("Start Date", datetime(2020, 1, 1))
end_date = st.date_input("End Date", datetime.today())

st.subheader("📋 Strategy Rules")
buy_pct = st.number_input("Buy if price increased by (%) over X days", value=5.0)
buy_window = st.number_input("X = Days to look back", value=3)
sell_pct = st.number_input("Sell if price drops by (%) from buy price", value=3.0)

st.subheader("💰 Capital Settings")
starting_capital = st.number_input("Starting Capital ($)", value=10000.0)
buy_per_trade = st.number_input("Amount to Buy per Trade ($)", value=1000.0)
max_invested = st.number_input("Maximum Capital Invested at a Time ($)", value=5000.0)

# Button
if st.button("Run Backtest"):
    df = yf.download(ticker, start=start_date, end=end_date)
    if df.empty:
        st.error("No data found. Check ticker or date range.")
        st.stop()

    df["Buy"] = False
    df["Sell"] = False

    capital = starting_capital
    invested = 0
    positions = []
    portfolio_values = []
    trade_log = []

    for i in range(buy_window, len(df)):
        date = df.index[i]
        price = df["Close"].iloc[i]

        # Check Buy condition
        past_price = df["Close"].iloc[i - buy_window]
        change_pct = ((price - past_price) / past_price) * 100

        if change_pct >= buy_pct and capital >= buy_per_trade and invested + buy_per_trade <= max_invested:
            shares = buy_per_trade / price
            positions.append({
                "buy_price": price,
                "shares": shares,
                "date": date
            })
            capital -= buy_per_trade
            invested += buy_per_trade
            df.at[date, "Buy"] = True
            trade_log.append({"Type": "BUY", "Date": date, "Price": round(price, 2), "Shares": round(shares, 4)})

        # Check Sell condition
        for pos in positions[:]:
            drop_pct = ((price - pos["buy_price"]) / pos["buy_price"]) * 100
            if drop_pct <= -sell_pct:
                proceeds = pos["shares"] * price
                capital += proceeds
                invested -= pos["shares"] * pos["buy_price"]
                df.at[date, "Sell"] = True
                trade_log.append({"Type": "SELL", "Date": date, "Price": round(price, 2), "Shares": round(pos["shares"], 4)})
                positions.remove(pos)

        total_value = capital + sum(pos["shares"] * price for pos in positions)
        portfolio_values.append({"Date": date, "Value": total_value})

    # Final exit
    final_price = df["Close"].iloc[-1]
    for pos in positions:
        proceeds = pos["shares"] * final_price
        capital += proceeds
        invested -= pos["shares"] * pos["buy_price"]
        trade_log.append({"Type": "SELL (END)", "Date": df.index[-1], "Price": round(final_price, 2), "Shares": round(pos["shares"], 4)})

    final_value = capital
    portfolio_df = pd.DataFrame(portfolio_values).set_index("Date")
    trade_df = pd.DataFrame(trade_log)

    st.subheader("💹 Results")
    st.write(f"**Final Portfolio Value:** ${final_value:,.2f}")
    st.write(f"**Total Return:** {((final_value - starting_capital) / starting_capital * 100):.2f}%")

    if not portfolio_df.empty:
        years = (portfolio_df.index[-1] - portfolio_df.index[0]).days / 365.25
        cagr = ((final_value / starting_capital) ** (1 / years) - 1) * 100
        st.write(f"**CAGR (Compounded Annual Growth Rate):** {cagr:.2f}%")

        st.line_chart(portfolio_df["Value"], use_container_width=True)

    st.subheader("📜 Trade Log")
    st.dataframe(trade_df)
