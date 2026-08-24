# 1. READ YOUR ORIGINAL WEEK 2 GOLD TABLE
# (This is the table you saved manually during your earlier work)
original_gold_df = spark.read.table("gold_csv_summary")

# 2. RUN THE COMPARISON
print("--- VALIDATING AUTOMATED RESULTS VS ORIGINAL RESULTS ---")

# Check 1: Do they have the same number of rows?
original_count = original_gold_df.count()
automated_count = gold_df.count()

print(f"Original Pipeline Row Count: {original_count}")
print(f"Automated Pipeline Row Count: {automated_count}")

if original_count == automated_count:
    print("✅ Row counts match perfectly!")
else:
    print("❌ Row counts do not match.")

# Check 2: Is the actual data identical?
# Using 'exceptAll' checks if there are any rows in one table that don't exist in the other
differences_1 = original_gold_df.exceptAll(gold_df)
differences_2 = gold_df.exceptAll(original_gold_df)

diff_count_1 = differences_1.count()
diff_count_2 = differences_2.count()

if diff_count_1 == 0 and diff_count_2 == 0:
    print("✅ Data validation PASSED! Both pipelines produced identical results.")
else:
    print(f"❌ Validation FAILED. Found discrepancies in the data.")
    print("Rows in original but not in automated:")
    differences_1.show()
    print("Rows in automated but not in original:")
    differences_2.show()
