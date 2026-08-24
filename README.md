import json
from pyspark.sql import SparkSession
from pyspark.sql.functions import sum, col

# 1. SIMULATING THE SYNAPSE METADATA (Using your REAL Week 2 rules!)
synapse_metadata_repo = {
    "BronzeToSilver_Pipeline": {
        "type": "MappingDataFlow",
        "operation": "filter",
        "rule": "transaction_id IS NOT NULL" # Your actual cleaning rule
    },
    "SilverToGold_Pipeline": {
        "type": "MappingDataFlow",
        "operation": "aggregate",
        "group_by": "department",             # Your actual grouping column
        "agg_metric": "total_after_discount", # Your actual math column
        "agg_function": "sum"
    }
}

print("--- STEP 1: READING SYNAPSE OBJECTS DYNAMICALLY ---")
b2s_meta = synapse_metadata_repo["BronzeToSilver_Pipeline"]
s2g_meta = synapse_metadata_repo["SilverToGold_Pipeline"]

print(f"Loaded BronzeToSilver Rule: '{b2s_meta['rule']}'")
print(f"Loaded SilverToGold Rule: Group By '{s2g_meta['group_by']}', Summing '{s2g_meta['agg_metric']}'\n")


# 2. INGESTING YOUR ACTUAL WEEK 2 DATA
print("--- STEP 2: READING REAL WEEK 2 DATA ---")
# Reading directly from your Volume just like we did in the original script
raw_df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("/Volumes/workspace/default/raw_data_volume/big_box_home_improvement_dataset.csv")

print("Raw Bronze Data (First 5 rows):")
raw_df.show(5)


# 3. EXECUTING THE METADATA FACTORY
print("--- STEP 3: EXECUTING MIGRATED LOGIC ---")

# Executing Bronze -> Silver translation
print("Executing Silver Layer (Applying Filter extracted from Synapse):")
# Notice how we pass the rule string dynamically!
silver_df = raw_df.where(b2s_meta["rule"])
silver_df.show(5)

# Executing Silver -> Gold translation
print("Executing Gold Layer (Applying Aggregation extracted from Synapse):")
if s2g_meta["agg_function"] == "sum":
    gold_df = silver_df.groupBy(s2g_meta["group_by"]) \
                       .agg(sum(s2g_meta["agg_metric"]).alias("total_department_revenue"))
    gold_df.show(5)
