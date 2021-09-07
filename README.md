# TEAM 9 Fork


# Challenge 2: Approach:

a) Boot up the SQL container

Default user might be SA
```
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>" \
   -p 1433:1433 --name sql1 -h sql1 \
   -d \
   mcr.microsoft.com/mssql/server:2017-latest
```
Test connection 
[tbc]
```
sqlcmd -S localhost:1433 -U SA -P "<YourStrong@Passw0rd>"
```

b) create database
docker exec -it sql1 "bash"

then in the container
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "WelcomeAzure23!"

CREATE DATABASE mydrivingDB

---
or

[tbc - doesn't work]

sqlcmd -S localhost:1433 -U SA -P "<YourStrong@Passw0rd>"

c) Data loading:
uses an open? image called data-load v1

docker network ls

```
docker run --network host -e SQLFQDN=127.0.0.1 -e SQLUSER=sa -e SQLPASS="<YourStrong@Passw0rd>" -e SQLDB=mydrivingDB openhack/data-load:v1
```

or 
```
docker inspect sql1
```
to get the *bridge* network IP Address
```
docker run --network bridge -e SQLFQDN=172.17.0.2 -e SQLUSER=sa -e SQLPASS="<YourStrong@Passw0rd>" -e SQLDB=mydrivingDB openhack/data-load:v1
```


c) build POI container
first edit Dockerfile to give the right connection info




# Containers 2.0 Openhack

<!-- 
Guidelines on README format: https://review.docs.microsoft.com/help/onboard/admin/samples/concepts/readme-template?branch=master

Guidance on onboarding samples to docs.microsoft.com/samples: https://review.docs.microsoft.com/help/onboard/admin/samples/process/onboarding?branch=master

Taxonomies for products and languages: https://review.docs.microsoft.com/new-hope/information-architecture/metadata/taxonomies?branch=master
-->

This repo houses the source code and dockerfiles for the Containers OpenHack event.

The application used for this event is a heavily modified and recreated version of the original [My Driving application](https://github.com/Azure-Samples/MyDriving).

## Contents

| File/folder       | Description                                |
|-------------------|--------------------------------------------|
| `.devcontainer`   | VS Code [development container](https://code.visualstudio.com/docs/remote/containers) with useful utils (Azure CLI, Kubectl, Helm, etc.)   |
| `dockerfiles`     | Dockerfiles for source code                |
| `src`             | Sample source code for POI, Trips, User (Java), UserProfile (Node.JS), and TripViewer                     |
| `.gitignore`      | Define what to ignore at commit time.      |
| `CODE_OF_CONDUCT.md` | Code of conduct.                        |
| `LICENSE`         | The license for the sample.                |

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
