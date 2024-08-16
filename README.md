# 🚀 RedisInsight Docker Automated Configuration

<p align="center">
    <img src="https://img.shields.io/badge/Redis-FF4438.svg?style=for-the-badge&logo=Redis&logoColor=white" alt="Redis">
    <img src="https://img.shields.io/badge/Docker-2496ED.svg?style=for-the-badge&logo=Docker&logoColor=white" alt="Docker">
    <img src="https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white"/>
    <img src="https://img.shields.io/badge/Linux-FCC624.svg?style=for-the-badge&logo=Linux&logoColor=black"/>
    <img src="https://img.shields.io/badge/GNU%20Bash-4EAA25.svg?style=for-the-badge&logo=GNU-Bash&logoColor=white"/>
</p>

Welcome to the **RedisInsight Docker AutoConfig** repository! This repository includes a Docker Compose setup for RedisInsight, alongside a powerful Bash script that automates the configuration of multiple Redis databases within RedisInsight. Simplify your workflow and let the script handle the heavy lifting for you!

---

## ✨ Features

Here’s what the script does for you:
1. **Network Configuration**: Automatically retrieves the gateway IP of your specified Docker network.
2. **Initial Setup**: Applies essential settings to RedisInsight, including EULA acceptance.
3. **Redis Integration**: Adds multiple Redis databases to RedisInsight in one go.

## 🚨 Docker Compose Setup

Use the following `docker-compose.yml` to set up your RedisInsight service:

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

---

## 📝 Script Overview

The `add_datasources.sh` script automates adding Redis databases to RedisInsight via its API. Here’s what it covers:

### 1. Configuration

The script starts by defining the API URL for RedisInsight and the Docker network name. It then retrieves the gateway IP for your specified network. If the network is not found, it defaults to `127.0.0.1`:

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

### 2. Apply Initial Settings

The script applies essential settings to RedisInsight, such as accepting the EULA and disabling analytics:

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

### 3. Define and Add Redis Databases

The script iterates over a list of Redis instances, adding each one to RedisInsight:

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

---

## 🚀 How to Use

1. **Customize the Script**: Edit the configuration variables to match your RedisInsight setup and Docker network. Modify the `REDIS_INSTANCES` array to include your Redis databases.

2. **Run the Script**: Make the script executable and run it:

    ```bash
    chmod +x add_datasources.sh
    ./add_datasources.sh
    ```

3. **Verify the Results**: Log into RedisInsight and check that your databases have been added successfully.

![image](https://github.com/user-attachments/assets/f6276046-624e-4690-bc28-d0d5df9a1eac)

---

## 📜 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
