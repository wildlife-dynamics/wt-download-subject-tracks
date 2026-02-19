# EarthRanger Subject Tracking Workflow

## Introduction

This workflow allows you to download and analyze subject tracking data from EarthRanger.

**What this workflow does:**
- Downloads tracking data for subjects from EarthRanger
- Processes observation relocations to trajectory segments
- Filters and processes observations based on your criteria
- Exports data in multiple formats (CSV, GeoParquet, GPKG)
- Optionally creates visual maps showing subject movement trajectories

**Who should use this:**
- Conservation managers monitoring animal movements
- Field coordinators tracking wildlife or assets
- Data analysts analyzing movement patterns and behavior
- Anyone needing to export EarthRanger subject tracking data in a structured format

## Prerequisites

Before using this workflow, you need:

1. **Ecoscope Desktop** installed on your computer
   - If you haven't installed it yet, please follow the installation instructions for Ecoscope Desktop

2. **EarthRanger Data Source** configured in Ecoscope Desktop
   - You must have already set up a connection to your EarthRanger server
   - Your data source should be configured with proper authentication credentials
   - You'll need to know the name of your configured data source (e.g., "mep_dev")

3. **Subject Group** set up in EarthRanger
   - You need to have at least one subject group configured in your EarthRanger system
   - You'll need to know the exact name of the subject group you want to analyze
   - Note: Using a group with mixed subtypes (e.g., animal species with ranger teams) could lead to unexpected results

## Installation

1. Open Ecoscope Desktop
2. Select "Workflow Templates"
3. Click "+ Add Template"
4. Copy and paste this URL https://github.com/wildlife-dynamics/wt-download-subjects and wait for the workflow template to be downloaded and initialized
5. The template will now appear in your available template list

## Configuration Guide

Once you've added the workflow template, you'll need to configure it for your specific needs. The configuration form is organized into several sections.

### Basic Configuration

These are the essential settings you'll need to configure for every workflow run:

#### 1. Workflow Details
Give your workflow a name and description to help you identify it later.

- **Workflow Name** (required): A descriptive name for this workflow run and the dashboard title
  - Example: `"Elephant Tracking - January 2015"`
- **Workflow Description** (optional): Additional details about this analysis
  - Example: `"Weekly movement analysis for Ecoscope subject group"`

#### 2. Time Range
Specify the time period for the tracking data you want to download.

- **Timezone** (required): Use the dropdown to select your timezone
- **Since** (required): Use the calendar picker to select the start date and time
  - Example: `01/01/2015, 12:00 AM`
- **Until** (required): Use the calendar picker to select the end date and time
  - Example: `01/07/2015, 11:59 PM`

#### 3. Data Source
Select your EarthRanger connection.

- **Data Source** (required): Choose from your configured data sources
  - Example: Select `mep_dev` from the dropdown

#### 4. Subject Group
Choose which subject group to analyze.

- **Subject Group Name** (required): Enter the exact name of the subject group as it appears in EarthRanger. You can find them at https:///<your-site>.pamdas.org/admin/observations/subjectgroup/
  - Example: `"Subjects"`
  - Note: Subject group contains mixed subtypes can lead to unexpected behaviors (e.g., different species or asset types)

#### 5. Group Data (Optional)
Organize your data into separate views based on time periods or categories.

- **Group by**: Create separate outputs grouped by:
  - Time: Year, Month, Day of week, Hour, etc.
  - Category: Select a categorical column from your tracking data. If you're unsure which columns are available, run the workflow once without grouping to see the data, then configure grouping in a subsequent run.

#### 6. Persist Subject Trajectories
Choose how to save your data.

- **Filetypes**: Select one or more output formats
  - **CSV**: Standard spreadsheet format, opens in Excel
  - **GeoParquet**: Efficient format for geospatial data
  - **GPKG**: GeoPackage format, opens in GIS software like QGIS
  - Example: Select both `CSV` and `GeoParquet`

#### 7. Skip Map Generation
Control whether to create map visualizations.

- **Skip**: Check this to skip map generation
  - Recommended for large datasets to improve performance
  - Default: unchecked (maps will be generated)

### Advanced Configuration

These optional settings provide additional control over your workflow:

#### Subject Group Options

##### Include Details
Whether or not to include observation details in the output data.

- **Include Details**: Include additional details from each observation
  - Default: `false` (not included)
  - check to include detailed observation information

##### Include Subject Source Details
Whether or not to include subject source details in the output data.

- **Include Subject Source Details**: Include information about the tracking device/source
  - Default: `false` (not included)
  - Check to include device and source information

##### Observation Exclusion Filter
Control which observations are returned based on EarthRanger's exclusion flags. Exclusion flags are data quality markers set on each observation in EarthRanger.

- **Filter**: Select which observations to include based on their exclusion status
  - `clean` (default): Only observations that have not been flagged as problematic
  - `none`: All observations regardless of exclusion flags (raw data)
  - `manually_filtered`: Only observations that were manually flagged
  - `automatically_filtered`: Only observations that were automatically flagged
  - `manually_and_automatically_filtered`: Observations flagged either manually or automatically

#### Process Observations

##### Bounding Box
Limit observations to a geographic area.

- **Bounding Box**: Restrict data to specific coordinates
  - Default: entire world (-180 to 180 longitude, -90 to 90 latitude)
  - Example: Set bounds for your study area to exclude observations outside the region

##### Filter Exact Point Coordinates
Exclude observations at specific coordinates.

- **Filter Exact Point Coordinates**: Remove data at exact locations
  - Useful for filtering out test data or GPS outliers
  - Example: `[{"Latitude": 0.0, "Longitude": 0.0}, {"Latitude": 180.0, "Longitude": 90.0}]`

##### Trajectory Segment Filter
Filter track data by setting limits on track segment length, duration, and speed. Segments outside these bounds are removed, reducing noise and focusing on meaningful movement patterns.

- **Minimum Segment Length (Meters)**: Shortest distance allowed for a segment
  - Default: `0.001`
  - Example: `0.001` (include very short movements)
- **Maximum Segment Length (Meters)**: Longest distance allowed for a segment
  - Default: `100000` (100 km)
  - Example: `10000` (1 km - filter out unrealistic jumps)
- **Minimum Segment Duration (Seconds)**: Shortest time allowed for a segment
  - Default: `1`
  - Example: `1` (minimum 1 second between observations)
- **Maximum Segment Duration (Seconds)**: Longest time allowed for a segment
  - Default: `172800` (48 hours)
  - Example: `21600` (6 hours - filter out large gaps)
- **Minimum Segment Speed (Kilometers per Hour)**: Slowest speed allowed
  - Default: `0.01`
  - Example: `0.01` (include stationary or very slow movement)
- **Maximum Segment Speed (Kilometers per Hour)**: Fastest speed allowed
  - Default: `500`
  - Example: `10.0` (filter out unrealistic speeds for elephants)

##### Process Columns
Customize which columns appear in your output.

- **Drop Columns**: List of columns to drop from the output
  - Default includes common internal/system columns: `location`, `patrol_serial_number`, `patrol_status`, `patrol_subject`, `patrol_type__value`, `subject__content_type`, `subject__device_status_properties`, `subject__user`
  - Modify the list based on your requirements - add columns you want to hide or remove columns you want to keep

##### Apply SQL Query
Advanced users can filter or transform data using SQL.

- **Query**: Write a SQL query to filter or modify the data
  - Use `df` as the table name
  - Example: `SELECT * FROM df WHERE speed_kmhr > 5.0`
  - Leave empty to skip
- **Columns**: Optional list of column names to include in the SQL query
  - Use this to exclude columns with unsupported data types (list, dict)
  - Leave empty to include all columns

#### Map Base Layers
Customize the background maps for your visualizations.

- **Base Maps**: Select one or more base layers
  - Available options: Open Street Map, Roadmap, Satellite, Terrain, LandDx, USGS Hillshade or custom layers with a URL
  - Default: Terrain and Satellite layers
  - The first layer will appear on the bottom

## Running the Workflow

Once you've configured all the settings:

1. **Review your configuration**
   - Double-check your time range, data source, and subject group

2. **Save and run**
   - Click the "Submit" and the workflow will show up in "My Workflows" table button in Ecoscope Desktop
   - Click on "Run" and the workflow will begin processing

3. **Monitor progress and wait for completion**
   - You'll see status updates as the workflow runs
   - Processing time depends on:
     - The size of your date range
     - Number of subjects in the group
     - Number of observations in the system
   - The workflow completes with status "Success" or "Failed"

## Understanding Your Results

After the workflow completes successfully, you'll find your outputs in the designated output folder.

### Data Outputs

Your subject tracking data will be saved in the format(s) you selected:


- **File formats**: CSV, GeoParquet, and/or GPKG (based on your selection)
- **Opens in**: Microsoft Excel, Google Sheets (CSV), Python/R (GeoParquet), QGIS/ArcGIS (GPKG)
- **Best for**:
  - CSV: Quick data review and analysis
  - GeoParquet: Large datasets, programmatic analysis
  - GPKG: Spatial analysis in GIS software
- **Contents**: All trajectory segment data in tabular format with one row per segment
  - **segment_start**: Start time of the trajectory segment
  - **timespan_seconds**: Duration of the segment in seconds
  - **speed_kmhr**: Speed of movement in kilometers per hour
  - **subject__name**: Name of the subject (animal/asset)
  - **subject__id**: Unique identifier for the subject
  - **subject__sex**: Sex of the subject (if applicable)
  - **geometry**: Geographic line geometry representing the movement path
  - Additional columns based on your configuration and filters

### Visual Outputs (When Maps are Generated)

If you didn't skip map generation, you'll also receive:

#### Interactive Map
- **Format**: HTML file or embedded in dashboard
- **Features**:
  - Trajectory segments plotted as lines showing movement paths
  - Trajectories colored by subject (different colors for each animal/asset)
    - We respect the subject colors set on EarthRanger unless there are subjects with missing/duplicate colors
  - Interactive - click on segments to see details
  - Tooltip information includes:
    - Start time of the segment
    - Duration in seconds
    - Speed in kilometers per hour
    - Nighttime indicator
    - Subject name and sex
  - Base map layers you selected (satellite, terrain, etc.)
  - Zoom and pan capabilities
  - Legend showing subject names and their corresponding colors

### Grouped Outputs

If you configured data grouping:
- You'll receive separate files for each group. Each file contains only the trajectory segments for that time period or category
- Maps (if generated) will also be separated by group, with each group view selectable in the dashboard

## Common Use Cases & Examples

Here are some typical scenarios and how to configure the workflow for each:

### Example 1: Simple Weekly Tracking Report
**Goal**: Download all tracking data for a specific week to review in Excel

**Configuration**:
- **Time Range**:
  - Since: `2015-01-01T00:00:00`
  - Until: `2015-01-07T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Subject Group Name**: `"Subjects"`
- **Filetypes**: Select `CSV`
- **Skip Map Generation**: Checked (for faster processing)

**Result**: Single CSV file with all trajectory segments for the Ecoscope subject group from January 1-7, 2015

---

### Example 2: Monthly Grouped Analysis
**Goal**: Separate files for each month to track movement patterns over the year

**Configuration**:
- **Time Range**:
  - Since: `2015-01-01T00:00:00`
  - Until: `2015-12-31T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Subject Group Name**: `"Subjects"`
- **Group Data**:
  - Select `"%B"` (Month name: January, February, etc.)
- **Filetypes**: Select `CSV` and `GeoParquet`
- **Skip Map Generation**: Unchecked

**Result**:
- 12 separate CSV and GeoParquet files, one for each month
- Interactive maps for each month showing subject trajectories

---

### Example 3: Filtered by Geographic Area
**Goal**: Download only tracking data within a specific study area

**Configuration**:
- **Time Range**: Your desired date range
- **Subject Group Name**: Your subject group
- **Bounding Box** (Advanced):
  - Set coordinates for your area of interest
  - Example: Min Longitude: 37.0, Max Longitude: 38.0, Min Latitude: -1.0, Max Latitude: 0.0
- **Trajectory Segment Filter** (Advanced):
  - **Min Segment Length**: `0.001` meters
  - **Max Segment Length**: `1000` meters (1 km - filter out GPS jumps)
  - **Min Time**: `1` second
  - **Max Time**: `21600` seconds (6 hours - filter out large gaps)
  - **Min Speed**: `0.01` km/hr
  - **Max Speed**: `20` km/hr
- **Filetypes**: Select preferred formats

**Result**: Trajectory segments only from within your specified geographic boundaries with reasonable moving patterns

---

### Example 4: Large Dataset - Skip Maps
**Goal**: Download tracking data for many subjects over a long period without generating maps

**Configuration**:
- **Time Range**:
  - Since: `2015-01-01T00:00:00`
  - Until: `2015-12-31T23:59:59`
- **Subject Group Name**: Your subject group
- **Filetypes**: Select `GeoParquet` (most efficient for large datasets)
- **Skip Map Generation**: Checked

**Result**:
- Efficient GeoParquet file with full year of tracking data
- Faster processing without map generation overhead

## Troubleshooting

### Common Issues and Solutions

#### Workflow fails to start
**Problem**: The workflow won't run or immediately fails

**Solutions**:
- Verify your EarthRanger data source is properly configured
- Check that you have network connectivity to the EarthRanger server
- Ensure your credentials haven't expired
- Confirm the data source name matches exactly

#### No observations returned
**Problem**: Workflow completes but produces empty results

**Solutions**:
- Verify the date range is correct (start date should be before end date)
- Check that the subject group name is spelled exactly as it appears in EarthRanger
- Visit `https://<your-site>.pamdas.org/admin/observations/subjectgroup/` to confirm subject group names
- Verify the subject group has observations during the specified time range
- Try a broader date range to verify observations exist
- Check if trajectory segment filters are too restrictive (temporarily use default values)

#### Workflow runs very slowly
**Problem**: The workflow takes an extremely long time to complete

**Solutions**:
- Enable "Skip Map Generation" for large datasets
- Reduce the date range to smaller time periods
- Process data in smaller batches (by week or month instead of year)
- Consider using only GeoParquet format (most efficient) instead of multiple formats
- The first run may take longer as the environment gets warmed up. The following ones should be faster.

#### Authentication errors
**Problem**: Errors related to login or permissions

**Solutions**:
- Re-configure your EarthRanger data source in Ecoscope Desktop
- Verify your user account has permission to access subject data in EarthRanger
- Check that your account has permission to view the specific subject group

#### Map won't generate
**Problem**: Data downloads successfully but no map is created

**Solutions**:
- Ensure "Skip Map Generation" is unchecked
- Verify your trajectory segments have valid geometry/location data
- Try using default base map settings
- Check that observations have coordinate data (not just null values)

#### All trajectory segments filtered out
**Problem**: Workflow completes but trajectory data is empty or has very few segments

**Solutions**:
- Review your trajectory segment filter settings
- Temporarily set all filters to default values to see full data
- Check if max speed is too low for your subjects' typical movement
- Verify min/max segment length matches your subjects' movement patterns
- Increase max time between observations if subjects have infrequent GPS fixes
