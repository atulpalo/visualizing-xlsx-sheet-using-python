import pandas as pd
import matplotlib.pyplot as plt

file_path = "ZEDW_BUDGET_DASH_V.xlsx"  # Use the provided file

# Read the Excel file
df = pd.read_excel(file_path)

# Ensure required columns are present
required_cols = {'GJAHR', 'SHOP', 'BUDGET'}
if required_cols.issubset(df.columns):
    # Clean and prepare data
    df = df.dropna(subset=['GJAHR', 'SHOP', 'BUDGET'])
    df['GJAHR'] = df['GJAHR'].astype(str)
    df['BUDGET'] = pd.to_numeric(df['BUDGET'], errors='coerce').fillna(0)

    # Group by year and shop, summing the budget
    grouped = df.groupby(['GJAHR', 'SHOP'])['BUDGET'].sum().reset_index()

    # Pivot for plotting: rows=year, columns=shop, values=budget
    pivot = grouped.pivot(index='GJAHR', columns='SHOP', values='BUDGET').fillna(0)

    # Limit to top 10 shops, aggregate others
    top_n = 10
    top_shops = pivot.sum().sort_values(ascending=False).head(top_n).index
    pivot['Other'] = pivot.drop(columns=top_shops).sum(axis=1)
    pivot = pivot[list(top_shops) + ['Other']]

    # Plot: grouped bar chart of actual budget values
    ax = pivot.plot(kind='bar', figsize=(16, 8))
    plt.xlabel('Year')
    plt.ylabel('Total Budget (INR)')  # <-- Added units here
    plt.title(f'Year-wise Total Budget by Shop (Top {top_n} + Other)\nUnit: Indian Rupees (INR)')  # <-- Added units here

    # Add scaling info text box inside plot
    textstr = 'Budget values are in Indian Rupees (INR)\nNo additional scaling applied (raw values)'
    props = dict(boxstyle='round', facecolor='wheat', alpha=0.5)
    ax.text(0.02, 0.95, textstr, transform=ax.transAxes, fontsize=10,
            verticalalignment='top', bbox=props)

    plt.legend(title='Shop', bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')
    plt.tight_layout()
    plt.grid(axis='y', linestyle='--', alpha=0.7)
    plt.show()
else:
    print(f"Required columns {required_cols} not found in the Excel file.")
