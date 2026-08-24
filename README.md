
# 1. Read Raw JSON (with the multiline fix!)
df_json_raw = spark.read.format("json") \
    .option("multiline", "true") \
    .load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.json")

# 2. Read Raw CSV
df_csv_raw = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.csv")

# 3. Save as Bronze Delta Tables
df_json_raw.write.format("delta").mode("overwrite").saveAsTable("bronze_json_data")
df_csv_raw.write.format("delta").mode("overwrite").saveAsTable("bronze_csv_data")

print("Bronze ingestion complete!")
