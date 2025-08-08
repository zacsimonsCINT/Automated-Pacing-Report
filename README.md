# Qualtrics Pacing and Media Analysis Tool

This tool provides an interactive interface within a Jupyter or Google Colab notebook to clean and analyze survey data from Qualtrics for ad effectiveness studies. Its primary purpose is to check the **pacing** of a study—that is, to count the number of completed surveys that have come from different advertising channels (e.g., Linear TV, Facebook, TikTok, etc.).

The script ingests a raw Qualtrics survey data file and a "media map" file. It automatically cleans the survey data, separates respondents who were exposed to Linear TV ads from those exposed to digital ads, and then uses the media map to categorize the digital respondents. The final output is a clear summary of respondent counts per media type, allowing researchers to quickly assess if their sample collection is on track. 📈

-----

## Key Features

  * **Interactive UI**: Uses `ipywidgets` to create a user-friendly interface, eliminating the need to edit code for each run.
  * **Automated Data Cleaning**: Filters out test data, survey previews, and incomplete responses from the raw Qualtrics export.
  * **Respondent Segmentation**: Intelligently separates "Linear TV" respondents from the main "Exposed" group based on a specified site ID (`SambaS2S`).
  * **Dynamic Media Mapping**: Joins the remaining survey data with a media mapping file to categorize respondents by their specific digital media source.
  * **Clear Pacing Summary**: Outputs a simple, clean table of respondent counts by media type, including a count for any respondents that could not be mapped.

-----

## Requirements

### Environment

  * A Jupyter Notebook or Google Colab environment.
  * The ability to upload your two CSV files to the notebook session's file system.

### Python Libraries

  * `pandas`
  * `ipywidgets`
  * `numpy`

-----

## Input File Requirements

You will need **two** CSV files. The script is flexible, allowing you to specify the exact column names your files use.

### 1\. Qualtrics Survey Data File

This is the raw data export from your Qualtrics survey.

  * It should be a **CSV** file.
  * The script is designed to handle the standard Qualtrics format where the **second row is a secondary header** (e.g., "Click here to edit question text"). The script automatically skips this row.
  * It should contain columns that identify:
      * **Placement ID**: A unique ID for the ad placement a user saw (e.g., `AT_PLACEMENT_ID`). This is the key used to join with the media map.
      * **Samba Site ID**: A column to identify Linear TV respondents, often with a value like `SambaS2S` (e.g., `AT_SITE_ID`).
      * **Survey Status**: Columns like `finished`, `status`, and `gc` that indicate if a survey was completed successfully or is just a preview/test.

### 2\. Media Mapping File

This is a simple lookup file that connects a placement ID to a media partner or type.

  * It should be a **CSV** file.
  * It must contain columns for:
      * **Placement ID**: The ID that matches the one in your Qualtrics file (e.g., `placement_id`). The script will remove any duplicate IDs from this file.
      * **Media Type**: The corresponding name of the media partner or channel (e.g., `media_type` with values like 'Facebook', 'YouTube', 'Programmatic').

-----

## How to Use

The script generates an easy-to-use control panel right in your notebook.

1.  **Upload Files**: First, upload your Qualtrics CSV and your Media Mapping CSV to your notebook environment's session storage.
2.  **Run the Script Cell**: Execute the Python code cell. This will display the UI shown below.
3.  **Fill in the Fields**:
      * Enter the exact filenames for your two uploaded files (e.g., `My_Qualtrics_Data.csv`).
      * Verify the column names. The script provides common defaults, but you should change them to match your files **exactly**.
4.  **Click to Analyze**: Press the green **"Clean and Analyze"** button.
5.  **Review the Output**: The results will be printed in the output area below the button. The process logs will show you how many rows were filtered at each step, and the final output will be a table with the pacing counts.

-----

## The Analysis Logic

The script follows a clear, multi-step process to get from raw files to the final pacing counts.

1.  **Load & Clean Qualtrics Data**: The script loads the Qualtrics file, converts all column headers to lowercase for consistency, and then performs several cleaning steps:
      * Removes test responses (where `gc` column is '2').
      * Removes preview responses (where `status` column is 'Survey Preview').
      * **Keeps only fully completed surveys** (where `finished` column is 'True'). This is the most important filter.
2.  **Initial Segmentation**: It counts the initial number of **Exposed** and **Control** group respondents from the `studytype` column.
3.  **Separate Linear TV**: From the pool of "Exposed" respondents, it identifies and separates anyone who came from Linear TV by checking the Samba Site ID column for the value `SambaS2S`. These are counted and set aside.
4.  **Map Digital Media**:
      * The remaining "Exposed" respondents are considered "Digital".
      * The script performs a **left merge**, using the Placement ID as the key. This means it keeps *all* the digital respondents from the Qualtrics file and attaches the corresponding `media_type` from the media map.
      * If a respondent's Placement ID isn't found in the media map, their `media_type` is labeled as **"Unmapped"**. This is useful for identifying missing IDs in your map.
5.  **Final Tally**: The script counts the number of respondents for each `media_type` from the newly merged data, adds the "Linear TV" count back into the results, and displays the final summary table.
