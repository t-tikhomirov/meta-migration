# Metabase Migration Guide: Exasol to StarRocks

This guide will help you migrate your Metabase dashboards from Exasol to StarRocks using the provided migration tool.

## 🚀 Quick Start

### Step 1: Install Dependencies
```bash
pip3 install -r requirements.txt
```

### Step 2: Configure Database Mappings
Edit `config.py` and add your table mappings:

```python
DATABASE_MAPPINGS = [
    DatabaseMapping(
        exasol_db="exasol",
        exasol_schema="mart", 
        exasol_table="transactions",
        starrocks_db="sr_mart",
        starrocks_table="transactions"
    ),
    # Add more mappings for your tables
]
```

### Step 3: Run Migration
```bash
python3 run_migration.py
```

## 📋 What Gets Migrated

### ✅ Native SQL Questions
- **Table References**: `MART.TRANSACTIONS` → `sr_mart.transactions`
- **Functions**: `ADD_DAYS()` → `DATE_ADD()`, `MEDIAN()` → `PERCENTILE_CONT()`
- **Syntax**: `LIMIT 100 OFFSET 0` → `LIMIT 0, 100`
- **Variables**: `{{start_date}}` preserved exactly

### 🔧 MBQL Questions
- Basic structure extraction (full conversion coming soon)
- Query metadata preservation

### 📊 Dashboards
- All dashboard questions
- Question metadata and settings
- Variable definitions

## 🔧 Configuration Options

### Database Mappings
Map your Exasol tables to StarRocks equivalents:

```python
@dataclass
class DatabaseMapping:
    exasol_db: str        # Exasol database name
    exasol_schema: str    # Exasol schema name  
    exasol_table: str     # Exasol table name
    starrocks_db: str     # StarRocks database name
    starrocks_table: str  # StarRocks table name
```

### Function Mappings
The tool automatically converts these Exasol functions:

| Exasol | StarRocks |
|--------|-----------|
| `ADD_DAYS()` | `DATE_ADD()` |
| `DAYS_BETWEEN()` | `DATEDIFF()` |
| `MEDIAN()` | `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ...)` |
| `INSTR()` | `LOCATE()` |
| `SUBSTR()` | `SUBSTRING()` |
| `CURRENT_TIMESTAMP` | `NOW()` |

### Migration Settings
```python
MIGRATION_SETTINGS = {
    "preserve_variables": True,    # Keep Metabase variables
    "backup_original_sql": True,   # Save original SQL
    "output_format": "json",       # Output format
    "include_metadata": True,      # Include question metadata
}
```

## 📊 Output Files

### migration_results.json
Contains detailed migration results:

```json
{
  "dashboard_id": 123,
  "dashboard_name": "Transaction Analytics",
  "questions": [
    {
      "question_id": 456,
      "question_name": "Daily Transactions",
      "type": "native",
      "original_sql": "SELECT * FROM MART.TRANSACTIONS...",
      "converted_sql": "SELECT * FROM sr_mart.transactions...",
      "variables": ["start_date", "end_date"],
      "validation": {
        "success": true,
        "variables_preserved": true,
        "tables_converted": 1,
        "functions_converted": 2
      }
    }
  ]
}
```

## 🧪 Testing

### Test SQL Converter
```bash
python3 test_converter.py
```

This will test:
- Table reference conversion
- Function mapping
- Variable preservation
- Syntax conversion

### Test Individual Queries
```python
from sql_converter import SQLConverter

converter = SQLConverter()
converted = converter.convert_sql("SELECT * FROM MART.TRANSACTIONS")
print(converted)
```

## 🔍 Troubleshooting

### Common Issues

#### 1. Authentication Failed
```
❌ Authentication failed: 401 - Unauthorized
```
**Solution**: Check credentials in `config.py`

#### 2. No Dashboards Found
```
❌ No dashboards found
```
**Solution**: Verify user has dashboard access permissions

#### 3. SQL Conversion Errors
```
❌ Error: Could not get details for question 123
```
**Solution**: Check Metabase API access and question permissions

#### 4. Table Mapping Issues
```
❌ Table mapping failed
```
**Solution**: Add missing table mappings to `config.py`

### Debug Mode
Enable detailed logging by modifying the logging level in `metabase_migrator.py`:

```python
logging.basicConfig(level=logging.DEBUG)
```

## 📝 Manual Review Checklist

After migration, review these items:

### ✅ SQL Conversion
- [ ] Table names converted correctly
- [ ] Functions mapped properly
- [ ] Variables preserved
- [ ] Syntax compatible with StarRocks

### ✅ Data Types
- [ ] Date/time functions work
- [ ] String functions compatible
- [ ] Aggregation functions correct

### ✅ Performance
- [ ] Queries optimized for StarRocks
- [ ] Indexes considered
- [ ] Partitioning strategy reviewed

## 🔄 Post-Migration Steps

### 1. Validate Queries
Test converted queries in StarRocks:
```sql
-- Test your converted query
SELECT * FROM sr_mart.transactions 
WHERE created_date >= '2023-01-01'
```

### 2. Update Metabase Connection
- Add StarRocks as a new data source
- Update dashboard questions to use new connection
- Test all variables and filters

### 3. Performance Tuning
- Review query execution plans
- Add appropriate indexes
- Consider StarRocks-specific optimizations

## 🛠️ Advanced Usage

### Custom Function Mappings
Add new function mappings in `config.py`:

```python
FUNCTION_MAPPINGS = {
    # ... existing mappings ...
    "YOUR_EXASOL_FUNC": "YOUR_STARROCKS_FUNC",
}
```

### Batch Processing
Process specific dashboards:

```python
from metabase_migrator import MetabaseMigrator, MetabaseConfig

migrator = MetabaseMigrator(config)
dashboards = migrator.get_dashboards()

# Migrate specific dashboard
for dashboard in dashboards:
    if dashboard['name'] == 'Target Dashboard':
        result = migrator.migrate_dashboard(dashboard)
        print(result)
```

### Export to Different Formats
Modify `MIGRATION_SETTINGS` to change output format:

```python
MIGRATION_SETTINGS = {
    "output_format": "csv",  # or "sql"
}
```

## 📞 Support

If you encounter issues:

1. **Check logs**: Review console output and migration_results.json
2. **Test components**: Run `test_converter.py` to verify SQL conversion
3. **Review configuration**: Ensure all table mappings are correct
4. **Check permissions**: Verify Metabase API access

## 🔐 Security Notes

- Credentials are stored in `config.py` - keep this file secure
- Consider using environment variables for production
- Review migration results before applying to production
- Test in a staging environment first

---

**Happy Migrating! 🎉** 