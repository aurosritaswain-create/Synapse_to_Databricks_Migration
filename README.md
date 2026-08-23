
%sql
SELECT 
    department, 
    ROUND(total_csv_revenue, 2) AS total_revenue
FROM gold_csv_summary
ORDER BY total_revenue DESC;
