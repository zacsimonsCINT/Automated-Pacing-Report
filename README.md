Of course\! Here is a `README.md` file that explains the Python script.

-----

# Campaign Pacing Report Generator

This script provides an on-demand reporting tool to check the delivery status (pacing) of active advertising campaigns. It is designed to run within a Databricks notebook environment.

The user uploads a CSV file containing a list of campaign owner emails. The script then queries campaign and impression data for each email, calculates pacing metrics, and generates a consolidated report. The final report flags campaigns as **Ahead**, **Behind**, or **On Pace**, allowing users to quickly identify campaigns that may require attention.

-----

## Requirements

  * **Environment**: A Databricks notebook. The script relies on a pre-configured `spark` session to execute SQL queries.
  * **Permissions**: Read access to the `dsm.measurement.vw_im_campaigns` and `dsm.measurement.vw_impressions` tables/views in your Databricks environment.
  * **Python Libraries**: The following libraries must be available in your notebook's environment:
      * `pandas`
      * `numpy`
      * `ipywidgets`

-----

## Input File Format

The script requires a CSV file as input. This file **must** contain a header row with a column named exactly `emails to query`. The script will process each email listed in this column.

#### Example `input.csv`:

```csv
emails to query
user1@example.com
user2@example.com
another.user@example.com
```

-----

## How to Use

The script creates an interactive widget within the Databricks notebook for easy operation.

1.  **Run the Script**: Execute the notebook cell containing the script code. This will display a file uploader and a "Generate Report" button.
2.  **Prepare Your Input**: Create a `.csv` file with the email addresses you want to check, as described in the "Input File Format" section.
3.  **Upload the File**: Click the **"Upload CSV"** button and select the `.csv` file you created.
4.  **Generate the Report**: Click the green **"Generate Report"** button. The script will begin querying the data, and progress messages will appear in the output area.
5.  **Save the Results**: Once complete, a large text block containing the report in CSV format will be printed. Copy the entire text block (from the header row to the last line of data) and paste it into a text editor. Save this new file with a `.csv` extension (e.g., `pacing_report.csv`). You can then open this file in any spreadsheet software like Excel or Google Sheets.

-----

## Logic & Calculations

The script performs the following key steps to generate the report:

### Data Querying

For each email provided, the script executes a SQL query to fetch all *active* campaigns (where `end_date` is in the future). It joins campaign data with impression data to count the number of impressions delivered so far for each campaign.

### Metric Calculation

After retrieving the data, several key metrics are calculated using `pandas`:

  * **Percent Delivered**: The percentage of expected impressions that have been delivered.
    $$\text{Percent Delivered} = \frac{\text{Delivered Impressions}}{\text{Expected Impressions}} \times 100$$
  * **Percent Time Elapsed**: The percentage of the campaign's total duration that has passed as of today.
    $$\text{Percent Time Elapsed} = \frac{\text{Days from Start to Today}}{\text{Total Days in Campaign}} \times 100$$
  * **Pacing Difference**: The simple arithmetic difference between the delivery progress and the time progress. A positive value means delivery is ahead of schedule, while a negative value means it is behind.
    $$\text{Pacing Difference} = \text{Percent Delivered} - \text{Percent Time Elapsed}$$
  * **Pacing Status**: A categorical flag to quickly identify the campaign's status based on the `pacing_difference`:
      * **Ahead**: The `pacing_difference` is greater than `+15%`.
      * **Behind**: The `pacing_difference` is less than `-15%`.
      * **On Pace**: The `pacing_difference` is between `-15%` and `+15%`.

-----

## Output Column Definitions

The final generated `.csv` report will contain the following columns:

| Column Name | Description |
| :--- | :--- |
| **campaign\_id** | The unique identifier for the campaign. |
| **campaign\_name** | The name of the campaign. |
| **start\_date** | The date the campaign began or is set to begin. |
| **end\_date** | The date the campaign ended or is set to end. |
| **expected\_impressions\_count** | The total number of impressions booked for the campaign. |
| **delivered\_impressions** | The actual number of impressions served to date. |
| **percent\_delivered** | The percentage of expected impressions that have been delivered. |
| **impressions\_remaining** | The number of impressions yet to be delivered (`expected - delivered`). |
| **percent\_time\_elapsed** | The percentage of the campaign's total time duration that has passed. |
| **pacing\_difference** | The numerical difference between `percent_delivered` and `percent_time_elapsed`. |
| **pacing\_status** | A flag indicating if the campaign is 'Ahead', 'Behind', or 'On Pace'. |
| **owner\_email** | The email address of the campaign owner, used to query the data. |
