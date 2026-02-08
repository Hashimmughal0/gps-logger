# GDP Analytics Dashboard

## What's This Project About?

So basically, we built a data analysis tool that takes World Bank GDP data and lets you filter it by region/year, compute stats like average or sum, and then visualize it with graphs. It's config-driven, meaning you don't hardcode anything — all the filtering and operations are controlled through a JSON config file.

## How It's Organized

The project is structured with clear separation of concerns (that's the Single Responsibility Principle they keep talking about):

```
SDA_Project/
├── main.py                    # Entry point, runs the whole pipeline
├── config/
│   └── config.json           # Where you define what region, year, operation to use
├── data/
│   └── GDP_Data.csv          # The actual World Bank data
└── utilities/
    ├── __pycache__/
    ├── config_loader.py      # Reads and validates config.json
    ├── data_loader.py        # Loads the CSV file
    ├── data_cleaner.py       # Handles the messy data transformation
    ├── data_processor.py      # Filters data and does the math
    ├── dashboard.py          # Prints results and calls the plotting
    └── visualizer.py         # Makes the graphs
```

Each module does one thing:
- **config_loader**: reads JSON, validates required fields
- **data_loader**: just opens the CSV, nothing fancy
- **data_cleaner**: transforms wide CSV format → long format (one row per year)
- **data_processor**: filters by region/year, calculates average/sum
- **dashboard**: displays results and kicks off visualization
- **visualizer**: creates bar charts, pie charts, histograms, etc.

## How to Use It

### 1. Install Dependencies

```bash
pip install pandas matplotlib seaborn numpy
```

### 2. Set Up Your Config

Edit `SDA_Project/config/config.json`:

```json
{
  "region": "South Asia",
  "year": 2020,
  "operation": "average",
  "output": "dashboard"
}
```

Fields:
- `region`: Filter by region (e.g., "South Asia", "Asia", "Africa"). Note: if the exact region isn't found, it falls back to year-only filtering.
- `year`: Which year you want to analyze (must be an integer)
- `operation`: Either `"average"` (mean GDP) or `"sum"` (total GDP)
- `output`: Currently just set this to `"dashboard"` (that's the main mode)
- `top_n` (optional): Limit the number of countries shown in the bar chart. Default is 12.

### 3. Run It

```bash
python SDA_Project/main.py
```

You'll see:
1. A text output with the config values and the computed result (formatted as Trillion/Billion/Million USD)
2. Four charts pop up:
   - **Bar chart**: Top N countries by GDP
   - **Pie chart**: GDP distribution (combines small slices into "Other")
   - **Histogram**: Distribution of GDP values
   - **Scatter plot**: GDP values by index

## Key Features

### Config-Driven Everything
No hardcoded region, year, or operation. It's all in the JSON file. If you want to run a different analysis, just edit the config and rerun — no code changes needed.

### Flexible Data Handling
The CSV is in "wide format" (days as columns), so we reshape it to "long format" (one row per year). Also handles missing `Region` by falling back to `Continent` field.

### Human-Readable Numbers
Instead of showing `1e12` or whatever, graphs display values in Billion, Million, Trillion USD. Makes the dashboard actually useful.

### Smart Filtering
- Region matching is case-insensitive and does substring matching (so "asia" will match "South Asia")
- If no countries match the region, it falls back to just filtering by year
- Small countries in pie charts are grouped into "Other" to reduce clutter

### Functional Programming
The code uses Python's functional constructs where it makes sense: `map`, `filter`, `lambda`, list comprehensions. We tried to follow the university's constraint about minimizing loops.

## What Each Graph Shows

1. **Bar Chart** → Top N countries ranked by GDP for the selected year
2. **Pie Chart** → Percentage breakdown of GDP (top 8 countries + "Other")
3. **Histogram** → How GDP values are distributed across countries
4. **Scatter Plot** → GDP values plotted in order

## Error Handling

The app will yell at you if:
- The CSV file doesn't exist
- The JSON config is missing required fields
- The `year` isn't an integer
- The `operation` is neither "average" nor "sum"

Error messages get printed to the terminal. Pretty straightforward.

## Technical Notes

### The Data
World Bank GDP data from 1960-2024 (ish). Each country has a region and continent tag, plus GDP values for each year.

### The Weird Stuff
- The CSV has year columns (1960, 1961, etc.), so `data_cleaner.py` unpacks those into individual rows. This is why it looks a bit ugly.
- If the exact region name isn't in the data, the code tries substring matching, then falls back to year-only. This is because region names in different sources vary (e.g., "Asia" vs. "East Asia & Pacific").

### Performance
For the full World Bank dataset (~200+ countries, 60+ years), the pipeline takes a second or two. Nothing crazy. If you're doing serious work, you'd want to cache the cleaned data or use SQL, but for a school project, this is fine.

## Stuff We Could Improve

- **Country-level filtering**: The config supports it in theory, but the code doesn't use it yet. We could add that.
- **Tests**: We don't have unit tests. In a real project, you'd want them.
- **Export**: Right now we just display graphs. Could add CSV export or save charts to PNG.
- **More chart types**: Could add line graphs, heatmaps, etc.
- **Better legend handling**: For super large datasets, the pie legend still gets cramped.

## Running a Quick Test

```bash
# Make sure dependencies are installed
pip install pandas matplotlib seaborn

# Run the default analysis (South Asia, 2020, average)
python SDA_Project/main.py

# Close the graphs as they pop up
```

You should see output like:
```
GDP ANALYTICS DASHBOARD
===================================
Region    : South Asia
Year      : 2020
Operation : AVERAGE
Result    : 1.52 Trillion USD
```

Then four graphs appear.

## Troubleshooting

**Error: "Config file not found"**
- Make sure `SDA_Project/config/config.json` exists
- Check the path is correct

**Error: "GDP CSV file not found"**
- Verify `SDA_Project/data/GDP_Data.csv` is there

**Graphs look weird / x-axis is unreadable**
- This is normal if you have tons of countries. The `top_n` config limits the bar chart. Set it lower if needed.
- Pie chart automatically combines small slices into "Other"

**No data shows up**
- The region might not exist in the dataset. Try a different region like "Asia", "Africa", or "Europe"
- Or check if that year has data for the region you selected

## Final Notes

This was built to learn functional programming principles and how to structure a real-ish data analysis pipeline. It works, it's not the most optimized thing ever, but it does what it's supposed to do without hardcoding stuff.

If you need to modify it, the modular structure makes it pretty easy. Just edit the relevant utility file and rerun.

Good luck! 🚀
