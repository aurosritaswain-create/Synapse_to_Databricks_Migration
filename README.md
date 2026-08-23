
# 1. Read the JSON file into a DataFrame
df_json = spark.read.format("json").load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.json")

# 2. Read the CSV file into a DataFrame
df_csv = spark.read.format("csv") \
  .option("header", "true") \
  .option("inferSchema", "true") \
  .load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.csv")

# 3. View the JSON data (This is like Synapse's Data Preview!)
display(df_json)
