from pyspark.sql.functions import col, upper

# 1. Read from the Bronze tables we just created
df_json_bronze = spark.read.table("bronze_json_data")
df_csv_bronze = spark.read.table("bronze_csv_data")

# 2. Clean JSON: Filter out null transactions
df_json_silver = df_json_bronze.filter(col("transaction_id").isNotNull())

# 3. Clean CSV: Create uppercase column and filter nulls
df_csv_silver = df_csv_bronze.withColumn("department_upper", upper(col("department"))) \
                             .filter(col("transaction_id").isNotNull())

# 4. Save as Silver Delta Tables
df_json_silver.write.format("delta").mode("overwrite").saveAsTable("silver_json_data")
df_csv_silver.write.format("delta").mode("overwrite").saveAsTable("silver_csv_data")

print("Silver transformations complete!")
