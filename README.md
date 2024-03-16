
# Exploring Arrhythmia Detection Through ECG Data Analysis: A Comprehensive Study

# Introduction
Arrhythmias are cardiac rhythm disorders that pose significant health risks if left undetected and untreated. In this blog post, we embark on a journey to understand arrhythmia detection using electrocardiogram (ECG) data analysis. We’ll explore the process of acquiring, preprocessing, and analyzing ECG data, aiming to uncover insights that can aid in the early diagnosis and management of arrhythmias.

# A large-scale 12-lead electrocardiogram database for arrhythmia study
I utilized the “WFDBRecords” dataset as the cornerstone of my analysis. This extensive repository comprises 12-lead electrocardiogram (ECG) recordings meticulously collected for arrhythmia studies. Leveraging this dataset provided a comprehensive array of ECG signals, enabling detailed exploration and analysis of various cardiac rhythms and abnormalities. By leveraging WFDBRecords, I delved into the intricate details of arrhythmias, uncovering valuable insights into the underlying patterns and characteristics of cardiac irregularities. Utilizing WFDBRecords empowered me to conduct thorough investigations, contributing to advancements in arrhythmia detection and cardiovascular health research.

Data Acquisition and Preprocessing
Our journey begins with the acquisition and preprocessing of ECG data. Using the wfdb library, we retrieve ECG records stored in a PostgreSQL database. These records contain vital information about ECG signals and associated metadata. Let's take a look at the code snippet:

!pip install wfdb
!pip install psycopg2
I installed the wfdb library to handle electrocardiogram (ECG) data efficiently. Additionally, the psycopg2 library facilitated seamless interaction with the PostgreSQL database for data storage and retrieval.

Here’s a code snippet that demonstrates the retrieval and storage of ECG data from local directories into a PostgreSQL database:
import os
import wfdb
import psycopg2
from psycopg2 import sql
import json
import numpy as np

## Function to fetch data for one user
def fetch_user_data(user_id, wfdb_records_dir):
    try:
        # Construct the paths to .hea and .mat files
        header_file = os.path.join(wfdb_records_dir, user_id + '.hea')
        data_file = os.path.join(wfdb_records_dir, user_id + '.mat')
        
        # Check if both .hea and .mat files exist
        if os.path.exists(header_file) and os.path.exists(data_file):
            # Load ECG data and metadata
            record = wfdb.rdrecord(os.path.join(wfdb_records_dir, user_id))
            ecg_data = record.p_signal
            metadata = record.comments
            return ecg_data, metadata
        else:
            print(f"Data files not found for user {user_id}")
            return None, None
    except Exception as e:
        print(f"Error fetching data for user {user_id}: {e}")
        return None, None

## Function to replace "NaN" values with None in the nested list
def replace_nan_with_none(data):
    if isinstance(data, list):
        return [replace_nan_with_none(x) for x in data]
    elif isinstance(data, float) and np.isnan(data):
        return None
    else:
        return data

## Function to traverse directories recursively and fetch data
def traverse_directories(wfdb_records_base_dir):
    for root, dirs, files in os.walk(wfdb_records_base_dir):
        for dir in dirs:
            records_dir = os.path.join(root, dir)
            records_file = os.path.join(records_dir, 'RECORDS')
            if os.path.exists(records_file):
                with open(records_file, 'r') as f:
                    for line in f:
                        user_id = line.strip()
                        ecg_data, metadata = fetch_user_data(user_id, records_dir)
                        if ecg_data is not None and metadata is not None:
                            # Replace "NaN" values with None
                            ecg_data_cleaned = replace_nan_with_none(ecg_data.tolist())
                            ecg_data_json = json.dumps(ecg_data_cleaned)
                            
                            if isinstance(metadata, list):
                                metadata_json = json.dumps(metadata)
                            else:
                                metadata_json = metadata
                            cur.execute(sql.SQL("""
                                INSERT INTO WFDBRecords (user_id, ecg_data, metadata)
                                VALUES (%s, %s, %s)
                            """), (user_id, ecg_data_json, metadata_json))
                            conn.commit()
                            print(f"User {user_id} added to the database. Path: {records_dir}")
                        else:
                            print(f"Skipping user {user_id} due to missing data in {records_dir}")

# Directory where WFDB records are stored locally
wfdb_records_base_dir = 'D:/New/Data Science/Assignment_3/a-large-scale-12-lead-electrocardiogram-database-for-arrhythmia-study-1.0.0/WFDBRecords'

# Connect to the PostgreSQL database
try:
    conn = psycopg2.connect(
        dbname="postgres",
        user="postgres",
        password="dost",
        host="localhost",
        port="1234"
    )
    cur = conn.cursor()

    # Create a table to store ECG data and metadata if not exists
    cur.execute("""
        CREATE TABLE IF NOT EXISTS WFDBRecords (
            id SERIAL PRIMARY KEY,
            user_id VARCHAR,
            ecg_data JSONB,
            metadata JSONB
        );
    """)
    conn.commit()

    # Traverse directories recursively and fetch data
    traverse_directories(wfdb_records_base_dir)
except psycopg2.Error as e:
    print("Error connecting to PostgreSQL:", e)
finally:
    # Close the cursor and connection
    if 'cur' in locals():
        cur.close()
    if 'conn' in locals():
        conn.close()
Exploratory Data Analysis (EDA)
import ipywidgets as widgets
from IPython.display import display
import json
import matplotlib.pyplot as plt
from sklearn.decomposition import PCA
import psycopg2
from matplotlib.widgets import Slider
In the initial stages of our analysis, we leverage Python libraries to visualize Electrocardiography (ECG) data, a cornerstone in cardiology diagnostics. Through the integration of powerful visualization tools, we embark on a comprehensive exploration aimed at uncovering intricate patterns and insights within cardiac signals.

# Pre-Processing: Retrieving ECG Data
Before delving into advanced analysis techniques like Principal Component Analysis (PCA), it’s crucial to ensure that our dataset is properly prepared and accessible. In this stage, we focus on fetching the necessary metadata and ECG data for a given file name from our PostgreSQL database. The Python function fetch_data serves as a conduit for this task.


This function acts as a bridge between our Python environment and the database, executing SQL queries to extract the necessary information. Upon successful retrieval, the metadata and ECG data are returned, enabling subsequent analysis. In cases where data is not found or errors occur during retrieval, appropriate error handling ensures robustness and reliability in our data pipeline.

# Visualizing Electrocardiogram (ECG) Data Before Dimensionality Reduction
Before applying dimensionality reduction techniques like PCA (Principal Component Analysis) to ECG data, it’s crucial to visualize the raw data to understand its structure and characteristics. The following Python function, plot_ecg_with_metadata, facilitates this visualization process:


The plot_ecg_with_waveform function generates a visual representation of electrocardiogram (ECG) waveform data. It plots the amplitude values of the ECG signal over time (samples) on a 12x6-inch figure. The title "ECG Plot" and labeled axes ("Sample" for x-axis and "Amplitude" for y-axis) aid in interpretation. Grid lines are included to assist in analyzing signal characteristics. Furthermore, if metadata is available, it is displayed, providing additional context such as patient ID or recording details. Overall, this function offers a clear visualization of ECG waveforms for analysis and interpretation, potentially aiding in medical diagnostics or research.


(ECG) Signals with Attribute Integration
The plot_ecg_with_attribute_combine function visualizes multiple ECG signals alongside their corresponding attributes, such as leads I, II, III, aVR, aVL, aVF, and chest leads V1-V6. Each attribute is represented as a separate line plot on the same figure, facilitating comparison of signal characteristics across different leads. This enables doctors to observe the electrical activity of the heart from various perspectives simultaneously, aiding in the diagnosis of arrhythmias or other cardiac conditions. The legend provides clarity by labeling each signal, while the axes denote time (x-axis) and amplitude (y-axis), enhancing interpretability for medical professionals. Additionally, grid lines assist in analyzing signal patterns and identifying abnormalities.


​
# Applying PCA
Following data visualization, PCA is applied to the ECG data, transforming it into a lower-dimensional space while preserving essential information. The transformed data (ecg_pca) is plotted in a 2D space, with each point representing an ECG record's projection onto the principal components. By examining the PCA-transformed data, clinicians gain insights into the underlying structure and patterns within the ECG dataset.

# Medical Relevance
PCA’s dimensionality reduction facilitates the identification of crucial features and patterns within ECG data, aiding in arrhythmia detection and diagnosis. Doctors can interpret the PCA-transformed data to discern distinct clusters or patterns indicative of various cardiac conditions, enhancing diagnostic accuracy and patient care.


# Gender Count Visualization: 
The plot_gender_distribution function visualizes the distribution of genders in a dataset using a bar chart. It iterates through the provided list of genders, counting the occurrences of each gender (male and female). Then, it plots a bar chart with 'Male' and 'Female' on the x-axis and their corresponding counts on the y-axis. Annotations are added to each bar to display the exact count for better clarity.

Age Distribution Visualization: The plot_age_distribution function illustrates the distribution of ages within a dataset using a histogram. It takes a list of ages as input and plots a histogram with 20 bins to represent the frequency distribution of ages. The x-axis denotes the age range, while the y-axis indicates the frequency (count) of individuals falling within each age bin.



# Conclusion
In this extensive exploration of electrocardiogram (ECG) data analysis, we’ve journeyed through a multifaceted approach to understanding cardiac health assessment. Beginning with data retrieval and visualization, we uncovered the raw signals of cardiac activity using functions like plot_ecg_with_metadata and plot_ecg_with_waveform, gaining crucial insights into ECG attributes. By visualizing specific attributes such as leads I, II, III, and others with plot_ecg_with_attribute_combine, we delved deeper into the intricacies of ECG signals, laying the foundation for advanced analysis. Moreover, demographic analysis provided valuable insights into the distribution of gender and age within the dataset, shedding light on population characteristics. Furthermore, our application of Principal Component Analysis (PCA) for dimensionality reduction, as facilitated by the apply_pca function, offered a condensed representation of the high-dimensional ECG data, preserving essential information while reducing complexity.

This comprehensive exploration holds profound implications for clinical practice, enabling healthcare professionals to derive actionable insights from ECG data. By leveraging visualization techniques and dimensionality reduction methods like PCA, clinicians can uncover patterns and anomalies indicative of cardiac abnormalities or arrhythmias. These insights empower medical practitioners to make informed decisions, leading to more accurate diagnoses and personalized treatment plans tailored to individual patient needs. As technology continues to advance, the integration of data-driven approaches in cardiovascular medicine promises to revolutionize patient care, ultimately contributing to improved health outcomes and enhanced quality of life for individuals worldwide.

