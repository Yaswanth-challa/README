# Build a Simple Portfolio Tracker application that allows users to:
# 1. Add, view, edit, and delete stock holdings.
# 2. Track the total portfolio value based on real-time stock prices.
# 3. View a dashboard displaying key portfolio metrics (e.g., total value,top-performing stock, portfolio distribution).

# 1. Add, view, edit, and delete stock holdings.

# app.py (Flask Application)

from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///portfolio.db'
db = SQLAlchemy(app)

# Define the Stock model
class Stock(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    symbol = db.Column(db.String(10), nullable=False)
    quantity = db.Column(db.Integer, nullable=False)
    purchase_price = db.Column(db.Float, nullable=False)
    def __repr__(self):
        return f"Stock('{self.name}', '{self.symbol}', {self.quantity}, {self.purchase_price})"

@app.route('/')
def index():
    stocks = Stock.query.all()
    return render_template('index.html', stocks=stocks)

@app.route('/add', methods=['GET', 'POST'])
def add_stock():
    if request.method == 'POST':
        name = request.form['name']
        symbol = request.form['symbol']
        quantity = int(request.form['quantity'])
        purchase_price = float(request.form['purchase_price'])
        new_stock = Stock(name=name, symbol=symbol, quantity=quantity, purchase_price=purchase_price)
        db.session.add(new_stock)
        db.session.commit()
        return redirect(url_for('index'))
    return render_template('add_stock.html')

@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit_stock(id):
    stock = Stock.query.get_or_404(id)
    if request.method == 'POST':
        stock.name = request.form['name']
        stock.symbol = request.form['symbol']
        stock.quantity = int(request.form['quantity'])
        stock.purchase_price = float(request.form['purchase_price'])
        db.session.commit()
        return redirect(url_for('index'))
    return render_template('edit_stock.html', stock=stock)

@app.route('/delete/<int:id>')
def delete_stock(id):
    stock = Stock.query.get_or_404(id)
    db.session.delete(stock)
    db.session.commit()
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)


# 2. Track the total portfolio value based on real-time stock prices.

# stock_price_tracker.py

import requests

def get_stock_price(symbol):
    # Example using Alpha Vantage API
    api_key = 'your_alpha_vantage_api_key'
    url = f'https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={symbol}&interval=5min&apikey={api_key}'
    response = requests.get(url)
    data = response.json()
    if "Time Series (5min)" in data:
        latest_time = list(data["Time Series (5min)"].keys())[0]
        latest_close = data["Time Series (5min)"][latest_time]["4. close"]
        return float(latest_close)
    return None

def get_portfolio_value(stocks):
    total_value = 0
    for stock in stocks:
        current_price = get_stock_price(stock['symbol'])
        if current_price:
            total_value += current_price * stock['quantity']
    return total_value

# Example stock holdings
stocks = [{'symbol': 'AAPL', 'quantity': 10},{'symbol': 'TSLA', 'quantity': 5},{'symbol': 'GOOGL', 'quantity': 3},
]

portfolio_value = get_portfolio_value(stocks)
print(f'Total Portfolio Value: ${portfolio_value:.2f}')


# 3. View a dashboard displaying key portfolio metrics (e.g., total value,top-performing stock, portfolio distribution).

<!-- dashboard.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Portfolio Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Portfolio Dashboard</h1>
    <div>
        <h2>Total Portfolio Value: <span id="total-value"></span></h2>
        <h3>Top-Performing Stock: <span id="top-stock"></span></h3>
        <h3>Portfolio Distribution:</h3>
        <canvas id="distribution-chart" width="400" height="200"></canvas>
    </div>
    <script>
        // Sample data (replace with dynamic data from backend)
        const portfolioData = [
            {symbol: 'AAPL', quantity: 10, price: 145},
            {symbol: 'TSLA', quantity: 5, price: 720},
            {symbol: 'GOOGL', quantity: 3, price: 2800}
        ];
        // Calculate total value
        let totalValue = 0;
        portfolioData.forEach(stock => {
            totalValue += stock.price * stock.quantity;
        });
        document.getElementById('total-value').textContent = `$${totalValue.toFixed(2)}`;
        // Find the top-performing stock
        let topStock = portfolioData.reduce((top, current) => {
            const currentValue = current.price * current.quantity;
            return (currentValue > top.value) ? {symbol: current.symbol, value: currentValue} : top;
        }, {value: 0});
        document.getElementById('top-stock').textContent = topStock.symbol;
        // Prepare data for chart
        const labels = portfolioData.map(stock => stock.symbol);
        const data = portfolioData.map(stock => stock.price * stock.quantity);
        const ctx = document.getElementById('distribution-chart').getContext('2d');
        const chart = new Chart(ctx, {
            type: 'pie',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Portfolio Distribution',
                    data: data,
                    backgroundColor: ['#FF5733', '#33FF57', '#3357FF'],
                    borderColor: ['#C70039', '#28B463', '#2980B9'],
                    borderWidth: 1
                }]
            }
        });
    </script>
</body>
</html>
