# Tutorials

Tutorials as a learning aid.

!!! warning
    Work in progress... :construction_worker:


### Alternative  
Create a file named `pgstac_event_logger.py` with the following content:

```python
"""
pgstac-event-logger (functions-framework variant)

A Knative function that receives CloudEvents from eoapi-notifier
(pgSTAC change events) and logs them to stdout.

The functions-framework library handles:
  - HTTP server binding
  - CloudEvent deserialisation (binary and structured encodings)
  - Returning 200 OK on success / 500 on unhandled exceptions
"""

import json
import logging
import sys

import functions_framework
from cloudevents.http import CloudEvent

logging.basicConfig(
    stream=sys.stdout,
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
)
logger = logging.getLogger("pgstac-event-logger")


@functions_framework.cloud_event
def handle_event(event: CloudEvent) -> None:
    """
    Receive a CloudEvent emitted by eoapi-notifier and log its attributes
    and data to stdout.

    The @cloud_event decorator:
      - Parses the incoming HTTP request as a CloudEvent (binary or structured).
      - Passes the deserialised CloudEvent object to this function.
      - Returns HTTP 200 when the function returns normally.
      - Returns HTTP 500 if an unhandled exception is raised.

    Args:
        event: The deserialised CloudEvent object.
    """
    logger.info(
        "Received CloudEvent | id=%s | type=%s | source=%s | time=%s",
        event["id"],
        event["type"],
        event["source"],
        event["time"],
    )

    data = event.data

    # Normalise data to a Python object for clean JSON logging
    if isinstance(data, (bytes, bytearray)):
        data = data.decode("utf-8")
    if isinstance(data, str):
        try:
            data = json.loads(data)
        except json.JSONDecodeError:
            pass  # log raw string if it is not valid JSON

    logger.info("Event data: %s", json.dumps(data, indent=2, default=str))
```

Dependency list for `requirements.txt`:
```
functions-framework==3.8.2
cloudevents==1.11.0
```

Note: The `functions-framework` library provides the necessary HTTP server and CloudEvent handling capabilities to run this code as a Knative function. 
The `cloudevents` library is used for working with CloudEvent objects.

## Step 2: Dockerfile
Next, create a `Dockerfile` to containerise the function:
```Dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY main.py .

ENV PORT=8080
EXPOSE 8080

# --target   : the decorated function name in main.py
# --signature-type: cloudevent tells the framework to expect CloudEvent requests
CMD ["functions-framework", \
     "--target=handle_event", \
     "--signature-type=cloudevent", \
     "--port=8080"]
```

### Build and Push the Container Image
Next you will have to build the container image and push it to a container registry:
```bash
# Build the image (replace with your registry and image name)
docker build -t ghcr.io/your-username/pgstac-event-logger:latest .
# Push the image to the registry
docker push ghcr.io/your-username/pgstac-event-logger:latest
```

Note: This is an example using GitHub Container Registry (ghcr.io). You can use any container registry you prefer, just make sure to update the image reference in the next steps accordingly.

## Step 3: Deploy the Knative Service
Now we will create a Knative Service that uses the container image we just built. Create a file named `knative-service.yaml` with the following content:
```yamlapiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: pgstac-event-logger
  namespace: na
spec:
  template:
    spec:
      containers:
        - image: ghcr.io/your-username/pgstac-event-logger:latest
          ports:
            - containerPort: 8080
``` 