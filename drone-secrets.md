# Secrets Per Repository

Official documentation for storing secrets can be found [here](https://docs.drone.io/secret/repository/). The steps below may also help:

> 1. Click on the desired repository, click on the `Settings` tab, and on the left-hand side choose `Secrets` as shown below:
![image](./pictures/drone-secrets.png) 
> 2. Click `NEW SECRET` and enter the key and value pair.
> 3. In your `drone.yaml` file, you can then access this secret as shown below:
```yaml
  steps:
  - name: build
  image: alpine
  environment:
    YOUR-ENVIRONMENT-VARIABLE:
      from_secret: my-secret-test
```