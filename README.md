# thanos-setup

**Scaling with Thanos**

To get started, you need a working Prometheus deployment. Install Thanos as an additional component and configure its components based on your specific requirements and settings (e.g. Prometheus endpoints, object storage credentials and compaction rules).

Integrate Thanos via the following steps.

Step 1: Clone Thanos Repository

Clone the Thanos repository from GitHub.

   ```
   git clone https://github.com/thanos-io/thanos.git
   ```

If you use Docker, navigate to the cloned Thanos repository and build the Thanos Docker images via the Dockerfiles using this command.

   ```
   cd thanos
   docker build -t thanos:latest .
   ```

Step 2: Run Prometheus with Thanos Sidecar

Start Prometheus and configure it to scrape metrics from a Sidecar instance. Create a `prometheus.yaml` file with the following content.

   ```
   global:
     scrape_interval: 15s
   scrape_configs:
   - job_name: 'prometheus'
     static_configs:
     - targets: ['localhost:9090']
   remote_write:
     - url: "http://sidecar:9201/api/v1/receive"
   ```

Run the following command to start Prometheus and mount the `prometheus.yaml` file.

   ```
   docker run -d -p 9090:9090 -v /path/to/prometheus.yaml:/etc/prometheus/prometheus.yml --name prometheus prom/prometheus
   ```

Step 3: Run Thanos Sidecar

Start the Thanos Sidecar to connect to Prometheus and upload the data to object storage. Run the following command.

   ```
   docker run -d -p 9201:9201 --link prometheus:prometheus -e OBJECT_STORAGE_CONFIG="/etc/config/object-storage.yml" -v /path/to/object-storage.yml:/etc/config/object-storage.yml thanos:latest sidecar --prometheus.url=http://prometheus:9090
   ```

Step 4: Configure Object Storage

Set up an account on your preferred object storage provider and obtain the required access credentials. Create an `object-storage.yml` file to configure object storage integration. Replace `my-bucket` with the preferred name and add the necessary credentials for your chosen object storage provider. Here's an example.

   ```
   type: S3
   config:
     bucket: my-bucket
     endpoint: s3.amazonaws.com
     access_key: <your-access-key>
     secret_key: <your-secret-key>
   ```

Step 5: Run Thanos Store

Start the Thanos Store component by running the following command.

   ```
   docker run -d -p 10901:10901 -e OBJECT_STORAGE_CONFIG="/etc/config/object-storage.yml" -v /path/to/object-storage.yml:/etc/config/object-storage.yml --name store thanos:latest store --data-dir=store --objstore.config-file=/etc/config/object-storage.yml
   ```

Step 6: Query Thanos

Run the Thanos Query component to query aggregated metrics from multiple Prometheus instances and object storage. Execute the following command.

   ```
   docker run -d -p 9091:10901 --link store:store -e OBJECT_STORAGE_CONFIG="/etc/config/object-storage.yml" -v /path/to/object-storage.yml:/etc/config/object-storage.yml thanos:latest query --store=store:10901
   ```

Step 7: Access Thanos Web UI

Open your web browser and visit `http://localhost:9091` to access the Thanos Query Web UI. From there, you can explore and query the aggregated metric data.

Step 8: Explore Advanced Features

Once the basic setup is working, you can explore advanced features like data compaction with Compactor, rule evaluation with Ruler, and query federation with Query Frontend.
