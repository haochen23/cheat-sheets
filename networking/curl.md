# Curl

Post json data from a local file named `sample.json`
```bash
curl --request POST --data "@sample.json" --header Content-Type:application/json http://localhost:8080/neworder
```