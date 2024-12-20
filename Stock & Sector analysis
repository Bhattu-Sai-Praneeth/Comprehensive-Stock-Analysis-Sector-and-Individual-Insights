import pandas as pd
import yfinance as yf
import plotly.graph_objects as go
from datetime import datetime

# Load sectors and their stock tickers from a CSV file
def load_sector_data(csv_file='sectors.csv'):
    try:
        df = pd.read_csv(csv_file)
        sector_stocks = {}
        for index, row in df.iterrows():
            sector_name = row[0]
            tickers = row[1:].dropna().tolist()  # Get tickers from the row, ignoring NaNs
            sector_stocks[sector_name] = tickers
        return sector_stocks
    except Exception as e:
        print(f"No sector data from CSV : {e}")
        return {}

# Function to select timeframe
def select_timeframe():
    print("Choose the timeframe for analysis:")
    print("1. 1 Day (1d)")
    print("2. 5 Days (5d)")
    print("3. 1 Month (1mo)")
    print("4. 3 Months (3mo)")
    print("5. 6 Months (6mo)")
    print("6. Year-to-Date (ytd)")
    print("7. 1 Year (1y)")
    print("8. 2 Years (2y)")
    print("9. 5 Years (5y)")
    print("10. Custom Date Range")

    timeframe_choice = input("Enter your choice: ")

    # Map user choice to Yahoo Finance period strings
    timeframe_map = {
        '1': '1d', '2': '5d', '3': '1mo', '4': '3mo', 
        '5': '6mo', '6': 'ytd', '7': '1y', '8': '2y', '9': '5y'
    }

    if timeframe_choice in timeframe_map:
        return timeframe_map[timeframe_choice]
    elif timeframe_choice == '10':
        # Custom date range
        start_date = input("Enter start date (YYYY-MM-DD): ")
        end_date = input("Enter end date (YYYY-MM-DD): ")
        try:
            datetime.strptime(start_date, '%Y-%m-%d')
            datetime.strptime(end_date, '%Y-%m-%d')
            return (start_date, end_date)
        except ValueError:
            print("Error: Invalid date format. Please use YYYY-MM-DD.")
            return None
    else:
        print("Error: Invalid choice.")
        return None

# Function to analyze individual stock with Bollinger Bands and RSI
def analyze_stock(tickerSymbol, period=None, start_date=None, end_date=None):
    try:
        tickerData = yf.Ticker(tickerSymbol)
        if period:
            tickerDf = tickerData.history(period=period)
        elif start_date and end_date:
            tickerDf = tickerData.history(start=start_date, end=end_date)
        else:
            print("Error: No valid timeframe or date range selected.")
            return None

        if tickerDf.empty:
            print(f"Error: No data available for ticker {tickerSymbol}.")
            return None

        # Calculate Bollinger Bands
        tickerDf['SMA'] = tickerDf['Close'].rolling(window=20).mean()
        tickerDf['SD'] = tickerDf['Close'].rolling(window=20).std()
        tickerDf['UB'] = tickerDf['SMA'] + 2 * tickerDf['SD']
        tickerDf['LB'] = tickerDf['SMA'] - 2 * tickerDf['SD']

        # Calculate RSI
        delta = tickerDf['Close'].diff()
        gain = delta.clip(lower=0)
        loss = -delta.clip(upper=0)
        avg_gain = gain.rolling(window=14).mean()
        avg_loss = loss.rolling(window=14).mean()
        rs = avg_gain / avg_loss
        tickerDf['RSI'] = 100 - (100 / (1 + rs))

        # Plot Bollinger Bands
        fig = go.Figure()
        fig.add_trace(go.Candlestick(
            x=tickerDf.index,
            open=tickerDf['Open'],
            high=tickerDf['High'],
            low=tickerDf['Low'],
            close=tickerDf['Close'],
            name='Candlestick'
        ))
        fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['UB'],
            line=dict(color='blue', width=1),
            name='Upper Band'
        ))
        fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['LB'],
            line=dict(color='blue', width=1),
            name='Lower Band',
            fill='tonexty', fillcolor='rgba(173, 216, 230, 0.2)'
        ))
        fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['SMA'],
            line=dict(color='red', width=1),
            name='SMA (20)'
        ))
        fig.update_layout(
            title=f'{tickerSymbol} Bollinger Bands',
            xaxis_title='Date',
            yaxis_title='Price',
            height=600
        )
        fig.show()

        # Plot RSI
        rsi_fig = go.Figure()
        rsi_fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['RSI'],
            line=dict(color='purple', width=1.5),
            name='RSI'
        ))
        rsi_fig.add_hline(y=70, line=dict(color='red', dash='dot'), annotation_text='Overbought (70)')
        rsi_fig.add_hline(y=30, line=dict(color='green', dash='dot'), annotation_text='Oversold (30)')
        rsi_fig.update_layout(
            title=f'{tickerSymbol} RSI (14)',
            xaxis_title='Date',
            yaxis_title='RSI',
            height=400
        )
        rsi_fig.show()

    except Exception as e:
        print(f"Error analyzing {tickerSymbol}: {e}")

# Function to analyze sector-wide performance with combined RSI and Bollinger Bands verdict
def analyze_sector_combined(sector_name, stock_tickers, period=None, start_date=None, end_date=None):
    sector_data = pd.DataFrame()

    for ticker in stock_tickers:
        print(f"Fetching data for {ticker}...")
        try:
            ticker_data = yf.Ticker(ticker)
            if period:
                data = ticker_data.history(period=period)
            elif start_date and end_date:
                data = ticker_data.history(start=start_date, end=end_date)
            else:
                print("Error: No valid timeframe or date range selected.")
                return None

            if not data.empty:
                sector_data[ticker] = data['Close']
            else:
                print(f"Warning: No data available for {ticker}.")
        except Exception as e:
            print(f"Error fetching data for {ticker}: {e}")

    if sector_data.empty:
        print(f"No data available for the sector {sector_name}.")
        return

    # Calculate sector average close price
    sector_data['Sector Average'] = sector_data.mean(axis=1)

    # Calculate Bollinger Bands on sector average
    sector_data['SMA'] = sector_data['Sector Average'].rolling(window=20).mean()
    sector_data['SD'] = sector_data['Sector Average'].rolling(window=20).std()
    sector_data['UB'] = sector_data['SMA'] + 2 * sector_data['SD']
    sector_data['LB'] = sector_data['SMA'] - 2 * sector_data['SD']

    # Calculate RSI on sector average
    delta = sector_data['Sector Average'].diff()
    gain = delta.clip(lower=0)
    loss = -delta.clip(upper=0)
    avg_gain = gain.rolling(window=14).mean()
    avg_loss = loss.rolling(window=14).mean()
    rs = avg_gain / avg_loss
    sector_data['RSI'] = 100 - (100 / (1 + rs))

    # Plot combined Bollinger Bands
    fig = go.Figure()
    fig.add_trace(go.Candlestick(
        x=sector_data.index,
        open=sector_data['Sector Average'], 
        high=sector_data['Sector Average'] + sector_data['SD'],  
        low=sector_data['Sector Average'] - sector_data['SD'],  
        close=sector_data['Sector Average'],  
        name='Candlestick'
    ))
    fig.add_trace(go.Scatter(
        x=sector_data.index, y=sector_data['UB'],
        line=dict(color='blue', width=1),
        name='Upper Band'
    ))
    fig.add_trace(go.Scatter(
        x=sector_data.index, y=sector_data['LB'],
        line=dict(color='blue', width=1),
        name='Lower Band',
        fill='tonexty', fillcolor='rgba(173, 216, 230, 0.2)'
    ))
    fig.add_trace(go.Scatter(
        x=sector_data.index, y=sector_data['SMA'],
        line=dict(color='red', width=1),
        name='SMA (20)'
    ))
    fig.update_layout(
        title=f'{sector_name} Combined Bollinger Bands',
        xaxis_title='Date',
        yaxis_title='Price',
        xaxis_rangeslider_visible=False,
        height=600
    )
    fig.show()

    # Plot RSI for the sector
    rsi_fig = go.Figure()
    rsi_fig.add_trace(go.Scatter(
        x=sector_data.index, y=sector_data['RSI'],
        line=dict(color='purple', width=1.5),
        name='RSI'
    ))
    rsi_fig.add_hline(y=70, line=dict(color='red', dash='dot'), annotation_text='Overbought (70)')
    rsi_fig.add_hline(y=30, line=dict(color='green', dash='dot'), annotation_text='Oversold (30)')
    rsi_fig.update_layout(
        title=f'{sector_name} Combined RSI (14)',
        xaxis_title='Date',
        yaxis_title='RSI',
        height=400
    )
    rsi_fig.show()

    # Combined Verdict: RSI + Bollinger Bands
    latest_rsi = sector_data['RSI'].iloc[-1]
    latest_close = sector_data['Sector Average'].iloc[-1]
    upper_band = sector_data['UB'].iloc[-1]
    lower_band = sector_data['LB'].iloc[-1]

    # Determine RSI verdict
    if latest_rsi > 70:
        rsi_verdict = "Not Investable (Overbought)"
    elif latest_rsi < 30:
        rsi_verdict = "Investable (Oversold)"
    else:
        rsi_verdict = "Neutral"

    # Determine Bollinger Bands verdict
    if latest_close > upper_band:
        bb_verdict = "Not Investable (Overbought)"
    elif latest_close < lower_band:
        bb_verdict = "Investable (Oversold)"
    else:
        bb_verdict = "Neutral"

    # Combine the verdicts
    if rsi_verdict == bb_verdict:
        final_verdict = rsi_verdict  # Both agree
    else:
        final_verdict = "Neutral"  # Conflict between RSI and Bollinger Bands

    print(f"Combined Verdict for {sector_name}: {final_verdict}")


# Function to analyze individual stock with Bollinger Bands and RSI
def analyze_stock(tickerSymbol, period=None, start_date=None, end_date=None):
    try:
        tickerData = yf.Ticker(tickerSymbol)
        if period:
            tickerDf = tickerData.history(period=period)
        elif start_date and end_date:
            tickerDf = tickerData.history(start=start_date, end=end_date)
        else:
            print("Error: No valid timeframe or date range selected.")
            return None

        if tickerDf.empty:
            print(f"Error: No data available for ticker {tickerSymbol}.")
            return None

        # Calculate Bollinger Bands
        tickerDf['SMA'] = tickerDf['Close'].rolling(window=20).mean()
        tickerDf['SD'] = tickerDf['Close'].rolling(window=20).std()
        tickerDf['UB'] = tickerDf['SMA'] + 2 * tickerDf['SD']
        tickerDf['LB'] = tickerDf['SMA'] - 2 * tickerDf['SD']

        # Calculate RSI
        delta = tickerDf['Close'].diff()
        gain = delta.clip(lower=0)
        loss = -delta.clip(upper=0)
        avg_gain = gain.rolling(window=14).mean()
        avg_loss = loss.rolling(window=14).mean()
        rs = avg_gain / avg_loss
        tickerDf['RSI'] = 100 - (100 / (1 + rs))

        # Plot Bollinger Bands
        fig = go.Figure()
        fig.add_trace(go.Candlestick(
            x=tickerDf.index,
            open=tickerDf['Open'],
            high=tickerDf['High'],
            low=tickerDf['Low'],
            close=tickerDf['Close'],
            name='Candlestick'
        ))
        fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['UB'],
            line=dict(color='blue', width=1),
            name='Upper Band'
        ))
        fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['LB'],
            line=dict(color='blue', width=1),
            name='Lower Band',
            fill='tonexty', fillcolor='rgba(173, 216, 230, 0.2)'
        ))
        fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['SMA'],
            line=dict(color='red', width=1),
            name='SMA (20)'
        ))
        fig.update_layout(
            title=f'{tickerSymbol} Bollinger Bands',
            xaxis_title='Date',
            yaxis_title='Price',
            height=600
        )
        fig.show()

        # Plot RSI
        rsi_fig = go.Figure()
        rsi_fig.add_trace(go.Scatter(
            x=tickerDf.index, y=tickerDf['RSI'],
            line=dict(color='purple', width=1.5),
            name='RSI'
        ))
        rsi_fig.add_hline(y=70, line=dict(color='red', dash='dot'), annotation_text='Overbought (70)')
        rsi_fig.add_hline(y=30, line=dict(color='green', dash='dot'), annotation_text='Oversold (30)')
        rsi_fig.update_layout(
            title=f'{tickerSymbol} RSI (14)',
            xaxis_title='Date',
            yaxis_title='RSI',
            height=400
        )
        rsi_fig.show()

    except Exception as e:
        print(f"Error analyzing {tickerSymbol}: {e}")

# Main function to drive the analysis
def main():
    sector_stocks = load_sector_data()

    if not sector_stocks:
        print("Error: Could not load sector data.")
        return

    print("Available Sectors:")
    for i, sector in enumerate(sector_stocks.keys(), start=1):
        print(f"{i}. {sector}")

    sector_choice = input("Select a sector by entering the number: ")
    try:
        sector_choice = int(sector_choice)
        sector_name = list(sector_stocks.keys())[sector_choice - 1]
        stock_tickers = sector_stocks[sector_name]

        print(f"Selected Sector: {sector_name}")
        timeframe = select_timeframe()

        if isinstance(timeframe, tuple):  # Custom date range
            start_date, end_date = timeframe
            analyze_sector_combined(sector_name, stock_tickers, start_date=start_date, end_date=end_date)
        else:
            analyze_sector_combined(sector_name, stock_tickers, period=timeframe)

        # Display available companies in the selected sector
        print("\nAvailable Companies in Selected Sector:")
        for ticker in stock_tickers:
            print(f"- {ticker}")

        # Option for individual stock analysis
        print("\nIndividual Stock Analysis:")
        selected_stock = input("Enter the stock ticker symbol (e.g., AAPL, TSLA): ")
        print(f"Selected Stock: {selected_stock}")

        if isinstance(timeframe, tuple):  # Custom date range
            analyze_stock(selected_stock, start_date=start_date, end_date=end_date)
        else:
            analyze_stock(selected_stock, period=timeframe)
    except Exception as e:
        print("Error: Invalid sector choice.")


        # Display available companies in the selected sector
        print("\nAvailable Companies in the Selected Sector:")
        for ticker in stock_tickers:
            print(f"- {ticker}")

        # Option for individual stock analysis
        print("\nIndividual Stock Analysis:")
        selected_stock = input("Enter the stock ticker symbol (e.g., AAPL, TSLA): ")
        print(f"Selected Stock: {selected_stock}")

        if isinstance(timeframe, tuple):  # Custom date range
            analyze_stock(selected_stock, start_date=start_date, end_date=end_date)
        else:
            analyze_stock(selected_stock, period=timeframe)
    except Exception as e:
        print("Error: Invalid sector choice.")


        # Option for individual stock analysis
        print("\nIndividual Stock Analysis:")
        selected_stock = input("Enter the stock ticker symbol (e.g., AAPL, TSLA): ")
        print(f"Selected Stock: {selected_stock}")

        if isinstance(timeframe, tuple):  # Custom date range
            analyze_stock(selected_stock, start_date=start_date, end_date=end_date)
        else:
            analyze_stock(selected_stock, period=timeframe)
    except Exception as e:
        print("Error: Invalid sector choice.")

# Run the program
if __name__ == "__main__":
    main()
