# Spotify Azure Data Warehouse Project

A cloud-based data warehouse solution for analyzing Spotify streaming data using Azure Databricks and SQL. This project implements a star schema design with incremental data loading capabilities using Change Data Capture (CDC).

## Overview

This project provides a scalable data warehouse architecture for storing and analyzing Spotify streaming metrics, including user information, artist details, track metadata, and streaming facts. The solution is designed to handle both initial full data loads and incremental updates efficiently.

## Architecture

The data warehouse follows a **star schema** design pattern optimized for analytical queries:

```
                    ┌─────────────┐
                    │  DimDate    │
                    └──────┬──────┘
                           │
    ┌──────────┐      ┌───┴────┐      ┌──────────┐
    │ DimUser  │──────┤FactStream├────│ DimTrack │
    └──────────┘      └───┬────┘      └──────────┘
                           │
                    ┌──────┴──────┐
                    │  DimArtist  │
                    └─────────────┘
```

### Technology Stack

- **Cloud Platform**: Microsoft Azure
- **Data Processing**: Azure Databricks
- **Database**: SQL Database
- **ETL Pattern**: Change Data Capture (CDC)

## Database Schema

### Dimension Tables

#### DimUser
Stores user profile information and subscription details.

| Column | Type | Description |
|--------|------|-------------|
| user_id | INT | Primary key |
| user_name | VARCHAR(255) | User's full name |
| country | VARCHAR(255) | User's country |
| subscription_type | VARCHAR(50) | Free, Premium, or Family |
| start_date | DATE | Subscription start date |
| end_date | DATE | Subscription end date |
| updated_at | DATETIME | Last update timestamp |

#### DimArtist
Contains artist information and metadata.

| Column | Type | Description |
|--------|------|-------------|
| artist_id | INT | Primary key |
| artist_name | VARCHAR(255) | Artist's name |
| genre | VARCHAR(100) | Music genre |
| country | VARCHAR(100) | Artist's country |
| updated_at | DATETIME | Last update timestamp |

#### DimTrack
Stores track/song details.

| Column | Type | Description |
|--------|------|-------------|
| track_id | INT | Primary key |
| track_name | VARCHAR(255) | Track title |
| artist_id | INT | Foreign key to DimArtist |
| album_name | VARCHAR(255) | Album name |
| duration_sec | INT | Track duration in seconds |
| release_date | DATE | Release date |
| updated_at | DATETIME | Last update timestamp |

#### DimDate
Date dimension for time-based analysis.

| Column | Type | Description |
|--------|------|-------------|
| date_key | INT | Primary key |
| date | DATE | Actual date |
| day | INT | Day of month |
| month | INT | Month number |
| year | INT | Year |
| weekday | VARCHAR(20) | Day of week name |

### Fact Table

#### FactStream
Central fact table storing streaming events.

| Column | Type | Description |
|--------|------|-------------|
| stream_id | BIGINT | Primary key |
| user_id | INT | Foreign key to DimUser |
| track_id | INT | Foreign key to DimTrack |
| date_key | INT | Foreign key to DimDate |
| listen_duration | INT | Actual listen time in seconds |
| device_type | VARCHAR(50) | Device used for streaming |
| stream_timestamp | DATETIME | When the stream occurred |

## Project Structure

```
Spotify_Azure_Project/
├── README.md                           # This file
├── Databricks Code/
│   ├── spotify_dab.dbc                # Databricks notebook archive
│   └── cdc.json                       # CDC configuration for Databricks
├── source scripts/
│   ├── spotify_initial_load.sql       # Full initial data load script
│   └── spotify_incremental_load.sql   # Incremental CDC-based load
├── cdc.json                           # CDC tracking file (stores last run date)
├── loop_input                         # Table configuration for incremental loads
└── empty.json                         # Placeholder file
```

## Features

### 1. Initial Data Load
The `spotify_initial_load.sql` script provides:
- Complete schema creation (DROP and CREATE statements)
- Full data population for all dimension and fact tables
- Sample data for testing and development

### 2. Incremental Loading with CDC
The incremental load system enables efficient updates by:
- Tracking changes using timestamp columns (`updated_at`, `stream_timestamp`)
- Processing only new or modified records
- Maintaining data freshness without full reloads
- Configurable CDC columns per table

### 3. CDC Configuration
The `loop_input` file defines CDC parameters for each table:
```json
{
  "schema": "dbo",
  "table": "DimUser",
  "cdc_col": "updated_at",
  "from_date": ""
}
```

## Getting Started

### Prerequisites
- Azure subscription
- Azure Databricks workspace
- SQL Database instance
- Access credentials for Azure services

### Setup Instructions

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Spotify_Azure_Project
   ```

2. **Configure Azure Resources**
   - Create an Azure Databricks workspace
   - Set up an Azure SQL Database
   - Configure network connectivity between services

3. **Initial Data Load**
   ```sql
   -- Run the initial load script in your SQL database
   -- This will create all tables and populate sample data
   source scripts/spotify_initial_load.sql
   ```

4. **Import Databricks Notebooks**
   - Upload `spotify_dab.dbc` to your Databricks workspace
   - Configure cluster settings and dependencies

5. **Configure CDC**
   - Update `cdc.json` with your initial load date
   - Modify `loop_input` if needed for your table structure

6. **Run Incremental Loads**
   ```sql
   -- Execute incremental load for updated records
   source scripts/spotify_incremental_load.sql
   ```

## Usage

### Running Initial Load
Execute the initial load script to set up your data warehouse:
```bash
sqlcmd -S <server> -d <database> -i "source scripts/spotify_initial_load.sql"
```

### Running Incremental Updates
Schedule regular incremental loads to capture changes:
```bash
sqlcmd -S <server> -d <database> -i "source scripts/spotify_incremental_load.sql"
```

### Databricks Integration
1. Open the imported notebooks in Databricks
2. Configure connection strings to your SQL Database
3. Run notebooks according to your ETL schedule

## Data Analytics Use Cases

This data warehouse supports various analytical queries:

- **User Behavior Analysis**: Track listening patterns by subscription type
- **Artist Performance**: Analyze streaming metrics by artist and genre
- **Geographic Insights**: Compare streaming trends across countries
- **Time-based Analysis**: Identify peak listening hours and seasonal trends
- **Track Popularity**: Measure track performance over time
- **Device Usage**: Understand platform preferences (mobile, desktop, etc.)

## Maintenance

### CDC Updates
The CDC system automatically tracks the last processed date in `cdc.json`:
```json
{"cdc":"2025-01-01"}
```

Update this file to control the incremental load window.

### Performance Optimization
- Create appropriate indexes on foreign key columns
- Partition large fact tables by date
- Regularly update table statistics
- Monitor query performance and adjust as needed

## Contributing

When contributing to this project:
1. Follow the existing schema design patterns
2. Maintain backward compatibility in SQL scripts
3. Update CDC configurations for new tables
4. Test both initial and incremental loads
5. Document any schema changes

## License

This project is available for educational and commercial use.

## Contact

For questions or support, please open an issue in the repository.

---
