# Metabase Dashboard Migration: Exasol to StarRocks

This repository contains tools and scripts to migrate Metabase dashboards from Exasol to StarRocks database, preserving SQL compatibility, visualization settings, and column formatting.

## 🤖 AI Assistant Notes

**For Cursor AI:** This is a Python-based migration tool that converts Metabase dashboards from Exasol to StarRocks. The main entry point is `migrate_dashboard.py`. Key files to understand:
- `migrate_dashboard.py` - Main migration script (1300+ lines)
- `column_mapping_config.json` - Configuration for column mappings and formatting
- `metabase_migrator.py` - Core migration library
- `fetch_metadata.py` - Metadata fetching utility

**Common AI Tasks:**
- Change dashboard ID in `migrate_dashboard.py` line ~15
- Add new column mappings in `column_mapping_config.json`
- Debug SQL conversion issues in `clean_sql_for_starrocks()` function
- Add new dashboard configurations in `DASHBOARD_CONFIG` dictionary

## 🎯 Overview

The migration process automatically:
- Converts Exasol-specific SQL syntax to StarRocks compatibility
- Maps database field IDs from Exasol to StarRocks
- Preserves column formatting (percentages, currency, conditional formatting)
- Maintains original column display names
- Updates template tags and parameters
- Validates migrated questions

## 📁 Repository Structure

```
meta migration/
├── README.md                           # This file
├── config.py                           # Metabase connection configuration
├── metabase_migrator.py               # Core migration library
├── migrate_dashboard.py               # Main migration script
├── column_mapping_config.json         # Column mappings and formatting rules
├── fetch_metadata.py                  # Script to fetch Exasol/StarRocks metadata
├── migrations/                        # Migration mapping files
│   └── migration_mapping.json        # Field ID mappings between databases
└── inspections/                       # Dashboard inspection data
    └── dashboard_*.json              # Cached dashboard metadata
```

## 🚀 Quick Start

### 1. Prerequisites

- Python 3.7+
- Access to Metabase instance
- Exasol and StarRocks database credentials
- Cursor IDE (recommended)

### 2. Setup

1. **Clone or download this repository**
2. **Install Python dependencies:**
   ```bash
   pip install requests
   ```

3. **Configure Metabase connection:**
   Edit `config.py` with your Metabase credentials:
   ```python
   METABASE_CONFIG = {
       "base_url": "https://your-metabase-instance.com",
       "username": "your-username",
       "password": "your-password"
   }
   ```

### 3. Initial Setup (One-time)

Before migrating dashboards, you need to fetch metadata mappings:

```bash
python3 fetch_metadata.py
```

This creates `migrations/migration_mapping.json` with field ID mappings between Exasol and StarRocks.

### 4. Migrate a Dashboard

To migrate a specific dashboard:

1. **Edit the dashboard ID in `migrate_dashboard.py`:**
   ```python
   def main():
       dashboard_id = 421  # Change this to your target dashboard
   ```

2. **Run the migration:**
   ```bash
   python3 migrate_dashboard.py
   ```

## 📋 Configuration Files

### `column_mapping_config.json`

This file contains all column mappings and formatting rules:

```json
{
  "column_mappings": {
    "exasol_to_starrocks": {
      "AR_PERIOD1": "ar_period1",
      "DELTA_AR": "delta_ar"
      // ... more mappings
    }
  },
  "dashboard_specific_mappings": {
    "421": {
      "description": "Dashboard 421",
      "additional_mappings": {}
    }
  },
  "formatting_preservation": {
    "percentage_columns": ["AR_PERIOD1", "AR_PERIOD2"],
    "currency_columns": ["TURNOVER_EUR_PERIOD1"],
    "mini_bar_columns": ["DELTA_AR"]
  },
  "display_name_mappings": {
    "ar_period1": "AR: period 1",
    "delta_ar": "Delta AR"
  }
}
```

**Key Sections:**
- `column_mappings`: Maps Exasol column names to StarRocks
- `dashboard_specific_mappings`: Additional mappings for specific dashboards
- `formatting_preservation`: Defines which columns get percentage/currency formatting
- `display_name_mappings`: Preserves original column titles in visualizations

### `migrate_dashboard.py`

The main migration script. Key functions:

- **`clean_sql_for_starrocks()`**: Converts Exasol SQL to StarRocks compatibility
- **`update_question()`**: Updates individual questions in Metabase
- **`validate_question_response()`**: Tests migrated questions
- **`enhance_visualization_settings_with_formatting()`**: Applies formatting rules

## 🔧 Migration Process

### Step 1: Dashboard Inspection
- Fetches dashboard metadata from Metabase
- Identifies all questions and their types
- Caches inspection data for reuse

### Step 2: Question Processing
For each native SQL question:
- Fetches current SQL and visualization settings
- Applies StarRocks compatibility fixes
- Maps column names and field IDs
- Preserves formatting and display names

### Step 3: Validation
- Tests each migrated question
- Verifies SQL execution
- Checks data retrieval

## 🎨 Formatting Preservation

The migration preserves:

- **Percentage formatting**: Applied to acceptance rate columns
- **Currency formatting**: Applied to turnover/amount columns  
- **Mini bars**: Applied to delta columns
- **Conditional formatting**: Color-coded rules for delta values
- **Column titles**: Original display names like "AR: period 1"

## 🔍 SQL Compatibility Fixes

The script automatically converts:

- `NULLIFZERO(value)` → `NULLIF(value, 0)`
- `zeroifnull(value)` → `ifnull(value, 0)`
- `to_char()` → `char()`
- `to_date()` → `date()`
- `json_value()` → `parse_json()->'field'`
- `listagg()` → `group_concat()`
- `median()` → `PERCENTILE_CONT(column, 0.5)`
- Schema references: `mart.table` → `MART__TABLE`

## 📊 Dashboard Configuration

Add dashboard-specific settings in `migrate_dashboard.py`:

```python
DASHBOARD_CONFIG = {
    421: {
        "granularity_to_static_list": True,
        "granularity_static_values": [
            ["hour"], ["day"], ["week"], ["month"], ["quarter"], ["year"]
        ],
        "granularity_default": "week"
    }
}
```

## 🤖 AI-Friendly Code Examples

### Common AI Tasks

**1. Change Dashboard ID:**
```python
# In migrate_dashboard.py, around line 15
def main():
    dashboard_id = 422  # AI: Change this number to migrate different dashboard
```

**2. Add New Column Mapping:**
```json
// In column_mapping_config.json
{
  "column_mappings": {
    "exasol_to_starrocks": {
      "NEW_COLUMN": "new_column"  // AI: Add new mappings here
    }
  }
}
```

**3. Add New Dashboard Configuration:**
```python
# In migrate_dashboard.py, in DASHBOARD_CONFIG dictionary
422: {
    "granularity_to_static_list": True,
    "granularity_static_values": [
        ["hour"], ["day"], ["week"]
    ],
    "granularity_default": "week"
}
```

**4. Debug SQL Conversion:**
```python
# In migrate_dashboard.py, in clean_sql_for_starrocks() function
# AI: Add debug prints here to see SQL transformations
print(f"  🔍 Debug: Original SQL: {sql[:200]}...")
# ... conversion logic ...
print(f"  🔍 Debug: Converted SQL: {sql[:200]}...")
```

### File Relationships for AI

- **`migrate_dashboard.py`** imports from `metabase_migrator.py` and `config.py`
- **`column_mapping_config.json`** is loaded by `migrate_dashboard.py`
- **`migrations/migration_mapping.json`** is created by `fetch_metadata.py`
- **`inspections/dashboard_*.json`** are created by dashboard inspection functions

### Common AI Interaction Patterns

1. **"Migrate dashboard X"** → Change `dashboard_id` in `migrate_dashboard.py`
2. **"Add column mapping for Y"** → Add to `column_mapping_config.json`
3. **"Fix SQL error"** → Look at `clean_sql_for_starrocks()` function
4. **"Add new dashboard config"** → Add to `DASHBOARD_CONFIG` dictionary
5. **"Debug migration issue"** → Add debug prints in `update_question()` function

## 🚨 Troubleshooting

### Common Issues

1. **"No mapping found for field-id"**
   - Run `fetch_metadata.py` to update field mappings
   - Check if the field exists in both databases

2. **"SQL Error" during validation**
   - Check the specific SQL error in logs
   - May need manual SQL adjustments for complex queries

3. **"Authentication failed"**
   - Verify Metabase credentials in `config.py`
   - Check network connectivity to Metabase

4. **"Column formatting not preserved"**
   - Verify column names in `column_mapping_config.json`
   - Check if columns exist in the migrated SQL

### Debug Mode

Enable detailed logging by modifying the script to print more information:

```python
# Add debug prints in update_question() function
print(f"  🔍 Debug: SQL before cleaning: {current_sql[:200]}...")
print(f"  🔍 Debug: SQL after cleaning: {cleaned_sql[:200]}...")
```

### AI Debugging Tips

**For Cursor AI:** When debugging issues:
1. Look at the function `clean_sql_for_starrocks()` for SQL conversion problems
2. Check `update_question()` function for Metabase API issues
3. Review `validate_question_response()` for validation problems
4. Examine `column_mapping_config.json` for mapping issues

## 📈 Migration Results

After successful migration, you'll see:

```
🎉 Migration Summary:
📊 Total dashcards processed: 12
📝 Native SQL questions found: 8
✅ Successfully migrated: 8/8 questions
📊 Dashboard 421 migration completed!

🎊 FINAL RESULT: Migration successful!
✅ All questions migrated and validated
✅ Dashboard 421 is now operational with StarRocks
```

## 🔄 Adding New Dashboards

To migrate additional dashboards:

1. **Add dashboard configuration:**
   ```python
   # In migrate_dashboard.py
   DASHBOARD_CONFIG = {
       # ... existing configs
       422: {
           "granularity_to_static_list": True,
           "granularity_static_values": [["hour"], ["day"], ["week"]],
           "granularity_default": "week"
       }
   }
   ```

2. **Add dashboard-specific mappings:**
   ```json
   // In column_mapping_config.json
   "dashboard_specific_mappings": {
       "422": {
           "description": "New Dashboard",
           "additional_mappings": {
               "NEW_COLUMN": "new_column"
           }
       }
   }
   ```

3. **Update the dashboard ID and run:**
   ```python
   dashboard_id = 422  # In migrate_dashboard.py
   ```

## 📝 Best Practices

1. **Always validate migrations** - Check the validation results
2. **Test with small dashboards first** - Start with dashboards having few questions
3. **Backup before migration** - Export dashboard configurations if needed
4. **Review SQL changes** - Check the logs for any manual intervention needed
5. **Update mappings regularly** - Re-run `fetch_metadata.py` when database schemas change

## 🤝 Support

For issues or questions:
1. Check the troubleshooting section above
2. Review the migration logs for specific error messages
3. Verify database connectivity and permissions
4. Ensure all required files are present and properly configured

## 📄 License

This migration tool is provided as-is for internal use. Please ensure you have proper permissions to modify Metabase dashboards and database connections. 