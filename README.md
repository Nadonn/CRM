

## 🔧 Cleaning Steps

1. **Header Cleaning**
   - Converted all headers to lower case.
   - Removed duplicates.
   - Trimmed and standardized column names.

2. **Blank Checking**
   - Found ~100,000 blank `customer_id` values (20% of total).
   - Replaced blanks with `"n/a"` instead of deletion to preserve useful sales data.

3. **Quantity Error Checking**
   - Detected 12,302 rows with negative quantity (returns).
   - Deleted 2,486 rows where `quantity < 0` and `customer_id = "n/a"` (invalid record).

4. **Country Column Cleaning
- Change EIRA to Irland and Channel Islands to Island
- delete RSA from data bacause there only 110 rows
- Change USA to United States of America

5. **price colume checking 
- filter and remove minus data there is only 3 rows

6. **description colume checking
- remove all outliners


### Cleaning Steps Excel process
**1.Column Standardization**
   - Converted all column headers to lowercase.
   - Removed duplicate columns.
   - Trimmed and standardized whitespace in column names.

**2.Handling Missing Data**
- Found ~100,000 missing customer_id entries (~20%).
- Decision: Replaced with "n/a" instead of deleting to retain potentially valuable records (e.g., sales trends).

**3.Quantity Cleaning**
- Detected 12,302 rows with negative quantity values (interpreted as returns).
- Removed 2,486 rows where: quantity < 0 and customer_id = "n/a" → likely invalid return records.

**4.Country Column Cleanup**
- Standardized country names: "EIRE" → Ireland "Channel Islands" → Ireland "USA" → United States of America
- Deleted country: "RSA" (only 110 rows; minimal impact, removed for consistency).

**5.Price Column Validation**
- Checked for negative values in price column.
- Removed 3 rows with negative pricing values (data entry errors).

**6.Description Column Cleaning**
- Removed outlier text entries in description field that were:Too long, Contained non-product text, Possibly code or noise

**7. invoicedate format for SQL
- From dd/mm/yyyy hh:mm to yyyy/mm/dd hh:mm 
  1. Text to colume
  2. TEXT(DATE(G2,F2,E2),"yyyy/mm/dd") & " " & TEXT(H2,"hh:mm")
