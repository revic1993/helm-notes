# Basics

## Concepts
- Chart - a helm package
- Repository - place where charts can be collected and shared
- Release - is an instance of a chart running on the cluster
- ***Helm*** installs ***charts*** into ***Kubernetes***, creating a new ***release*** for each installation. And to find new charts, you can search Helm chart ***repositories***

## Finding charts
- ```helm search hub``` searches **Helm Hub**.
- ```helm search repo``` searches in local repository for installation that has been added with ```helm repo add```.

