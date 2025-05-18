# 🚀 LeetCode Data Pipeline: End to End Big Data Engineering with MySQL, Hive, Spark, PySpark Visual Analytics 📊

## 🌟 Project Overview

This project demonstrates a comprehensive end-to-end data engineering pipeline, transforming data from a LeetCode dataset (problem data including company, difficulty, title, frequency, acceptance rate, link, and topics) into insightful visualizations. The pipeline starts with a raw CSV file and processes it through various big data technologies, showcasing proficiency in data ingestion, storage, transformation, and analysis.

## 🚀 How to Setup

1.  **Prerequisites & Setup:**
    *   A working Hadoop cluster (e.g., Cloudera, Hortonworks, or a custom setup) with HDFS, YARN, Hive, Spark, and MySQL installed and running.
    *   Sqoop installed and configured.
    *   Client tools for MySQL, Hive, Spark (`spark-shell`, `pyspark` or `spark-submit`).
    *   WinSCP or a similar tool for file transfers.
    *   Python 3 with `pandas`, `matplotlib`, and `findspark` installed.
    *   Ensure the necessary Spark CSV support JARs are available if using older Spark versions for `format("com.databricks.spark.csv")`.

## 🗺️ Project Steps & Implementation Details

This section outlines each step of the pipeline, including the specific commands and code used for execution.

### 1. 📥 Data Acquisition & Preparation
The project begins with a CSV file named `Leetcode.csv` (referred to as `merged.csv` in the steps below) containing LeetCode problem data.

*   **Action:** Transfer `leetcode.csv` (as `merged.csv`) to `/home/cloudera` on your VM.
    *(See [Figure 2: Dataset Transfer to VM](Executions/fig-2.png) for a conceptual representation of this step if you have such a figure).*

### 2. ➡️🗄️ Data Ingestion into MySQL
The raw data is first loaded into a MySQL relational database.

*   **Action:** Create a MySQL database (e.g., `ravi`) and the `dsa` table.

    ```sql
    -- Connect to MySQL
    -- mysql -u root -p
    -- Example: Create database if it doesn't exist
    -- CREATE DATABASE IF NOT EXISTS ravi;
    -- USE ravi;

    mysql> Create table dsa(
        company_name varchar(250),
        difficulty varchar(100),
        title varchar(250),
        frequency float,
        Acc_rate decimal(38,16),
        link varchar(255),
        topics varchar(255)
    );
    ```

*   **Action:** Load data from `merged.csv` into the `dsa` table.

    ```sql
    mysql> LOAD DATA INFILE '/home/cloudera/merged.csv'
    INTO TABLE dsa
    FIELDS TERMINATED BY ','
    ENCLOSED BY '"'
    LINES TERMINATED BY '\n'
    IGNORE 1 LINES; -- Add this if your CSV has a header row
    ```
    *(See [Figure 3: Creating MySQL table and Loading Data](Executions/fig-3.png) for a conceptual representation).*

### 3. ➡️📦 Data Transfer to Hive using Sqoop
Sqoop is used to efficiently import data from the MySQL table into a Hive table.

*   **Action:** Execute the Sqoop import command.

    ```bash
    $ sqoop import \
        --connect jdbc:mysql://localhost/ravi \
        --username root \
        --password cloudera \
        --table dsa \
        --hive-import \
        --create-hive-table \
        --hive-table ravi.dsa \
        --delete-target-dir \
        --direct -m 1
    ```
    *(See [Figure 4: Sqoop Import to Hive](Executions/fig-4.png) for a conceptual representation).*

### 4. 🧩 Data Partitioning in Hive
To optimize queries, the data in Hive is partitioned by `difficulty`. First, the original `merged.csv` is split by difficulty using `awk`.

*   **Action:** Split `merged.csv` into `EASY.csv`, `MEDIUM.csv`, `HARD.csv` using `awk`. (Ensure `merged.csv` is in the current directory where you run `awk`, or provide the full path).

    ```bash
    $ awk -F',' '
    BEGIN {
        OFS = ","  # Output Field Separator
    }
    NR > 1 { # Skip header row if present
        difficulty = $2;  # Difficulty is in column 2
        # Sanitize difficulty string for use as filename (e.g., remove spaces, convert to uppercase)
        gsub(/ /, "_", difficulty);
        sub(/"/, "", difficulty); # Remove leading quote if any
        sub(/"/, "", difficulty); # Remove trailing quote if any
        file = toupper(difficulty) ".csv";
        print >> file;
    }' /home/cloudera/merged.csv
    ```

*   **Action:** Create Hive tables partitioned by difficulty (e.g., within a `parts1` database).

    ```hiveql
    -- Assuming 'parts1' database, create if not exists
    -- CREATE DATABASE IF NOT EXISTS parts1;
    -- USE parts1;

    Hive> Create table parts1.dsa_easy(
        company_name varchar(250),
        title varchar(250),
        frequency float,
        Acc_rate decimal(38,16),
        link varchar(255),
        topics varchar(255)
    )
    PARTITIONED BY (difficulty string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    STORED AS TEXTFILE;

    Hive> Create table parts1.dsa_medium(
        company_name varchar(250),
        title varchar(250),
        frequency float,
        Acc_rate decimal(38,16),
        link varchar(255),
        topics varchar(255)
    )
    PARTITIONED BY (difficulty string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    STORED AS TEXTFILE;

    Hive> Create table parts1.dsa_hard(
        company_name varchar(250),
        title varchar(250),
        frequency float,
        Acc_rate decimal(38,16),
        link varchar(255),
        topics varchar(255)
    )
    PARTITIONED BY (difficulty string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    STORED AS TEXTFILE;
    ```
    *(See [Figure 5: AWK processing and Hive Partitioned Table Creation](Executions/fig-5.png) for a conceptual representation).*

*   **Action:** Load data from the split CSV files into the corresponding Hive partitions.

    ```hiveql
    Hive> LOAD DATA LOCAL INPATH '/home/cloudera/EASY.csv' INTO TABLE parts1.dsa_easy PARTITION (difficulty="EASY");

    Hive> LOAD DATA LOCAL INPATH '/home/cloudera/MEDIUM.csv' INTO TABLE parts1.dsa_medium PARTITION (difficulty="MEDIUM");

    Hive> LOAD DATA LOCAL INPATH '/home/cloudera/HARD.csv' INTO TABLE parts1.dsa_hard PARTITION (difficulty="HARD");
    ```
    *(See [Figure 6: Loading Data into Hive Partitions](Executions/fig-6.png) for a conceptual representation).*

### 5. ✨ Spark Analytics (Scala)
Apache Spark (Scala API) is used to perform aggregate analysis on the Hive tables.

*   **Action:** Launch `spark-shell` with necessary CSV support JARs (if direct CSV writing is used later, otherwise HiveContext handles Hive tables natively).

    ```bash
    # Check for JARs existence if needed, or ensure they are in Spark's classpath
    # $ ls spark-csv_2.10-1.5.0.jar univocity-parsers-1.5.1.jar commons-csv-1.1.jar
    $ spark-shell --jars path/to/spark-csv_2.10-1.5.0.jar,path/to/univocity-parsers-1.5.1.jar,path/to/commons-csv-1.1.jar
    ```

*   **Action:** In the Spark Scala shell, initialize `HiveContext` and run SQL queries.

    ```scala
    // Step 1: Use HiveContext if not already available
    Scala> val sqlContext = new org.apache.spark.sql.hive.HiveContext(sc)

    // Import the Hive database if not default
    Scala> sqlContext.sql("USE parts1")

    // Step 2: Define and Execute Spark SQL Queries
    // Note: In your original query, avg(link) was used for Avg_acceptancerate.
    // This seems like a typo. Assuming you meant avg(Acc_rate) for acceptancerate
    // and avg(frequency) for frequency. I've corrected it below.
    // Please verify if 'acc_rate' or 'frequency' should be used for 'Avg_frequency' alias.
    // For now, I'll assume 'acc_rate' for Avg_frequency and 'frequency' for a new 'Avg_actual_frequency'.
    // Or if 'Avg_frequency' from your text means 'the average of the frequency column':
    val resultDF1 = sqlContext.sql("SELECT company_name, AVG(frequency) as Avg_problem_frequency, AVG(Acc_rate) as Avg_acceptance_rate FROM dsa_easy GROUP BY company_name ORDER BY Avg_problem_frequency DESC, Avg_acceptance_rate DESC")

    val resultDF2 = sqlContext.sql("SELECT company_name, AVG(frequency) as Avg_problem_frequency, AVG(Acc_rate) as Avg_acceptance_rate FROM dsa_medium GROUP BY company_name ORDER BY Avg_problem_frequency DESC, Avg_acceptance_rate DESC")

    val resultDF3 = sqlContext.sql("SELECT company_name, AVG(frequency) as Avg_problem_frequency, AVG(Acc_rate) as Avg_acceptance_rate FROM dsa_hard GROUP BY company_name ORDER BY Avg_problem_frequency DESC, Avg_acceptance_rate DESC")

    // Step 3: Save Results to CSV on the local filesystem
    (resultDF1.coalesce(1)
      .write
      .format("com.databricks.spark.csv") // or "csv" for newer Spark versions
      .option("header", "true")
      .save("file:///home/cloudera/company_analysis_dsa_easy"))

    (resultDF2.coalesce(1)
      .write
      .format("com.databricks.spark.csv") // or "csv" for newer Spark versions
      .option("header", "true")
      .save("file:///home/cloudera/company_analysis_dsa_medium"))

    (resultDF3.coalesce(1)
      .write
      .format("com.databricks.spark.csv") // or "csv" for newer Spark versions
      .option("header", "true")
      .save("file:///home/cloudera/company_analysis_dsa_hard"))
    ```
    *(See [Figure 7: Spark Analytics using Scala and Spark SQL](Executions/fig-7.png) for a conceptual representation).*

### 6. 💾 HDFS Storage
The aggregated results from Spark (saved as CSVs locally) are then moved to HDFS for persistent distributed storage.

*   **Action:** Use HDFS commands to put the result files into HDFS.

    ```bash
    $ hdfs dfs -put /home/cloudera/company_analysis_dsa_easy/* /user/cloudera/company_analysis_dsa_easy_results
    $ hdfs dfs -put /home/cloudera/company_analysis_dsa_medium/* /user/cloudera/company_analysis_dsa_medium_results
    $ hdfs dfs -put /home/cloudera/company_analysis_dsa_hard/* /user/cloudera/company_analysis_dsa_hard_results
    ```
    *(Note: It's good practice to put them into a subdirectory in HDFS, e.g., `company_analysis_dsa_easy_results` rather than directly into `/user/cloudera` if those are directories produced by Spark save).*
    *(See [Figure 8: Saving Spark Results and Moving to HDFS](Executions/fig-8.png) for a conceptual representation).*

### 7. 🐍📊 Spark Python3 Visualization
Finally, PySpark is used to read the processed data from HDFS and Matplotlib is used to create visualizations.

*   **Action:** Run the PySpark script for visualization.

    ```python
    import findspark
    findspark.init() # Ensure Spark is findable

    from pyspark.sql import SparkSession
    import pandas as pd
    import matplotlib.pyplot as plt

    # Spark Session and HDFS config
    hdfs_uri = "hdfs://192.168.181.132:8020"  # Replace with your HDFS NameNode URI

    # Define paths to the result directories in HDFS
    csv_path_easy = f"{hdfs_uri}/user/cloudera/company_analysis_dsa_easy_results"
    csv_path_medium = f"{hdfs_uri}/user/cloudera/company_analysis_dsa_medium_results"
    csv_path_hard = f"{hdfs_uri}/user/cloudera/company_analysis_dsa_hard_results"

    spark = SparkSession.builder \
        .appName("Read HDFS and Analyze") \
        .config("spark.hadoop.fs.defaultFS", hdfs_uri) \
        .config("spark.hadoop.dfs.client.use.datanode.hostname", "true") \
        .getOrCreate()

    def process_and_plot(csv_path, difficulty_label, spark_session):
        print(f"Processing data for: {difficulty_label} from {csv_path}")
        # Read CSV from HDFS using Spark
        try:
            df_spark = spark_session.read.option("header", "true").csv(csv_path)
            df_spark.printSchema()
        except Exception as e:
            print(f"Error reading {csv_path}: {e}")
            return

        # Convert to Pandas DataFrame for analysis
        df = df_spark.toPandas()

        # Strip any whitespace from column names
        df.columns = [col.strip() for col in df.columns]

        # Ensure correct column names based on Spark SQL output
        # Using 'Avg_problem_frequency' and 'Avg_acceptance_rate' as defined in Spark Scala step
        freq_col = 'Avg_problem_frequency'
        acc_rate_col = 'Avg_acceptance_rate'

        # Convert numeric columns to float
        df[freq_col] = pd.to_numeric(df[freq_col], errors='coerce')
        df[acc_rate_col] = pd.to_numeric(df[acc_rate_col], errors='coerce')

        # Drop rows with NaNs in important columns
        df.dropna(subset=[freq_col, acc_rate_col], inplace=True)

        if df.empty:
            print(f"No data to plot for {difficulty_label} after cleaning.")
            return

        # Sort by Avg_frequency (using the corrected column name)
        df_sorted = df.sort_values(by=freq_col, ascending=False) # Typically want highest frequency first

        # Chunk into groups of 55 for plotting
        chunks = [df_sorted[i:i+55] for i in range(0, len(df_sorted), 55)]

        # Plot Avg Frequency
        for idx, chunk in enumerate(chunks):
            plt.figure(figsize=(18, 6))
            plt.bar(chunk['company_name'], chunk[freq_col], color='blue') # Using bar for better readability
            plt.title(f"Avg Problem Frequency ({difficulty_label} - Companies {idx*55+1} to {min((idx+1)*55, len(df_sorted))})")
            plt.xlabel("Company Name")
            plt.ylabel("Avg Problem Frequency")
            plt.xticks(rotation=90)
            plt.tight_layout()
            plt.grid(True, axis='y')
            plt.savefig(f"./{difficulty_label}_avg_frequency_chunk_{idx+1}.png") # Save plot
            plt.show()

        # Plot Avg Acceptance Rate
        for idx, chunk in enumerate(chunks):
            plt.figure(figsize=(18, 6))
            plt.bar(chunk['company_name'], 100 * chunk[acc_rate_col], color='red') # Using bar
            plt.title(f"Avg Acceptance Rate ({difficulty_label} - Companies {idx*55+1} to {min((idx+1)*55, len(df_sorted))})")
            plt.xlabel("Company Name")
            plt.ylabel("Avg Acceptance Rate (%)")
            plt.xticks(rotation=90)
            plt.tight_layout()
            plt.grid(True, axis='y')
            plt.savefig(f"./{difficulty_label}_avg_acceptance_rate_chunk_{idx+1}.png") # Save plot
            plt.show()

    # Process and plot for each difficulty
    process_and_plot(csv_path_easy, "Easy", spark)
    process_and_plot(csv_path_medium, "Medium", spark)
    process_and_plot(csv_path_hard, "Hard", spark)

    # Stop Spark session
    spark.stop()
    ```
    *(See [Figure 9: Spark Python3 Visual Analytics](Executions/fig-9.png) for a conceptual representation).*

## 📁 Code Structure

This repository contains:
*   This `README.md` file.
*   An `Executions/` directory (conceptual) containing screenshots or diagrams of the steps.
*   Potentially, separate script files for SQL, HiveQL, Scala, and Python if not run interactively.

## 📬 Contact

Ravi Kumar Rangu
*   ✉️ Email: [ravikumarrangu2@gmail.com](mailto:ravikumarrangu2@gmail.com)
*   💼 LinkedIn: [linkedin.com/in/ravikumar-rangu](https://www.linkedin.com/in/ravikumar-rangu/)