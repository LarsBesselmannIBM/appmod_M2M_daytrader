# Mono2Micro Sample Applications

Please read the documentation at https://www.ibm.com/docs/mono2micro to learn more about Mono2Micro and the various tasks of application data collecting, running the analysis, viewing the analysis results, and generating starter micrservices java code.

To get you started, here are some examples of monolith applications, data and analysis files, and microservices code

## DayTrader - Application Data, Analysis, Source

Monolith source code: ```./daytrader/monolith```

Monolith application data: ```./daytrader/application-data/```

Mono2Micro analysis (initial recommendations): ```./daytrader/mono2micro-analysis```

Mono2Micro analysis (further customized by hand): ```./daytrader/mono2micro-analysis-custom```


## DayTrader - Refactored to microservices with Mono2Micro

Refer to the Mono2Micro documentation for detailed guidance on how to further develop the initial partitions recommended by the AI analysis to customize the final partitioning of the application (see above), refactor the application's server config files, build config files, and JEE deployment descriptor files, and containerize the partitions to run as these microservices in Docker.

Change directory to the refactored microservices source code:
```
cd ./daytrader/microservices/
```

First run the DB2 container to set up the database for the microservices:

```
# The script creates a Docker network and runs DB2 in the network
chmod +x run_db2_container.sh 
./run_db2_container.sh
```

This can take a few minutes to complete as it creates all required database tables and schema. If the DB2 setup fails, the script times out in 10 minutes.

If needing to start a fresh DB2, stop and remove the running container with the docker commands below, and then run the above script again:
```
docker stop <DB2 container ID>
docker container rm db2
```

If the DB2 container starts up successfully, then run the containers for all the partitions of the application:

```
docker-compose up -d
```

To stop the application containers and do a full clean rebuild:

```
docker-compose down
docker-compose up -d --build && docker system prune -f
```

Monitor the running microservices and wait till all containers are running their daytrader application in WebSphere Liberty, and stabilize:

```
docker-compose logs -f
```

If all the containers have started successfully, the DayTrader application can be opened at `http://localhost:9080/daytrader/`

Be sure to go into the 'Configuration' tab from the application's home page and choose the option to populate the database first (to change the defaults for max # of users, quotes, etc, modify ```./daytrader7-web/daytrader-ee7-web/src/main/webapp/properties/daytrader.properties``` and rebuild the application containers per instructions above).

If new microservices code is generated with Mono2Micro for this application, edit and run the following script, pointing its environment variables to where the base directory for the generated code is, as well as the monolith source directory name used when running the mono2micro-cardinal tool. It will copy over the newly generated code and replace the existing files. After that, rebuild the application containers per instructions above.
```
./copy_generated_code.sh
```

To modify the level of logging and tracing in the Mono2Mirco generated code and see more or less information about what's happening in the code flow within and between microservices, change the ```DEFAULT_LOG_LEVEL``` variable in the each partition's ```com.ibm.cardinal.util.CardinalLogger``` Java source file located in the ```cardinal-utils``` module. 


