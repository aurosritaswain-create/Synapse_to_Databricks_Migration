
from pyspark.sql.functions import col, upper, sum

# ==========================================
# 1. BRONZE LAYER (Read Raw Data)
# ==========================================
# Read JSON dataset
df_json_raw = spark.read.format("json") \
    .load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.json")

# Read CSV dataset
df_csv_raw = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.csv")

# ==========================================
# 2. SILVER LAYER (Clean & Transform)
# ==========================================
# Clean JSON: Filter out null transactions
df_json_silver = df_json_raw.filter(col("transaction_id").isNotNull())

# Clean CSV: Create uppercase column and filter nulls
df_csv_silver = df_csv_raw.withColumn("department_upper", upper(col("department"))) \
                          .filter(col("transaction_id").isNotNull())

# Save as Silver Delta Tables
df_json_silver.write.format("delta").mode("overwrite").saveAsTable("silver_json_data")
df_csv_silver.write.format("delta").mode("overwrite").saveAsTable("silver_csv_data")

# ==========================================
# 3. GOLD LAYER (Aggregate & Summarize)
# ==========================================
# Summarize JSON: Group by date
df_json_gold = df_json_silver.groupBy("purchase_date") \
    .agg(sum("total_after_discount").alias("total_daily_revenue"))

# Summarize CSV: Group by department
df_csv_gold = df_csv_silver.groupBy("department") \
    .agg(sum("total_after_discount").alias("total_csv_revenue"))

# Save as Gold Delta Tables
df_json_gold.write.format("delta").mode("overwrite").saveAsTable("gold_json_summary")
df_csv_gold.write.format("delta").mode("overwrite").saveAsTable("gold_csv_summary")

# ==========================================
# 4. VIEW THE FINAL OUTPUT
# ==========================================
display(df_csv_gold)
