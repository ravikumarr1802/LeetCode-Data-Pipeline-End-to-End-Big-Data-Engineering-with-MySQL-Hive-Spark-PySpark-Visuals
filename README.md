# 🚀 LeetCode Data Pipeline: End to End Big Data Engineering with MySQL Hive Spark PySpark Visuals 📊

## 🌟 Project Overview

This project demonstrates a comprehensive data engineering pipeline, transforming data from a LeetCode dataset (likely problem data) into interactive visualizations. The pipeline utilizes a variety of big data technologies, showcasing proficiency in data ingestion, processing, storage, and analysis.

## 🛠️ Technologies Used

*   📄 **Data Source:** LeetCode Dataset (CSV format)
*   ↔️ **Data Transfer:** [WinSCP](https://winscp.net/eng/index.php) (for transferring data to the VM)
*   🗄️ **Relational Database:** [MySQL](https://www.mysql.com/)
*   📥 **Data Ingestion & ETL:** [Apache Sqoop](https://sqoop.apache.org/)
*   📦 **Data Warehousing:** [Apache Hive](https://hive.apache.org/) (with Partitions)
*   ⚙️ **Data Processing & Analytics:** [Apache Spark](https://spark.apache.org/) (Scala and Python)
*   💾 **Data Storage:** [HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) (Hadoop Distributed File System)
*   📈 **Visualization:** [Matplotlib](https://matplotlib.org/) (Python)

## 🗺️ Project Steps

1.  📥 **Data Acquisition:**
    *   The project begins with a CSV file (likely named `Leetcode.csv`) containing LeetCode problem data.
    *   This CSV file is transferred to a virtual machine (VM) using WinSCP. (See Figure 2)

2.  ➡️🗄️ **Data Ingestion into MySQL:**
    *   A MySQL database and table (`dsa`) are created.
    *   The `merged.csv` (which is a copy of the `Leetcode.csv`) is loaded into the `dsa` table using a `LOAD DATA INFILE` command. (See Figure 3)

3.  ➡️📦 **Data Transfer to Hive using Sqoop:**
    *   Sqoop is used to import data from the `dsa` table in MySQL into a Hive table (`ravi.dsa`). The `--hive-import` and related flags automate table creation in Hive. (See Figure 4)

4.  🧩 **Data Partitioning in Hive:**
    *   An `awk` script is used to create separate CSV files for each difficulty level (EASY, MEDIUM, HARD) based on the `merged.csv` file.
    *   Three Hive tables are created (`dsa_easy`, `dsa_medium`, `dsa_hard`) with partitioning by difficulty. (See Figure 5)
    *   Data is loaded into the partitioned tables from the respective difficulty-level CSV files. (See Figure 6)

5.  ✨ **Spark Analytics (Scala):**
    *   A `HiveContext` is initialized in Spark to access the Hive data.
    *   Spark SQL queries are used to perform aggregate analysis on the partitioned Hive tables, calculating average frequency and acceptance rates for each company, grouped by difficulty. (See Figure 7)
    *   The results are saved as CSV files.

6.  💾 **HDFS Storage:**
    *   The CSV results from Spark are transferred to HDFS using the `hdfs dfs -put` command. (See Figure 8)

7.  🐍📊 **Spark Python3 Visualization:**
    *   A Python script uses `findspark` to initialize Spark.
    *   The script reads the CSV data from HDFS using Spark.
    *   The Spark DataFrame is converted to a Pandas DataFrame for easier manipulation and plotting.
    *   Data cleaning (handling missing values, converting data types) is performed.
    *   The data is sorted and chunked to create separate plots for each subset of the companies.
    *   Matplotlib is used to generate visualizations (line plots) showing company names on the x-axis and average frequency/acceptance rates on the y-axis. (See Figure 9)

## 📁 Code Structure

*   📜 **MySQL Scripts:** SQL scripts for table creation and data loading (See Figure 3).
*   ⌨️ **Sqoop Command:** Sqoop command for transferring data to Hive (See Figure 4).
*   📜 **Hive Scripts:** HiveQL for table creation, partitioning, and data loading (See Figure 5 & 6).
*   ✨ **Spark Scala Code:** Spark Scala code with HiveContext and Spark SQL queries for data processing and aggregation (See Figure 7).
*   🐍 **Spark Python Code:** Python script using Spark and Matplotlib for data reading from HDFS, analysis, and visualization (See Figure 9).

## 🎓 Key Learning Outcomes

*   ✅ **Data Pipeline Design:** Understanding and implementing a complete data pipeline.
*   ✅ **Data Ingestion:** Working with MySQL and Sqoop.
*   ✅ **Data Warehousing:** Using Hive and partitioning.
*   ✅ **Big Data Processing:** Leveraging Spark for data aggregation and analysis.
*   ✅ **Data Visualization:** Creating insightful visualizations using Matplotlib.
*   ✅ **HDFS Interaction:** Using HDFS for data storage.
*   ✅ **Scalability and Performance:** Demonstrating the ability to process and analyze large datasets.

## 🚀 How to Run

1.  🔧 **Setup:**
    *   Ensure you have a Hadoop cluster (or a single-node Hadoop setup like Cloudera) with Hive, Spark, and MySQL installed and running.
    *   You will also need to have the necessary Java JARs installed for Spark.
2.  📄 **Data Preparation:**
    *   Create `merged.csv` file, or update the path for your `Leetcode.csv` file.
    *   Modify the paths in the Python script to match your HDFS environment.
3.  ▶️ **Execution:**
    *   Run the MySQL scripts to create the database and table, and load data.
    *   Execute the Sqoop command to import the data into Hive.
    *   Run the Hive scripts to create the partitioned tables.
    *   Run the Spark Scala code within `spark-shell`. Ensure you include the required JARs.
    *   Execute the Python script using `spark-submit` or within a suitable Python environment configured to use Spark. Make sure `findspark` is initialized.
4.  🖼️ **Visualization:** The Python script will generate plots which are usually displayed in the console or can be saved to files.

## 💡 Future Improvements

*   ➡️ **Data Validation:** Implement data validation steps to ensure data quality.
*   ➡️ **Automated Pipeline:** Automate the entire pipeline using tools like [Apache Airflow](https://airflow.apache.org/) or [Luigi](https://github.com/spotify/luigi).
*   ➡️ **More Advanced Analysis:** Perform more sophisticated statistical analysis and machine learning tasks.
*   ➡️ **Interactive Dashboards:** Create interactive dashboards using tools like [Apache Superset](https://superset.apache.org/) or [Tableau](https://www.tableau.com/).
*   ➡️ **Error Handling:** Implement robust error handling and logging.
*   ➡️ **ETL Framework:** Explore more complex ETL frameworks like [Apache NiFi](https://nifi.apache.org/).

## 📬 Contact

Ravi Kumar Rangu
*   ✉️ Email: [ravikumarrangu2@gmail.com](mailto:ravikumarrangu2@gmail.com)
*   💼 LinkedIn: [linkedin.com/in/ravikumar-rangu](https://www.linkedin.com/in/ravikumar-rangu/)
