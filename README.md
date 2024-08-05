
# RedisInsight Docker AutoConfig

This repository contains a Docker Compose setup for RedisInsight and a Bash script to automate the configuration of Redis databases within RedisInsight. This script makes it easy to add multiple Redis instances to your RedisInsight setup, streamlining your workflow.

## Docker Compose Setup

The following `docker-compose.yml` sets up a RedisInsight service:

```yaml
services:
  redis_insight:
    image: "redislabs/redisinsight:latest"
    container_name: redisinsight
    ports:
      - "5540:5540"
    volumes:
      - redis_insight_data:/data
 
volumes:
  redis_insight_data:
```

## Script Overview

The `add_datasources.sh` script automates the process of adding Redis databases to RedisInsight using its API. The script performs the following tasks:

1. Retrieves the gateway IP address of a specified Docker network.
2. Applies initial settings to RedisInsight (e.g., EULA agreements).
3. Iterates over a list of Redis instances, adding each one to RedisInsight.

### Script Breakdown

#### 1. Configuration

The script begins by defining the API URL for RedisInsight and the Docker network name. It then retrieves the gateway IP for the specified network. If the network is not found, it defaults to `127.0.0.1`:

```bash
API_URL="http://localhost:5540/api"
NETWORK_NAME="backend-network"

GATEWAY_IP=$(docker network inspect "$NETWORK_NAME" --format '{{range .IPAM.Config}}{{.Gateway}}{{end}}')

if [ -z "$GATEWAY_IP" ]; then
  echo "Warning: Unable to retrieve gateway IP for network '$NETWORK_NAME'. Using 127.0.0.1 as the default."
  GATEWAY_IP="127.0.0.1"
fi

echo "Gateway IP for network '$NETWORK_NAME' is $GATEWAY_IP"
```

#### 2. Apply Initial Settings

The script applies initial settings to RedisInsight, such as accepting the EULA and disabling analytics:

```bash
SETTINGS_PAYLOAD='{
  "agreements": {
    "eula": true,
    "analytics": false,
    "notifications": false,
    "encryption": false
  }
}'

curl -X GET "${API_URL}/settings"      -H "Content-Type: application/json"      -d "$SETTINGS_PAYLOAD" > /dev/null 2>&1
```

#### 3. Define and Add Redis Databases

The script then iterates over a list of Redis instances, adding each one to RedisInsight:

```bash
# Redis instances configuration
REDIS_INSTANCES=(
  "Gateway Redis|6490|gateway_password|gateway_redis"
  "User Redis|6235|user_password|user_redis"
  "Restaurant Redis|6236|restaurant_password|restaurant_redis"
  "Location Redis|6300|location_password|location_redis"
  "Order Redis|6301|order_password|order_redis"
)

# Function to add a Redis database to RedisInsight
add_redis_database() {
  local NAME=$1
  local PORT=$2
  local PASSWORD=$3
  local DB_NAME=$4

  echo "Adding Redis database: $NAME..."

  curl -s -X POST "${API_URL}/databases"        -H "Content-Type: application/json"        -d "{
            "host": "${GATEWAY_IP}",
            "port": ${PORT},
            "password": "${PASSWORD}",
            "name": "${DB_NAME}"
          }" > /dev/null

  if [ $? -eq 0 ]; then
    echo "Successfully added Redis database: $NAME"
  else
    echo "Error: Failed to add Redis database: $NAME"
  fi
}

# Iterate over the Redis instances and add them to RedisInsight
for redis in "${REDIS_INSTANCES[@]}"; do
  IFS='|' read -r REDIS_NAME REDIS_PORT REDIS_PASSWORD REDIS_DB_NAME <<< "$redis"
  add_redis_database "$REDIS_NAME" "$REDIS_PORT" "$REDIS_PASSWORD" "$REDIS_DB_NAME"
done

echo "All Redis databases have been added successfully."
```

### Usage

1. **Customize the Script**: Update the configuration variables with your RedisInsight details and Docker network name. Modify the `REDIS_INSTANCES` array to include your Redis databases.

2. **Run the Script**: Make the script executable and run it:

    ```bash
    chmod +x add_![image](https://github.com/user-attachments/assets/af131d10-dc22-4802-8728-8f30f3788d74).sh
    ./add_datasources.sh
    ```

3. **Check Results**: Verify that the Redis databases have been added to your RedisInsight instance by logging into RedisInsight and checking the list of databases.

![image](https://github.com/user-attachments/assets/f6276046-624e-4690-bc28-d0d5df9a1eac)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
