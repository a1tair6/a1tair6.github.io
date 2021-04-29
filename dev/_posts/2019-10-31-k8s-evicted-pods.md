# evicted pods

- evicted pods 가 발생함.
  - delete evicted pods
  - ```sh
    kubectl get pods | grep Evicted | awk '{print $1}' | xargs kubectl delete pod
    ```