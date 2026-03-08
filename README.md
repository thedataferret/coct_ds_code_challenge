# City of Cape Town Service Request Density Classification

A data science project that classifies areas of the City of Cape Town by population density, using service request volumes as a proxy indicator. The assumption is that densely-populated areas generate higher volumes of service requests.

## Main Deliverable

`**classification.ipynb**` is the primary notebook and should be run first. It contains:

- Data loading and preprocessing
- Exploratory data analysis with visualisations
- Statistical analysis of service request distributions
- Density classification of H3 hex areas
- Interactive hex maps showing density classifications

If you run nothing else, run this notebook.

## Project Structure

```
├── classification.ipynb    # Main analysis and classification (run this)
├── data_quality.ipynb      # Data quality checks and validation
├── ai_work.md              # Reflections on using AI (Opus 4.5) in this project
├── requirements.txt        # Python dependencies
├── raw_data/
│   ├── sr_hex.csv              # Service request data (~236MB)
│   └── hex_polygons.geojson    # H3 hex polygon geometries
└── README.md
```

## Data Quality

Before the main analysis, I designed a huge workbook full of data quality checks that turned out to be unneccessary because the data was really clean. You can check it out in the notebook`data_quality.ipynb`.

It's a thrill ride:

- Schema validation and data type checks
- Null rate analysis per field
- Coordinate bounds validation
- Timestamp parsing and range checks
- Detection of malformed records

## AI-Assisted Development

I used Claude Opus 4.5 via the Cursor IDE. I detail some of the times it was completely wrong, or lied to me, in the file `**ai_work.md**`. 

Highlights include:

- Silent pandas coercion problems
- Timezone handling pitfalls
- Statistical calculation errors that required manual correction
- My detailed, front-loaded prompting approach

## Prerequisites

### Python Version

- Python 3.10+

### Dependencies

Install required packages:

```bash
pip install -r requirements.txt
```

Or install manually:

```bash
pip install pandas>=2.0.0 geopandas>=0.14.0 numpy>=1.24.0 matplotlib>=3.7.0 plotly>=5.15.0 folium>=0.14.0 branca>=0.6.0
```

### Data Files

The `raw_data/` folder must contain the following files. Download them from the public S3 bucket:

| Local File | Download From |
|------------|---------------|
| `sr_hex.csv` | [sr_hex.csv.gz](https://cct-ds-code-challenge-input-data.s3.af-south-1.amazonaws.com/sr_hex.csv.gz) (decompress after download) |
| `hex_polygons.geojson` | [city-hex-polygons-8.geojson](https://cct-ds-code-challenge-input-data.s3.af-south-1.amazonaws.com/city-hex-polygons-8.geojson) |

**Quick setup:**

```bash
mkdir -p raw_data && cd raw_data

# Download and decompress service request data
curl -O https://cct-ds-code-challenge-input-data.s3.af-south-1.amazonaws.com/sr_hex.csv.gz
gunzip sr_hex.csv.gz

# Download hex polygons
curl -o hex_polygons.geojson https://cct-ds-code-challenge-input-data.s3.af-south-1.amazonaws.com/city-hex-polygons-8.geojson
```

## Running the Notebook

1. Clone the repository
2. Install dependencies: `pip install -r requirements.txt`
3. Open `classification.ipynb` in Jupyter Notebook/Lab or VS Code
4. Run all cells sequentially

The notebook produces interactive Plotly charts and Folium maps that render in the notebook output.

## Outputs

- Statistical tables of service requests by code and suburb
- Distribution histograms with mean/median/standard deviation markers
- Interactive choropleth hex maps with hover tooltips
- Density classifications: `low_density`, `normal_density`, `high_density`, `very_high_density`

## License

See `LICENSE` file.