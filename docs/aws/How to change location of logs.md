# How to change location of logs

- update values.yaml

    ```yaml
    api:
      logs: 
        region: <REGION>
        bucket: <BUCKET>
    ```
- upgrade helm chart

    ```sh
    helm upgrade cdefense charts/cdefense -f values.yaml --debug    
    ```

## In case of private bucket

- Edit the scan-server-secrets.yaml file

  ```yaml
    AWS_SCAN_S3_ACCESS_KEY: <AWS_SCAN_S3_ACCESS_KEY>
    AWS_SCAN_S3_SECRET_KEY: <AWS_SCAN_S3_SECRET_KEY>
  ```

  ```sh
  kubectl apply -f scan-server-secrets.yaml
  ```

- or update secrets on cluster

    - encode values as base64 strings

    ```sh
    AWS_SCAN_S3_ACCESS_KEY=<AWS_ACCESS_KEY>
    BASE64_AWS_SCAN_S3_ACCESS_KEY=$(echo $AWS_SCAN_S3_ACCESS_KEY | base64)
    ```

    ```sh
    AWS_SCAN_S3_SECRET_KEY=<AWS_SECRET_KEY>
    BASE64_AWS_SCAN_S3_ACCESS_KEY=$(echo $AWS_SCAN_S3_SECRET_KEY | base64)
    ```

    - edit scan-server-secrets

    ```sh
    kubectl edit secret scan-server-secrets
    ```

    ```yaml
      AWS_SCAN_S3_ACCESS_KEY: <BASE64_AWS_SCAN_S3_ACCESS_KEY>
      AWS_SCAN_S3_SECRET_KEY: <BASE64_AWS_SCAN_S3_SECRET_KEY>
    ```

- save and restart api pod

  ```sh
  kubectl delete pod api-<some-string>
  ```
