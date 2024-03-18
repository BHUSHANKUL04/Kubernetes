# Airflow on Kubernetes clusters
1. kubectl create namespace airflow
   kubectl config set-context --current --namespace=airflow
2. postgres-configmap.yaml
3. kubectl apply -f postgres-configmap.yaml -n airflow
4. postgres-storage.yaml
5. kubectl apply -f postgres-storage.yaml -n airflow
   --checking
   kubectl get pv -n airflow 
   kubectl get pvc -n airflow 
6. postgres-deployment.yaml 
7. kubectl apply -f postgres-deployment.yaml -n airflow
8. postgres-service.yaml
9. kubectl apply -f postgres-service.yaml -n airflow
10. Open postgres container and install python inside postgress container
    kubectl exec --stdin --tty pod/postgres-778fc74489-27w9d -- /bin/bash
	apt-get update
	apt-get install Python3.11
	AIRFLOW_VERSION=2.8.3
        PYTHON_VERSION="$(python3 --version | cut -d " " -f 2 | cut -d "." -f 1-2)" 
	CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
	apt install python3-pip
	apt install libpq-dev python3-dev
        apt install vim
	pip3 install psycopg2 --break-system-packages
        pip3 install psycopg2-binary --break-system-packages
	#pip3 install "apache-airflow[postgres]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}" --break-system-packages	
	
	pip3 install "apache-airflow[postgres]==${AIRFLOW_VERSION}" --break-system-packages
	airflow db init #Initialize the db

11. pv-dags.yaml
12. kubectl apply -f pv-dags.yaml -n airflow
13. pv-logs.yaml
14. kubectl apply -f pv-logs.yaml -n airflow
15. check status of pvc
    kubectl get pv
16. airflow-storageclass.yaml - Not done yet
17. airflow-configmap.yaml 
18. kubectl apply -f airflow-configmap.yaml -n airflow 
19. describe config map
    kubectl describe configmap airflow-config 
20. scheduler-serviceaccount.yaml 
    kubectl apply -f scheduler-serviceaccount.yaml -n airflow 
21. pod-launcher-role.yaml 
    kubectl apply -f pod-launcher-role.yaml -n airflow
22. pod-launcher-rolebinding.yaml    
    kubectl apply -f pod-launcher-rolebinding.yaml -n airflow 
23. airflow-deployment.yaml
    kubectl apply -f airflow-deployment.yaml -n airflow
24. check status of new containers
    kubectl get pods -n airflow 
25. airflow-service.yaml
26. kubectl apply -f airflow-service.yaml -n airflow
27. status
    kubectl describe service webserver-svc -n airflow 
28. pip install 'apache-airflow[cncf.kubernetes]'  --break-system-packages
29. pip install apache-airflow-providers-cncf-kubernetes --break-system-packages
30. On terminal
    airflow users create --username admin --password admin --firstname --lastname --role Admin --email kulkarni.bhushan85@gmail.com
31. Add below sql connection for postgress in airflow.cfg inside pod
    postgresql+psycopg2://airflow_user:airflow_pass123@postgres:5432/airflow_db

32. Add KubernetesExecutor as executor in airflow.cfg
33. Add below in airflow.cfg at the end
    [kubernetes]
    namespace = airflow     
34. 
    #Connecting to postgres on container
    PGPASSWORD=test123 psql -h postgres -U admin postgresdb
34. Create below in postgres DB

CREATE DATABASE airflow_db;
CREATE USER airflow_user WITH PASSWORD 'airflow_pass';
GRANT ALL PRIVILEGES ON DATABASE airflow_db TO airflow_user;
-- PostgreSQL 15 requires additional privileges:
USE airflow_db;
GRANT ALL ON SCHEMA public TO airflow_user;
GRANT CREATE ON SCHEMA public TO airflow_user;
ALTER DATABASE airflow_db OWNER TO airflow_user;

35. Connect to postgress from pod
    PGPASSWORD=airflow_pass psql -h postgres -U airflow_user airflow_db
    select * from jobs;
36. localhost:30961
37. additional command 
   kubectl exec -c scheduler airflow-59f45d7656-5bllv --namespace airflow -- /bin/bash -c "airflow config get-value  kubernetes  namespace"
