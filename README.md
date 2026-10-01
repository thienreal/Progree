# Data Analytics Project

## Project Structure

- `requirements.txt`: List of required Python libraries.
- `notebooks/`: Contains Jupyter Notebook files for executing the code step-by-step.
- `data/`: Contains input datasets and intermediate/final outputs.
  - `raw/`: Stores the original dataset (Online Retail Dataset).
  - `processed/`: Stores cleaned data and segmented customer data after processing.
- `dashboards/`: Contains files related to data visualization using BI tools (Power BI/Tableau) and screenshots.

## Execution Guide

1. **Environment Setup:**
   Run the following command to install the required libraries:
   ```bash
   pip install -r requirements.txt
   ```

2. **Data Preparation:**
   - Download the raw dataset (e.g., Online Retail Dataset) in `.csv` format and place it in the `data/raw/` directory.

3. **Notebook Execution Order:**
   Run the notebooks in the `notebooks/` directory sequentially:
   - **Step 1:** Run `01_eda_and_cleaning.ipynb` to perform data cleaning. The output will be saved in `data/processed/`.
   - **Step 2:** Run `02_customer_segmentation.ipynb` for RFM calculation and customer clustering. The output will be saved in `data/processed/`.
   - **Step 3:** Run `03_hypothesis_and_forecasting.ipynb` to perform statistical hypothesis testing and time-series forecasting.

4. **Visualization (BI Dashboard):**
   - Open the `.pbix` (Power BI) file located in the `dashboards/` directory and refresh the data source to point to the files in `data/processed/`.
