# ETL-Project-using-Apache-Airflow
This project involves creating an Airflow DAG that automates downloading a file, transforming the content, and saving the result.


#### **Components of the Project:**
- **DAG (Directed Acyclic Graph):** The backbone of Airflow workflow, organizing tasks and their execution order.
- **Tasks:** Individual steps in the ETL pipeline, each represented by a Python function:
  - `download`: Download the log file.
  - `extract`: Extract specific data fields (timestamp and visitorid).
  - `transform`: Capitalize the `visitorid`.
  - `load`: Save the transformed data to a new file (`capitalized.txt`).
  - `check`: Verify the loaded data.

#### **Server Access Log Fields:**
- **timestamp:** TIMESTAMP
- **latitude:** float
- **longitude:** float
- **visitorid:** char(37)
- **accessed_from_mobile:** boolean
- **browser_code:** int

---

### **2. Imports Block**

The first step is to import the necessary modules.

```python
from datetime import timedelta
from airflow.models import DAG
from airflow.operators.python import PythonOperator
from airflow.utils.dates import days_ago
import requests
```

These imports handle Airflow’s task management, HTTP requests (to download the file), and scheduling.

---

### **3. DAG Arguments Block**

Define default arguments for the DAG, like who owns it, when it should start, how many retries are allowed, and the delay between retries.

```python
default_args = {
    'owner': 'Your name',
    'start_date': days_ago(0),
    'email': ['your email'],
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}
```

This ensures that if a task fails, Airflow will retry it after 5 minutes.

---

### **4. DAG Definition Block**

Create the DAG definition, specifying how often the workflow should run.

```python
dag = DAG(
    'ETL_Server_Access_Log_Processing',
    default_args=default_args,
    description='A simple ETL DAG',
    schedule_interval=timedelta(days=1),  # Run daily
)
```

This DAG will run once every day (`schedule_interval=timedelta(days=1)`).

---

### **5. Tasks Definitions**

Each task is created using a `PythonOperator` to call the respective Python function.

#### a) **Download Task:**
This task downloads the file from the remote server and stores it locally.

```python
def download_file():
    url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0250EN-SkillsNetwork/labs/Apache%20Airflow/Build%20a%20DAG%20using%20Airflow/web-server-access-log.txt"
    with requests.get(url, stream=True) as response:
        response.raise_for_status()
        with open(input_file, 'wb') as file:
            for chunk in response.iter_content(chunk_size=8192):
                file.write(chunk)
    print(f"File downloaded successfully: {input_file}")
```

This Python function downloads the file from the given URL and saves it as `web-server-access-log.txt`.

#### b) **Extract Task:**
This task reads the log file and extracts the `timestamp` and `visitorid` fields.

```python
def extract():
    with open(input_file, 'r') as infile, open(extracted_file, 'w') as outfile:
        for line in infile:
            fields = line.split('#')
            if len(fields) >= 4:
                outfile.write(fields[0] + "#" + fields[3] + "\n")
```

It splits each line by the `#` character and extracts the first (`timestamp`) and fourth (`visitorid`) fields.

#### c) **Transform Task:**
This task capitalizes the `visitorid` field.

```python
def transform():
    with open(extracted_file, 'r') as infile, open(transformed_file, 'w') as outfile:
        for line in infile:
            outfile.write(line.upper())
```

The function reads the `extracted-data.txt` file and writes an uppercase version of each `visitorid` to `transformed.txt`.

#### d) **Load Task:**
This task loads the transformed data into a new file.

```python
def load():
    with open(transformed_file, 'r') as infile, open(output_file, 'w') as outfile:
        for line in infile:
            outfile.write(line)
```

This writes the capitalized content to `capitalized.txt`.

#### e) **Check Task:**
This optional task prints out the content of the final `capitalized.txt` file for verification.

```python
def check():
    with open(output_file, 'r') as infile:
        for line in infile:
            print(line)
```

---

### **6. Task Pipeline**

The tasks are arranged in a pipeline, ensuring that they run sequentially.

```python
download = PythonOperator(
    task_id='download',
    python_callable=download_file,
    dag=dag,
)

execute_extract = PythonOperator(
    task_id='extract',
    python_callable=extract,
    dag=dag,
)

execute_transform = PythonOperator(
    task_id='transform',
    python_callable=transform,
    dag=dag,
)

execute_load = PythonOperator(
    task_id='load',
    python_callable=load,
    dag=dag,
)

execute_check = PythonOperator(
    task_id='check',
    python_callable=check,
    dag=dag,
)

download >> execute_extract >> execute_transform >> execute_load >> execute_check
```

Here, each task depends on the completion of the previous one.

---

### **7. Submitting the DAG**

After saving the code in a Python file (`ETL_Server_Access_Log_Processing.py`), place it in the Airflow DAGs directory:

```bash
cp ETL_Server_Access_Log_Processing.py $AIRFLOW_HOME/dags
```

Check if the DAG was submitted using:

```bash
airflow dags list | grep etl-server-logs-dag
```

If there are errors, you can troubleshoot using:

```bash
airflow dags list-import-errors
```

---

### **8. Running and Monitoring the DAG**

Once the DAG is submitted, you can trigger and monitor it using the Airflow UI or command-line interface.

