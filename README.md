# Data Engineer Project: Retail Data Pipeline

[Youtube tutorial]([https://www.youtube.com/watch?v=NP08fHker5U](https://www.youtube.com/watch?v=DzxtCxi4YaA))

Thanks to [Marc Lamberti](https://www.youtube.com/@MarcLamberti) videos, I've been following this [Notion](https://robust-dinosaur-2ef.notion.site/PUBLIC-Retail-Project-af398809b643495e851042fa293ffe5b) guide to create a complete project.

*Using Airflow, BigQuery, Google Cloud Storage, dbt, Soda, Metabase and Python*

## Objective
The goal of this project is to create an end-to-end data pipeline from a Kaggle dataset on retail data. This involves modeling the data into fact-dimension tables, implementing data quality steps, utilizing modern data stack technologies (dbt, Soda, and Airflow), and storing the data in the cloud (Google Cloud Platform). Finally, the reporting layer is consumed in Metabase. The project is containerized via Docker and versioned on GitHub.
- Tech Stack Used
- Python
- Docker and Docker-compose
- Soda.io
- Metabase
- Google Cloud Storage
- Google BigQuery
- Airflow (Astronomer version)
- dbt
- GitHub


In the folder ```dags/include/datasets/``` you will find 3 files, the ```online_retail.csv``` is the original one downloaded from Kaggle and the ```country.csv``` was generated using a BigQuery table. It's all the data that is needed for this project.

---------
## To run this project you must

### Install Docker
[Install Docker for your OS](https://docs.docker.com/desktop/)

### Install Astro CLI
[Install Astro CLI for your OS](https://www.astronomer.io/docs/astro/cli/install-cli)

Note: Astro CLI is an alternative to direct Airflow Apache local installation - faster and easier Airflow deployment, and is a Docker wrapper.
The Astro CLI automatically runs Podman containers whenever you run a command that requires them. To run CLI commands in Docker containers, run the following command:

```bash
astro config set container.binary docker -g
```

### Clone the GitHub repo

In your terminal:

Clone the repo
```bash
git clone https://github.com/BrainMcS/ETL-3-Airflow.git
```

Open the folder with your code editor.

### Reinitialize the Airflow project
Open the code editor terminal:
```bash
astro dev init
```
It will ask: ```You are not in an empty directory. Are you sure you want to initialize a project? (y/n)```
Type ```y``` and the project will be reinitialized.


### Build the project
In the code editor terminal, type:

```bash
astro dev start
```
The default Airflow endpoint is http://localhost:8080/
- Default username: admin
- Default password: admin

### Create the GCP project

In your browser go to https://console.cloud.google.com/ and create a project, recomended something like: ```airflow-dbt-soda-pipeline```

Copy your project ID and save it for later. Note that you have to replace any instance of ```helical-glass-448521-n1``` with your GCD project ID!

#### Using the project ID from GCP

Change the following files:
- .env (GCP_PROJECT_ID)
- include\dbt\models\sources\sources.yml (database)
- include\dbt\profiles.yml (project)

#### Create a Bucket on GCP

With the project selected, go to https://console.cloud.google.com/storage/browser and create a Bucket.
Use the name ```<yourname>_online_retail```.
And change the variable ```bucket_name``` value to your bucket name at the ```dags\retail.py``` file.

#### Create an service account for the project

Go to the IAM tab, and create the Service account with the name ```airflow-online-retail```.
Give admin access to GCS and BigQuery, and export the json keys. Rename the file to service_account.json and put inside the folder ```include/gcp/``` (you will have to create this folder).

#### Build a connection in your airflow

In your airflow, at the http://localhost:8080/, login and go to Admin → Connections.
Create a new connection and use this configs:
- id: gcp
- type: Google Cloud
- Keypath Path: `/usr/local/airflow/include/gcp/service_account.json`

Save it.

### Create you SODA account and API Keys

Go to https://www.soda.io/ and click "start a trial" and create an account. Then, login and go to your profile, API Keys and create a new API key.

Copy the soda_cloud code, it will look like this:
```
soda_cloud:
  host: cloud.us.soda.io
  api_key_id: <KEY>
  api_key_secret: <SECRET>
```
Edit the .env file with the respective values.
Warning: If you copy and paste it in ```include\soda\configuration.yml``` file, it will be pushed to you GitHub! I don't recommend doing so for security purposes.

### All set, start the DAG

With your Airflow running, go to http://localhost:8080/ and click on DAGs, and click on the retail DAG.
Then, start the DAG (play button on the upper right side).

It will go step by step, and if everything was followed, you will get a green execution at the end.
Check in your GCP Storage account if the file was uploaded succesfully, in your BigQuery tab if the tables was been built and in your Soda dashboard if everithing is fine.

Then, move to the Metabase and build your own Dashboard. The Metabase service is on the http://localhost:3002/

### Metabase

Go to the http://localhost:3002/ and create your local account.
When the `Add your data` option shows up, choose BigQuery, and enter your details.
It will ask for a Display name, I recomend `BigQuery_DW` or something like that.
The `project ID` is your GCP Project name (mine was the project ID generated by GCP). And finally the `service_account.json` the one that you saved in the `include/gcp/` folder.

Connect the database, and it's all set.
