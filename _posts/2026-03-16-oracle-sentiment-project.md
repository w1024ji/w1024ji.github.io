Yesterday's goal: Let's automize crawling process!
So, yesterday I set up EC2(ubuntu) in AWS and set up a Airflow inside that ubuntu.

I first looked for the Managed Airflow in AWS, but found out it is for big enterprises.
So I detoured to do on my EC2. 

Built my airflow_env inside that ubuntu, and the settings..

ec2 code below: 
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv -y

python3 -m venv airflow_env
source airflow_env/bin/activate

pip install apache-airflow
pip install apache-airflow-providers-amazon

And I wrote DAG to run the crawling daily in the morning.
Of course I already set up the AWS connection in Airflow.

Once it was succeeded, I made the Airflow to run in the background, so it won't stop even I turn off the ubuntu CLI.

nohup airflow standalone > airflow_server.log 2>&1 &


----------


And this is I saw in this morning!
It is working well, the DAG succeeded in 8am in the morning.

![airflow in the morningl](./photos/airflow success.png)
