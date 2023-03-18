Ref: https://medium.com/geekculture/my-jq-cheatsheet-34054df5b650

-   `.key`
-   `.[]` (array)
-   `.[n]`, (Nth element of the array)
-   `.[n:m]`, (Nth to Mth elements of the array)
-   `A,B`, run two filters A and B with the same input
-   `A|B`, the output of A is the input of B
-   `(A)`, group operator

Examples:
```bash
kubectl get pod xxx -n {namespace} -ojson | jq ‘.kind’

kubectl -n ns get pods -ojson | jq '.items[].status'

kubectl -n test-ns get pods -ojson | jq '.items[].spec.nodeName' | xargs -I{} kubectl get node {} | grep ExternalIP

kubectl -n ns get pods -ojson | jq ‘ . | length’
```
