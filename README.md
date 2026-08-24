ffrom pyspark.sql.functions import sum

# 1. Read from the clean Silver tables
df_json_silver = spark.read.table("silver_json_data")
df_csv_silver = spark.read.table("silver_csv_data")

# 2. Summarize JSON: Group by date
df_json_gold = df_json_silver.groupBy("purchase_date") \
    .agg(sum("total_after_discount").alias("total_daily_revenue"))

# 3. Summarize CSV: Group by department
df_csv_gold = df_csv_silver.groupBy("department") \
    .agg(sum("total_after_discount").alias("total_csv_revenue"))

# 4. Save as Gold Delta Tables
df_json_gold.write.format("delta").mode("overwrite").saveAsTable("gold_json_summary")
df_csv_gold.write.format("delta").mode("overwrite").saveAsTable("gold_csv_summary")

print("Gold aggregations complete! Pipeline finished.")
