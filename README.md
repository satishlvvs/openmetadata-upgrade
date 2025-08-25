# openmetadata-upgrade
open metadata update script
# version : 1.4.0 download
# Document prepared on 2024-05-30

# take backups from old version like 1.3.0

pg_dump -h localhost -U postgres -p 5444 -d airflow_db >airflow_db_v1.3.0_20240530.sql
pg_dump -h localhost -U postgres -p 5444 -d openmetadata_db  > openmetadata_db_v1.3.0_20240530.sql


version : 1.4.0 download
mkdir 140
wget https://github.com/open-metadata/OpenMetadata/releases/download/1.4.1-release/docker-compose-postgres.yml

mv docker-compose-postgres.yml docker-compose-postgres-1.4.0.yml

# note change the postgresql port number and build the docker

docker compose -f docker-compose-postgres-1.4.0.yml up -d

verify the application working or not

http://13.201.75.192:8585
http://13.201.75.192:8080

# once application working then stop docker 1.4.0 

docker compose -f docker-compose-postgres-1.4.0.yml stop

# Docker run only postgresql container for version 1.4.0

docker start openmetadata_postgresql
docker ps

# Connect to Postgre and create new database for airflow and openmetadata with new 
psql -h localhost -U postgres -p 5444

psql  -h localhost -U postgres -p 5444 -c "create database airflow_db_n with owner airflow_user"
psql  -h localhost -U postgres -p 5444 -c "create database openmetadata_db_n with owner openmetadata_user"

# restored data from old version backups

psql  -h localhost -U postgres -p 5444 -d airflow_db_n -f /opt/open-metadata/airflow_db_v1.3.0_20240530.sql 1>air_sucess.log 2>air_error.log
psql  -h localhost -U postgres -p 5444 -d openmetadata_db_n -f /opt/open-metadata/openmetadata_db_v1.3.0_20240530.sql 1>open_sucess.log 2>open_error.log

# alter database name exising to old and new to existing name

psql  -h localhost -U postgres -p 5444 -c "alter database airflow_db rename to airflow_db_old"
psql  -h localhost -U postgres -p 5444 -c "alter database openmetadata_db rename to openmetadata_db_old"

psql  -h localhost -U postgres -p 5444 -c "alter database airflow_db_n rename to airflow_db"
psql  -h localhost -U postgres -p 5444 -c "alter database openmetadata_db_n rename to openmetadata_db"

# after completed the database creation , restore backup and rename datbases. stop postgresql docker services.

docker stop openmetadata_postgresql
docker ps

# start completed openmeda data new version docker services.

docker compose -f docker-compose-postgres-1.3.0.yml start
docker ps

# verify the application  

http://13.201.75.192:8585
http://13.201.75.192:8080

# end of the document for upgrade open-metadata version 1.3.0 to 1.4.0.
