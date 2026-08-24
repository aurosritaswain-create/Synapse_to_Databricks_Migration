import json
from pyspark.sql.functions import expr

# 1. THE SYNAPSE OBJECT (Simulating your SilverToGold.json file)
synapse_gold_json = """
{
    "name": "SilverToGold",
    "properties": {
        "type": "MappingDataFlow",
        "sources": [{"name": "silver_sales"}],
        "transformations": [
            {
                "name": "AggregateRevenue",
                "group_by_column": "department",
                "aggregate_expression": "sum(sales_amount) as total_revenue"
            }
        ]
    }
}
"""

# 2. THE PARSER 
print("--- 1. READING SYNAPSE GOLD OBJECT ---")
gold_obj = json.loads(synapse_gold_json)

gold_pipeline_name = gold_obj["name"]
group_col = gold_obj["properties"]["transformations"][0]["group_by_column"]
agg_expr = gold_obj["properties"]["transformations"][0]["aggregate_expression"]

print(f"Successfully read Pipeline: {gold_pipeline_name}")
print(f"Found Group By Rule: '{group_col}'")
print(f"Found Aggregation Rule: '{agg_expr}'\n")

# 3. THE EXECUTION 
print("--- 2. EXECUTING MIGRATED GOLD LOGIC ---")

# Fake Silver Data
silver_data = [("Hardware", 50.0), ("Hardware", 100.0), ("Garden", 20.0)]
silver_df = spark.createDataFrame(silver_data, ["department", "sales_amount"])

print("Clean Silver Data:")
silver_df.show()

# Automatically apply the extracted grouping and aggregation rules!
print("Applying extracted Synapse aggregation automatically...")
gold_df = silver_df.groupBy(group_col).agg(expr(agg_expr))

print("Final Gold Summary Data:")
gold_df.show()
