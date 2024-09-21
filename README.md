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

### 1. `def download_file():`
This line defines a Python function named `download_file()`. In this case, the task is downloading a file from the internet.

---

### 2. `url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0250EN-SkillsNetwork/labs/Apache%20Airflow/Build%20a%20DAG%20using%20Airflow/web-server-access-log.txt"`
Here, we’re storing the link (URL) of the file that we want to download into the variable `url`. This URL points to a file (a web server access log) that’s stored online.

---

### 3. `with requests.get(url, stream=True) as response:`
This line sends a request to the website asking for the file located at the `url`. The `requests.get()` function does the job of contacting the server to get the file.

- **`stream=True`:** This tells Python to download the file in small pieces (chunks) rather than all at once. This is useful for large files.
- **`as response`:** This assigns the result of the `requests.get()` call to a variable called `response`. The `with` keyword ensures that everything is cleaned up properly when done, even if there's an error.

---

### 4. `response.raise_for_status()`
This line checks if there was any problem with the download request. If the server couldn’t find the file or if there was some other issue, this command will raise an error. It’s like saying, “If something went wrong, stop here and raise an alert!”

---

### 5. `with open(input_file, 'wb') as file:`
This line opens a new file on your computer for writing. 

- **`input_file`:** This is the name of the file on your computer where the downloaded content will be saved. You’d define this earlier in the script.
- **`'wb'`:** This means "write binary," telling Python to write the file in binary mode. Binary mode is used because we’re downloading a raw file, which might contain data other than just text (like images or special characters).
- **`as file`:** This assigns the opened file to the variable `file`, so that the program can write into it.

---

### 6. `for chunk in response.iter_content(chunk_size=8192):`
This line begins a loop to process the file in chunks (pieces). 

- **`response.iter_content()`:** This function allows us to download the file piece by piece instead of all at once. Each chunk is a small portion of the file.
- **`chunk_size=8192`:** This specifies the size of each chunk (8 KB or 8192 bytes). So, the file will be downloaded in 8 KB pieces.
- **`for chunk in ...`:** This loop runs once for each chunk and writes it to the file.

---

### 7. `file.write(chunk)`
This line writes each chunk of data into the file on your computer. The loop continues until all chunks are downloaded and written, so the full file is saved piece by piece.

---

### 8. `print(f"File downloaded successfully: {input_file}")`
After the file is fully downloaded, this line prints a message to confirm that everything worked. It includes the name of the file where the content was saved (`input_file`).

---

### Final Overview:
- **Define the function.**
- **Specify the file URL.**
- **Request the file from the server.**
- **Check if the request was successful.**
- **Open a local file for writing.**
- **Download and save the file chunk by chunk.**
- **Print a success message.**

This process ensures you can download a file from the web and store it locally on your computer in a safe and efficient manner.

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

