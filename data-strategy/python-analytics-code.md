# Python Code Examples for StudyHub Café Analytics
## Data Analysis and Visualization Scripts

---

## 1. Market Analysis and Visualization

### 1.1 Competitive Pricing Analysis

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from datetime import datetime, timedelta
import plotly.express as px
import plotly.graph_objects as go

# Competitive pricing data
pricing_data = {
    'Coffee Shop': ['StudyHub Café', 'Coffee Bean', 'Book Café', 'Campus Coffee', 'Starbucks'],
    'Espresso': [2.00, 2.50, 2.00, 1.50, 3.00],
    'Americano': [2.50, 3.00, 2.50, 2.00, 4.00],
    'Latte': [3.00, 3.50, 3.00, 2.50, 5.00],
    'Cappuccino': [3.00, 3.50, 3.00, 2.50, 5.00],
    'Study_Time_Limit': [0, 0, 180, 0, 120],  # minutes (0 = no limit)
    'WiFi_Quality': [5, 3, 2, 3, 4],  # 1-5 scale
    'Study_Amenities': [5, 2, 3, 2, 1]  # 1-5 scale
}

df_pricing = pd.DataFrame(pricing_data)

# Visualization 1: Coffee Price Comparison
def plot_coffee_price_comparison():
    """Create a comparison chart of coffee prices across competitors"""
    fig, ax = plt.subplots(figsize=(12, 8))
    
    coffee_types = ['Espresso', 'Americano', 'Latte', 'Cappuccino']
    x = np.arange(len(coffee_types))
    width = 0.15
    
    for i, shop in enumerate(df_pricing['Coffee Shop']):
        prices = [df_pricing.loc[i, coffee] for coffee in coffee_types]
        ax.bar(x + i*width, prices, width, label=shop)
    
    ax.set_xlabel('Coffee Types')
    ax.set_ylabel('Price (USD)')
    ax.set_title('Coffee Price Comparison - Tashkent Market')
    ax.set_xticks(x + width * 2)
    ax.set_xticklabels(coffee_types)
    ax.legend()
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.show()
    return fig

# Visualization 2: Competitive Positioning Matrix
def plot_competitive_positioning():
    """Create a scatter plot showing competitive positioning"""
    fig, ax = plt.subplots(figsize=(10, 8))
    
    # Calculate average coffee price for each shop
    df_pricing['Avg_Coffee_Price'] = df_pricing[['Espresso', 'Americano', 'Latte', 'Cappuccino']].mean(axis=1)
    
    scatter = ax.scatter(df_pricing['Avg_Coffee_Price'], 
                        df_pricing['Study_Amenities'], 
                        s=df_pricing['WiFi_Quality']*100,
                        c=df_pricing['Study_Time_Limit'], 
                        cmap='RdYlGn_r', 
                        alpha=0.7,
                        edgecolors='black')
    
    # Add labels for each point
    for i, shop in enumerate(df_pricing['Coffee Shop']):
        ax.annotate(shop, 
                   (df_pricing.loc[i, 'Avg_Coffee_Price'], df_pricing.loc[i, 'Study_Amenities']),
                   xytext=(5, 5), textcoords='offset points')
    
    ax.set_xlabel('Average Coffee Price (USD)')
    ax.set_ylabel('Study Amenities Quality (1-5 scale)')
    ax.set_title('Competitive Positioning Matrix\n(Bubble size = WiFi Quality, Color = Time Limit)')
    
    # Add colorbar
    cbar = plt.colorbar(scatter)
    cbar.set_label('Study Time Limit (minutes)')
    
    plt.tight_layout()
    plt.show()
    return fig

# Run the visualizations
if __name__ == "__main__":
    price_fig = plot_coffee_price_comparison()
    position_fig = plot_competitive_positioning()
```

### 1.2 Customer Segmentation Analysis

```python
# Customer demographic and behavior analysis
import pandas as pd
import numpy as np
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt
import seaborn as sns

# Simulated customer data
np.random.seed(42)

def generate_customer_data(n_customers=1000):
    """Generate simulated customer data for analysis"""
    
    data = {
        'customer_id': range(1, n_customers + 1),
        'age': np.random.normal(22, 4, n_customers).clip(16, 35),
        'monthly_visits': np.random.poisson(8, n_customers),
        'avg_session_duration': np.random.gamma(2, 2, n_customers),  # hours
        'monthly_spending': np.random.gamma(2, 20, n_customers),     # USD
        'study_group_preference': np.random.choice([0, 1], n_customers, p=[0.7, 0.3]),
        'preferred_time': np.random.choice(['morning', 'afternoon', 'evening', 'night'], 
                                         n_customers, p=[0.2, 0.3, 0.4, 0.1]),
        'student_type': np.random.choice(['university', 'high_school', 'professional'], 
                                       n_customers, p=[0.7, 0.2, 0.1])
    }
    
    df = pd.DataFrame(data)
    
    # Add some realistic correlations
    df.loc[df['student_type'] == 'high_school', 'age'] = np.random.normal(17, 1, 
                                                          sum(df['student_type'] == 'high_school')).clip(15, 19)
    df.loc[df['student_type'] == 'professional', 'age'] = np.random.normal(28, 3, 
                                                           sum(df['student_type'] == 'professional')).clip(23, 35)
    df.loc[df['student_type'] == 'professional', 'monthly_spending'] *= 1.5
    
    return df

def perform_customer_segmentation(df):
    """Perform RFM analysis and customer segmentation"""
    
    # RFM Analysis (Recency, Frequency, Monetary)
    rfm_data = df[['monthly_visits', 'monthly_spending', 'avg_session_duration']].copy()
    
    # Standardize the data
    scaler = StandardScaler()
    rfm_scaled = scaler.fit_transform(rfm_data)
    
    # Perform K-means clustering
    kmeans = KMeans(n_clusters=4, random_state=42)
    df['segment'] = kmeans.fit_predict(rfm_scaled)
    
    # Define segment names based on characteristics
    segment_names = {
        0: 'Casual Visitors',
        1: 'Regular Studiers', 
        2: 'Heavy Users',
        3: 'Premium Customers'
    }
    
    df['segment_name'] = df['segment'].map(segment_names)
    
    return df, kmeans

def visualize_customer_segments(df):
    """Create visualizations for customer segments"""
    
    fig, axes = plt.subplots(2, 2, figsize=(15, 12))
    
    # 1. Segment distribution
    df['segment_name'].value_counts().plot(kind='pie', ax=axes[0,0], autopct='%1.1f%%')
    axes[0,0].set_title('Customer Segment Distribution')
    axes[0,0].set_ylabel('')
    
    # 2. Age distribution by segment
    sns.boxplot(data=df, x='segment_name', y='age', ax=axes[0,1])
    axes[0,1].set_title('Age Distribution by Segment')
    axes[0,1].tick_params(axis='x', rotation=45)
    
    # 3. Monthly spending by segment
    sns.boxplot(data=df, x='segment_name', y='monthly_spending', ax=axes[1,0])
    axes[1,0].set_title('Monthly Spending by Segment')
    axes[1,0].tick_params(axis='x', rotation=45)
    
    # 4. Visit frequency vs session duration
    scatter = axes[1,1].scatter(df['monthly_visits'], df['avg_session_duration'], 
                               c=df['segment'], cmap='viridis', alpha=0.6)
    axes[1,1].set_xlabel('Monthly Visits')
    axes[1,1].set_ylabel('Average Session Duration (hours)')
    axes[1,1].set_title('Visit Frequency vs Session Duration')
    
    plt.tight_layout()
    plt.show()
    
    return fig

# Generate and analyze customer data
customer_df = generate_customer_data(1000)
segmented_df, kmeans_model = perform_customer_segmentation(customer_df)
segment_viz = visualize_customer_segments(segmented_df)

# Segment analysis summary
def analyze_segments(df):
    """Provide detailed analysis of each customer segment"""
    
    segment_analysis = df.groupby('segment_name').agg({
        'age': ['mean', 'std'],
        'monthly_visits': ['mean', 'std'],
        'avg_session_duration': ['mean', 'std'],
        'monthly_spending': ['mean', 'std'],
        'customer_id': 'count'
    }).round(2)
    
    print("Customer Segment Analysis:")
    print("=" * 50)
    print(segment_analysis)
    
    return segment_analysis

segment_summary = analyze_segments(segmented_df)
```

---

## 2. Demand Forecasting and Operational Analytics

### 2.1 Demand Prediction Model

```python
# Demand forecasting using time series analysis
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error, mean_squared_error
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

def generate_demand_data():
    """Generate simulated daily demand data with seasonal patterns"""
    
    # Create date range for one year
    dates = pd.date_range(start='2024-01-01', end='2024-12-31', freq='D')
    
    # Base demand with weekly and seasonal patterns
    base_demand = 100  # baseline customers per day
    
    demand_data = []
    for date in dates:
        # Weekly pattern (higher on weekends before exams)
        day_of_week = date.weekday()
        weekly_factor = 1.2 if day_of_week >= 5 else 1.0  # Weekend boost
        
        # Seasonal pattern (higher during exam periods)
        month = date.month
        if month in [1, 5, 6, 11, 12]:  # Exam months
            seasonal_factor = 1.4
        elif month in [7, 8]:  # Summer break
            seasonal_factor = 0.6
        else:
            seasonal_factor = 1.0
        
        # Weather impact (simulated)
        weather_factor = np.random.normal(1.0, 0.1)
        
        # Holiday impact
        holiday_factor = 0.5 if date.month == 1 and date.day <= 7 else 1.0  # New Year
        
        # Calculate demand
        daily_demand = base_demand * weekly_factor * seasonal_factor * weather_factor * holiday_factor
        daily_demand = max(0, int(daily_demand + np.random.normal(0, 10)))
        
        demand_data.append({
            'date': date,
            'demand': daily_demand,
            'day_of_week': day_of_week,
            'month': month,
            'is_weekend': day_of_week >= 5,
            'is_exam_period': month in [1, 5, 6, 11, 12],
            'temperature': np.random.normal(15, 10),  # Celsius
            'is_holiday': holiday_factor < 1.0
        })
    
    return pd.DataFrame(demand_data)

def create_demand_features(df):
    """Create features for demand prediction model"""
    
    features_df = df.copy()
    
    # Time-based features
    features_df['day_of_year'] = features_df['date'].dt.dayofyear
    features_df['week_of_year'] = features_df['date'].dt.isocalendar().week
    features_df['quarter'] = features_df['date'].dt.quarter
    
    # Lag features (previous days' demand)
    features_df['demand_lag_1'] = features_df['demand'].shift(1)
    features_df['demand_lag_7'] = features_df['demand'].shift(7)
    features_df['demand_lag_30'] = features_df['demand'].shift(30)
    
    # Rolling averages
    features_df['demand_ma_7'] = features_df['demand'].rolling(window=7).mean()
    features_df['demand_ma_30'] = features_df['demand'].rolling(window=30).mean()
    
    # Cyclical encoding for day of week
    features_df['day_sin'] = np.sin(2 * np.pi * features_df['day_of_week'] / 7)
    features_df['day_cos'] = np.cos(2 * np.pi * features_df['day_of_week'] / 7)
    
    # Cyclical encoding for month
    features_df['month_sin'] = np.sin(2 * np.pi * features_df['month'] / 12)
    features_df['month_cos'] = np.cos(2 * np.pi * features_df['month'] / 12)
    
    # Drop rows with NaN values (due to lag features)
    features_df = features_df.dropna()
    
    return features_df

def train_demand_model(df):
    """Train a Random Forest model for demand forecasting"""
    
    feature_columns = [
        'day_of_week', 'month', 'is_weekend', 'is_exam_period', 
        'temperature', 'is_holiday', 'day_of_year', 'week_of_year', 'quarter',
        'demand_lag_1', 'demand_lag_7', 'demand_lag_30',
        'demand_ma_7', 'demand_ma_30', 'day_sin', 'day_cos', 'month_sin', 'month_cos'
    ]
    
    X = df[feature_columns]
    y = df['demand']
    
    # Split data
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # Train model
    model = RandomForestRegressor(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)
    
    # Predictions
    y_pred = model.predict(X_test)
    
    # Evaluate model
    mae = mean_absolute_error(y_test, y_pred)
    rmse = np.sqrt(mean_squared_error(y_test, y_pred))
    
    print(f"Demand Forecasting Model Performance:")
    print(f"Mean Absolute Error: {mae:.2f}")
    print(f"Root Mean Square Error: {rmse:.2f}")
    
    # Feature importance
    feature_importance = pd.DataFrame({
        'feature': feature_columns,
        'importance': model.feature_importances_
    }).sort_values('importance', ascending=False)
    
    return model, feature_importance, X_test, y_test, y_pred

def visualize_demand_forecast(df, model, feature_importance):
    """Create visualizations for demand forecasting"""
    
    fig, axes = plt.subplots(2, 2, figsize=(15, 12))
    
    # 1. Demand over time
    axes[0,0].plot(df['date'], df['demand'])
    axes[0,0].set_title('Daily Customer Demand Over Time')
    axes[0,0].set_xlabel('Date')
    axes[0,0].set_ylabel('Number of Customers')
    axes[0,0].tick_params(axis='x', rotation=45)
    
    # 2. Seasonal pattern
    monthly_demand = df.groupby('month')['demand'].mean()
    axes[0,1].bar(monthly_demand.index, monthly_demand.values)
    axes[0,1].set_title('Average Monthly Demand')
    axes[0,1].set_xlabel('Month')
    axes[0,1].set_ylabel('Average Daily Customers')
    
    # 3. Weekly pattern
    weekly_demand = df.groupby('day_of_week')['demand'].mean()
    day_names = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    axes[1,0].bar(range(7), weekly_demand.values)
    axes[1,0].set_title('Average Daily Demand by Day of Week')
    axes[1,0].set_xlabel('Day of Week')
    axes[1,0].set_ylabel('Average Customers')
    axes[1,0].set_xticks(range(7))
    axes[1,0].set_xticklabels(day_names)
    
    # 4. Feature importance
    top_features = feature_importance.head(10)
    axes[1,1].barh(top_features['feature'], top_features['importance'])
    axes[1,1].set_title('Top 10 Most Important Features')
    axes[1,1].set_xlabel('Feature Importance')
    
    plt.tight_layout()
    plt.show()
    
    return fig

# Generate demand data and train model
demand_df = generate_demand_data()
features_df = create_demand_features(demand_df)
demand_model, importance, X_test, y_test, y_pred = train_demand_model(features_df)
forecast_viz = visualize_demand_forecast(demand_df, demand_model, importance)
```

### 2.2 Inventory Optimization

```python
# Inventory management optimization
import pandas as pd
import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

def generate_inventory_data():
    """Generate inventory and sales data for coffee shop items"""
    
    items = {
        'Coffee Beans (Arabica)': {'cost': 12, 'shelf_life': 30, 'daily_usage': 5},
        'Coffee Beans (Robusta)': {'cost': 8, 'shelf_life': 30, 'daily_usage': 3},
        'Milk': {'cost': 3, 'shelf_life': 7, 'daily_usage': 20},
        'Sugar': {'cost': 2, 'shelf_life': 365, 'daily_usage': 2},
        'Cups (Small)': {'cost': 0.15, 'shelf_life': 1000, 'daily_usage': 50},
        'Cups (Large)': {'cost': 0.25, 'shelf_life': 1000, 'daily_usage': 80},
        'Pastries': {'cost': 1.5, 'shelf_life': 3, 'daily_usage': 15},
        'Sandwiches': {'cost': 2.5, 'shelf_life': 2, 'daily_usage': 10}
    }
    
    inventory_df = pd.DataFrame.from_dict(items, orient='index')
    inventory_df['item'] = inventory_df.index
    inventory_df['current_stock'] = [25, 15, 40, 10, 200, 300, 20, 15]
    inventory_df['reorder_point'] = inventory_df['daily_usage'] * 3  # 3-day safety stock
    
    return inventory_df.reset_index(drop=True)

def calculate_eoq(annual_demand, ordering_cost, holding_cost_rate, unit_cost):
    """Calculate Economic Order Quantity"""
    holding_cost = holding_cost_rate * unit_cost
    eoq = np.sqrt((2 * annual_demand * ordering_cost) / holding_cost)
    return eoq

def optimize_inventory(inventory_df, ordering_cost=50, holding_cost_rate=0.2):
    """Optimize inventory levels for all items"""
    
    results = []
    
    for _, item in inventory_df.iterrows():
        annual_demand = item['daily_usage'] * 365
        
        # Calculate EOQ
        eoq = calculate_eoq(annual_demand, ordering_cost, holding_cost_rate, item['cost'])
        
        # Calculate safety stock (based on demand variability)
        safety_stock = item['daily_usage'] * np.sqrt(7)  # Assume 7-day lead time
        
        # Reorder point
        reorder_point = safety_stock + (item['daily_usage'] * 7)  # Lead time demand
        
        # Total cost calculation
        ordering_cost_annual = (annual_demand / eoq) * ordering_cost
        holding_cost_annual = (eoq / 2 + safety_stock) * holding_cost_rate * item['cost']
        total_cost = ordering_cost_annual + holding_cost_annual
        
        results.append({
            'item': item['item'],
            'current_stock': item['current_stock'],
            'daily_usage': item['daily_usage'],
            'eoq': round(eoq, 2),
            'safety_stock': round(safety_stock, 2),
            'reorder_point': round(reorder_point, 2),
            'total_annual_cost': round(total_cost, 2),
            'recommended_action': 'ORDER' if item['current_stock'] <= reorder_point else 'MAINTAIN'
        })
    
    return pd.DataFrame(results)

def visualize_inventory_optimization(inventory_df, optimized_df):
    """Create inventory optimization visualizations"""
    
    fig, axes = plt.subplots(2, 2, figsize=(15, 12))
    
    # 1. Current stock vs reorder point
    axes[0,0].bar(range(len(optimized_df)), optimized_df['current_stock'], 
                  alpha=0.7, label='Current Stock')
    axes[0,0].bar(range(len(optimized_df)), optimized_df['reorder_point'], 
                  alpha=0.7, label='Reorder Point')
    axes[0,0].set_title('Current Stock vs Reorder Point')
    axes[0,0].set_xlabel('Items')
    axes[0,0].set_ylabel('Quantity')
    axes[0,0].set_xticks(range(len(optimized_df)))
    axes[0,0].set_xticklabels(optimized_df['item'], rotation=45, ha='right')
    axes[0,0].legend()
    
    # 2. EOQ vs Daily Usage
    axes[0,1].scatter(optimized_df['daily_usage'], optimized_df['eoq'], s=100)
    axes[0,1].set_title('Economic Order Quantity vs Daily Usage')
    axes[0,1].set_xlabel('Daily Usage')
    axes[0,1].set_ylabel('EOQ')
    
    # Add item labels
    for i, item in enumerate(optimized_df['item']):
        axes[0,1].annotate(item[:10], 
                          (optimized_df.loc[i, 'daily_usage'], optimized_df.loc[i, 'eoq']),
                          xytext=(5, 5), textcoords='offset points', fontsize=8)
    
    # 3. Total annual cost by item
    axes[1,0].bar(range(len(optimized_df)), optimized_df['total_annual_cost'])
    axes[1,0].set_title('Total Annual Inventory Cost by Item')
    axes[1,0].set_xlabel('Items')
    axes[1,0].set_ylabel('Annual Cost (USD)')
    axes[1,0].set_xticks(range(len(optimized_df)))
    axes[1,0].set_xticklabels(optimized_df['item'], rotation=45, ha='right')
    
    # 4. Action recommendations
    action_counts = optimized_df['recommended_action'].value_counts()
    axes[1,1].pie(action_counts.values, labels=action_counts.index, autopct='%1.1f%%')
    axes[1,1].set_title('Inventory Action Recommendations')
    
    plt.tight_layout()
    plt.show()
    
    return fig

# Generate and optimize inventory
inventory_data = generate_inventory_data()
optimized_inventory = optimize_inventory(inventory_data)
inventory_viz = visualize_inventory_optimization(inventory_data, optimized_inventory)

print("Inventory Optimization Results:")
print("=" * 50)
print(optimized_inventory)
```

---

## 3. Customer Analytics and Personalization

### 3.1 Customer Behavior Analysis

```python
# Customer behavior tracking and analysis
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime, timedelta

def generate_customer_behavior_data():
    """Generate detailed customer behavior data"""
    
    np.random.seed(42)
    n_customers = 500
    n_days = 90  # 3 months of data
    
    behavior_data = []
    
    for customer_id in range(1, n_customers + 1):
        # Customer characteristics
        customer_type = np.random.choice(['casual', 'regular', 'premium'], p=[0.4, 0.5, 0.1])
        
        if customer_type == 'casual':
            visit_frequency = np.random.poisson(2)  # visits per month
            avg_spending = np.random.normal(8, 2)
            session_duration = np.random.gamma(1, 1.5)
        elif customer_type == 'regular':
            visit_frequency = np.random.poisson(8)  # visits per month
            avg_spending = np.random.normal(15, 3)
            session_duration = np.random.gamma(2, 2)
        else:  # premium
            visit_frequency = np.random.poisson(20)  # visits per month
            avg_spending = np.random.normal(25, 5)
            session_duration = np.random.gamma(3, 2)
        
        # Generate visits over the period
        total_visits = max(1, int(visit_frequency * 3))  # 3 months
        
        for visit in range(total_visits):
            visit_date = datetime(2024, 1, 1) + timedelta(
                days=np.random.randint(0, n_days)
            )
            
            # Visit characteristics
            hour = np.random.choice(range(9, 21), p=[0.05, 0.08, 0.12, 0.15, 0.15, 0.15, 0.12, 0.08, 0.05, 0.03, 0.01, 0.01])
            duration = max(0.5, session_duration + np.random.normal(0, 0.5))
            spending = max(2, avg_spending + np.random.normal(0, 2))
            
            # Product preferences
            if np.random.random() < 0.7:  # 70% order coffee
                main_product = np.random.choice(['espresso', 'americano', 'latte', 'cappuccino'])
            else:
                main_product = np.random.choice(['tea', 'smoothie', 'hot_chocolate'])
            
            # Study behavior
            brought_laptop = np.random.random() < 0.6
            group_study = np.random.random() < 0.3
            used_whiteboard = np.random.random() < 0.1
            
            behavior_data.append({
                'customer_id': customer_id,
                'visit_date': visit_date,
                'visit_hour': hour,
                'duration_hours': round(duration, 2),
                'spending_usd': round(spending, 2),
                'main_product': main_product,
                'customer_type': customer_type,
                'brought_laptop': brought_laptop,
                'group_study': group_study,
                'used_whiteboard': used_whiteboard,
                'day_of_week': visit_date.weekday(),
                'is_weekend': visit_date.weekday() >= 5
            })
    
    return pd.DataFrame(behavior_data)

def analyze_customer_behavior(df):
    """Perform comprehensive customer behavior analysis"""
    
    analyses = {}
    
    # 1. Visit patterns by time
    hourly_visits = df.groupby('visit_hour').size()
    daily_visits = df.groupby('day_of_week').size()
    
    # 2. Customer lifetime value analysis
    customer_clv = df.groupby('customer_id').agg({
        'spending_usd': ['sum', 'mean', 'count'],
        'duration_hours': 'mean',
        'visit_date': ['min', 'max']
    }).round(2)
    
    customer_clv.columns = ['total_spending', 'avg_spending', 'total_visits', 
                           'avg_duration', 'first_visit', 'last_visit']
    customer_clv['days_active'] = (customer_clv['last_visit'] - customer_clv['first_visit']).dt.days + 1
    customer_clv['clv'] = customer_clv['total_spending'] * (90 / customer_clv['days_active'])  # Projected 3-month CLV
    
    # 3. Product preferences by segment
    product_preferences = pd.crosstab(df['customer_type'], df['main_product'], normalize='index') * 100
    
    # 4. Study behavior analysis
    study_behavior = df.groupby('customer_type').agg({
        'brought_laptop': 'mean',
        'group_study': 'mean',
        'used_whiteboard': 'mean',
        'duration_hours': 'mean'
    }).round(3) * 100  # Convert to percentages
    
    analyses['hourly_visits'] = hourly_visits
    analyses['daily_visits'] = daily_visits
    analyses['customer_clv'] = customer_clv
    analyses['product_preferences'] = product_preferences
    analyses['study_behavior'] = study_behavior
    
    return analyses

def create_behavior_visualizations(df, analyses):
    """Create comprehensive behavior analysis visualizations"""
    
    fig, axes = plt.subplots(3, 2, figsize=(15, 18))
    
    # 1. Hourly visit patterns
    analyses['hourly_visits'].plot(kind='bar', ax=axes[0,0])
    axes[0,0].set_title('Visit Patterns by Hour of Day')
    axes[0,0].set_xlabel('Hour')
    axes[0,0].set_ylabel('Number of Visits')
    axes[0,0].tick_params(axis='x', rotation=0)
    
    # 2. Daily visit patterns
    day_names = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    analyses['daily_visits'].plot(kind='bar', ax=axes[0,1])
    axes[0,1].set_title('Visit Patterns by Day of Week')
    axes[0,1].set_xlabel('Day of Week')
    axes[0,1].set_ylabel('Number of Visits')
    axes[0,1].set_xticklabels(day_names, rotation=45)
    
    # 3. Customer lifetime value distribution
    axes[1,0].hist(analyses['customer_clv']['clv'], bins=30, alpha=0.7)
    axes[1,0].set_title('Customer Lifetime Value Distribution')
    axes[1,0].set_xlabel('CLV (USD)')
    axes[1,0].set_ylabel('Number of Customers')
    
    # 4. Product preferences heatmap
    sns.heatmap(analyses['product_preferences'], annot=True, fmt='.1f', 
                cmap='YlOrRd', ax=axes[1,1])
    axes[1,1].set_title('Product Preferences by Customer Type (%)')
    
    # 5. Study behavior by customer type
    analyses['study_behavior'].plot(kind='bar', ax=axes[2,0])
    axes[2,0].set_title('Study Behavior by Customer Type (%)')
    axes[2,0].set_ylabel('Percentage')
    axes[2,0].legend(bbox_to_anchor=(1.05, 1), loc='upper left')
    axes[2,0].tick_params(axis='x', rotation=45)
    
    # 6. Spending vs Duration correlation
    axes[2,1].scatter(df['duration_hours'], df['spending_usd'], alpha=0.6)
    axes[2,1].set_title('Study Duration vs Spending Correlation')
    axes[2,1].set_xlabel('Session Duration (hours)')
    axes[2,1].set_ylabel('Spending (USD)')
    
    # Add correlation coefficient
    correlation = df['duration_hours'].corr(df['spending_usd'])
    axes[2,1].text(0.05, 0.95, f'Correlation: {correlation:.3f}', 
                   transform=axes[2,1].transAxes, verticalalignment='top')
    
    plt.tight_layout()
    plt.show()
    
    return fig

# Generate behavior data and analyze
behavior_df = generate_customer_behavior_data()
behavior_analyses = analyze_customer_behavior(behavior_df)
behavior_viz = create_behavior_visualizations(behavior_df, behavior_analyses)

# Print key insights
print("Customer Behavior Analysis Summary:")
print("=" * 50)
print(f"Total Customers: {behavior_df['customer_id'].nunique()}")
print(f"Total Visits: {len(behavior_df)}")
print(f"Average CLV: ${behavior_analyses['customer_clv']['clv'].mean():.2f}")
print(f"Peak Hour: {behavior_analyses['hourly_visits'].idxmax()}:00")
print(f"Busiest Day: {['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'][behavior_analyses['daily_visits'].idxmax()]}")
```

This comprehensive Python code suite provides StudyHub Café with powerful analytics capabilities covering market analysis, demand forecasting, inventory optimization, and customer behavior analysis. Each script includes realistic data generation, sophisticated analysis algorithms, and professional visualizations that can be directly used in your business presentation and operations.

The code demonstrates practical applications of pandas, matplotlib, seaborn, and scikit-learn for business intelligence and decision-making, perfectly aligning with your BDA strategy for the study café business.
