import json
from pyspark.sql import SparkSession

# 1. THE SYNAPSE OBJECT (Simulating your BronzeToSilver.json file)
# Instead of you typing the rules, we load the raw JSON that Synapse generated.
synapse_dataflow_json = """
{
    "name": "BronzeToSilver",
    "properties": {
        "type": "MappingDataFlow",
        "sources": [{"name": "raw_sales"}],
        "transformations": [
            {
                "name": "FilterNegativeSales",
                "filter_expression": "sales_amount >= 0"
            }
        ]
    }
}
"""

# 2. THE PARSER (The tool that reads the Synapse file)
print("--- 1. READING SYNAPSE OBJECT ---")
# In a real scenario, this would be: json.load(open("/path/to/BronzeToSilver.json"))
synapse_obj = json.loads(synapse_dataflow_json)

pipeline_name = synapse_obj["name"]
source_name = synapse_obj["properties"]["sources"][0]["name"]
# We extract the rule DIRECTLY from the JSON text!
extracted_rule = synapse_obj["properties"]["transformations"][0]["filter_expression"]

print(f"Successfully read Synapse Pipeline: {pipeline_name}")
print(f"Found Source: {source_name}")
print(f"Found Transformation Rule: '{extracted_rule}'\n")


# 3. THE EXECUTION (Applying the extracted rule to PySpark)
print("--- 2. EXECUTING MIGRATED LOGIC ---")

# Create quick fake data to prove the filter works
data = [("T001", "Drill", 50.0), ("T002", "Hammer", -10.0)]
raw_df = spark.createDataFrame(data, ["transaction_id", "product", "sales_amount"])

print("Raw Data:")
raw_df.show()

# We pass the STRING we extracted from the JSON directly into PySpark's where() clause
print(f"Applying extracted Synapse rule automatically...")
clean_df = raw_df.where(extracted_rule)

print("Final Curated Data:")
clean_df.show()

