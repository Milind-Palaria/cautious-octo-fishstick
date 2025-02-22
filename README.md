Here's a **README.md** file designed for an absolute beginner, explaining the overall flow, details of each file, and the process to set up and test the MQTT connector. This will guide someone step-by-step through the process.

---

# **MQTT Connector for Kafka: Setup and Testing Guide**

## **Overview of the MQTT Connector Flow**

This guide walks you through the setup and testing of the MQTT connector, which is used to connect to an MQTT broker, consume messages, and perform actions based on the data. The flow of the MQTT connector consists of several files working together.

### **Overall Flow of Files:**

1. **`mqtt_connector.py`:**
   - This is the main file where the logic for connecting to an MQTT broker and reading messages is implemented. It has a class called `MQTTConnector`, which defines methods for testing the connection and reading messages from an MQTT topic.
2. **`configuration.py`:**
   - This file defines configuration schemas using Pydantic models. These models describe the structure of the configuration data that the connector will use, including broker connection details, topic information, and message format.
3. **`test_mqtt_connector.py`:**
   - This is the test script you’ll use to ensure that the MQTT connector is working correctly. It runs the connector, tests the connection to the broker, and reads messages from the specified topic.

### **How These Files Work Together:**

- **`mqtt_connector.py`** contains the core logic for the connector (connecting to the MQTT broker, reading messages, and processing them).
- **`configuration.py`** is used to define the structure and validation for the configuration data, such as the broker URL and topic name.
- **`test_mqtt_connector.py`** is the file where we run the actual tests to ensure everything is set up correctly. It will call the methods defined in **`mqtt_connector.py`** to test the connection and read messages from the topic.

---

## **What Each File Does:**

### **1. `mqtt_connector.py`:**

- **Purpose:** Handles the core logic of connecting to the MQTT broker and reading messages from the MQTT topic.
- **Main Components:**
  - **`MQTTConnector` class:** Contains methods like `test()` for testing the connection and `read()` for reading messages from the topic.
  - **`on_message()` method:** This method is triggered whenever a new message arrives on the subscribed topic. It processes the message and stores it in the appropriate format.

### **2. `configuration.py`:**

- **Purpose:** Defines the configuration schema for the MQTT connector, including the broker details and the format of the messages.
- **Main Components:**
  - **`MQTTConnectorConfigSchema`:** Pydantic schema to validate the configuration for the MQTT connector (e.g., broker URL, port, etc.).
  - **`MQTTConnectorParamsSchema`:** Defines parameters like the topic and message format.

### **3. `test_mqtt_connector.py`:**

- **Purpose:** This is the test file used to check if the MQTT connector is correctly reading messages from the MQTT broker.
- **Main Components:**
  - **Test Setup:** It initializes the configuration and parameters and creates an instance of the `MQTTConnector` class.
  - **Test Execution:** It runs the `test()` method to ensure the connection works and calls the `read()` method to consume messages from the topic.

---

## **How to Set Up and Test the MQTT Connector**

### **Step 1: Set Up the MQTT Broker**

Before running the tests, you need to have an MQTT broker running. You can either use a local MQTT broker like **Mosquitto** or a public broker for testing.

- **Install Mosquitto** using Homebrew on macOS:

  ```bash
  brew install mosquitto
  ```

- **Start Mosquitto**:

  ```bash
  mosquitto
  ```

  By default, Mosquitto runs on **port 1883**.

### **Step 2: Modify the Configuration File**

Update the **`configuration.py`** file with your MQTT broker's information.

```python
class MQTTConnectorConfigSchema(BaseConnectorConfigSchema):
    broker_host: str = Field(
        title="Broker Host",
        description="Host address of the MQTT broker (e.g., mqtt.example.com)"
    )
    broker_port: int = Field(
        title="Broker Port",
        description="Port for connecting to the MQTT broker (e.g., 1883 for non-secure)"
    )

    use_tls: bool = Field(
        title="Use TLS",
        description="Whether to use TLS for secure communication",
        default=False
    )

class MQTTConnectorParamsSchema(BaseConnectorParamsSchema):
    topic: str = Field(
        title="Topic",
        description="MQTT topic to subscribe to"
    )
```

In this file, set the following:

- `broker_host` to `"localhost"` if using a local broker.
- `broker_port` to `1883` if using the default port for Mosquitto.
- `topic` to the name of the MQTT topic you want to subscribe to.

### **Step 3: Run the Test Script**

Now, you can run the test script to verify if the MQTT connector is working.

```bash
python test_mqtt_connector.py
```

This will:

1. Test the connection to the broker.
2. Subscribe to the specified topic.
3. Print any received messages.

### **Step 4: Test the MQTT Connector**

- Use a tool like **MQTT.fx** or **Mosquitto client** to publish messages to the topic you're subscribing to in the connector.
- For example, using the Mosquitto client:

  ```bash
  mosquitto_pub -h localhost -t "test/topic" -m "Hello MQTT!"
  ```

- The connector will print out any messages it receives from the subscribed topic.

---

## **Line-by-Line Explanation of Each File**

### **1. `mqtt_connector.py`:**

```python
import paho.mqtt.client as mqtt
from typing import Any, Optional, AsyncGenerator
import polars as pl
from nixdlt.plugins.base.connectors import BaseConnector, ConnectorReadResponse
from nixdlt.server.schemas.connections import TestConnectionResult
from nixdlt.utils.logging import logger
from .configuration import MQTTConnectorConfigSchema, MQTTConnectorParamsSchema
```

- **Imports:** Import necessary libraries like `paho.mqtt` (for MQTT communication), `polars` (for data processing), and the connector base classes.

```python
class MQTTConnector(BaseConnector[MQTTConnectorConfigSchema, MQTTConnectorParamsSchema]):
```

- **`MQTTConnector class`:** This is the main class that handles the connection and message reading.

```python
async def test(self) -> TestConnectionResult:
```

- **`test()` method:** Tests the connection to the MQTT broker by connecting and disconnecting.

```python
client = mqtt.Client()
if config.use_tls:
    client.tls_set()
```

- **TLS Configuration:** If the `use_tls` flag is set, it configures the client to use secure communication.

```python
client.connect(config.broker_host, config.broker_port)
client.loop_start()
client.disconnect()
```

- **Connecting to the Broker:** Connect to the specified MQTT broker, start the loop, and then disconnect after a short time.

```python
async def read(self, config: MQTTConnectorConfigSchema, params: MQTTConnectorParamsSchema, checkpoints: Optional[dict[str, Any]] = None, connector_checkpoints: Optional[dict[str, Any]] = None) -> AsyncGenerator[ConnectorReadResponse, None]:
```

- **`read()` method:** Subscribes to the specified topic and processes incoming messages.

```python
client.subscribe(params.topic)
client.loop_forever()
```

- **Subscribing and Listening:** The client subscribes to the given topic and listens for messages indefinitely.

---

### **2. `configuration.py`:**

```python
class MQTTConnectorConfigSchema(BaseConnectorConfigSchema):
    broker_host: str = Field(
        title="Broker Host",
        description="Host address of the MQTT broker (e.g., mqtt.example.com)"
    )
    broker_port: int = Field(
        title="Broker Port",
        description="Port for connecting to the MQTT broker (e.g., 1883 for non-secure)"
    )
```

- **`broker_host` and `broker_port`:** These fields hold the connection details for the MQTT broker.

```python
class MQTTConnectorParamsSchema(BaseConnectorParamsSchema):
    topic: str = Field(
        title="Topic",
        description="MQTT topic to subscribe to"
    )
```

- **`topic`:** This defines the MQTT topic to subscribe to.

---

### **3. `test_mqtt_connector.py`:**

```python
from mqtt_connector import MQTTConnector
from mqtt_connector.configuration import MQTTConnectorConfigSchema, MQTTConnectorParamsSchema
```

- **Imports:** Import the necessary classes to initialize the connector and configuration.

```python
config = MQTTConnectorConfigSchema(
    broker_host="localhost",  # MQTT broker address
    broker_port=1883,         # Port (default for Mosquitto)
    use_tls=False
)
params = MQTTConnectorParamsSchema(topic="test/topic")
```

- **Configuration:** Set up the configuration for the MQTT broker and topic.

```python
connector = MQTTConnector()
```

- **Connector Setup:** Instantiate the `MQTTConnector` class.

```python
async def run_test():
    test_result = connector.test(config)
    print("Test Result:", test_result.status, test_result.detail)
```

- **Run Test:** Call the `test()` method to verify the connection and print the result.

---

### **Conclusion**

This guide walks you through the complete process of setting up, testing, and understanding the MQTT connector in Python. You can now follow these steps to connect to an MQTT broker, consume messages, and test the setup with a simple script.

---

---

---

---

---

docker pull eclipse-mosquitto:2.0.15
docker run -it -p 1883:1883 eclipse-mosquitto:2.0.15
mosquitto_pub -h localhost -p 1883 -t "test/topic" -m '{"message": "Retained Message"}' -r

Great! Now that you've successfully pulled the `eclipse-mosquitto:2.0.15` image, you can run it to test your MQTT connector with a Docker-based broker. Here's how you can proceed step-by-step:

---

### **Step 1: Run the Mosquitto Docker Container**

1. **Start the Broker Container:**

   Run the following command to start the Mosquitto container on port 1883:

   ```bash
   docker run -it -p 1883:1883 eclipse-mosquitto:2.0.15
   ```

   This command will run Mosquitto and map port 1883 of the container to port 1883 on your host.

2. **Verify the Broker is Running:**

   In a new terminal, you can use the Mosquitto client tools to subscribe and publish messages:

   - **Subscribe:**
     ```bash
     mosquitto_sub -h localhost -p 1883 -t "test/topic"
     ```
   - **Publish:**
     ```bash
     mosquitto_pub -h localhost -p 1883 -t "test/topic" -m "Hello from Docker!"
     ```

   You should see the message "Hello from Docker!" in your subscriber terminal.

---

### **Step 2: Test Your MQTT Connector**

1. **Prepare Your Test Script:**

   Create a Python test script (e.g., `test_mqtt_connector.py`) that uses your MQTT connector. For example:

   ```python
   import asyncio
   from your_connector_module import MQTTConnector  # Replace with your module path
   from your_connector_module.configuration import MQTTConnectorConfigSchema, MQTTConnectorParamsSchema

   # Set up the configuration to connect to your local broker running in Docker.
   config = MQTTConnectorConfigSchema(
       broker_host="localhost",
       broker_port=1883,
       use_tls=False
   )

   params = MQTTConnectorParamsSchema(
       topic="test/topic",
       message_format="json"  # Change as needed (json, string, avro)
   )

   # Instantiate the connector and set its config.
   connector = MQTTConnector()
   connector.config = config  # Used in the test() method.

   async def run_test():
       # Test connection first.
       test_result = await connector.test()
       print("Test Result:", test_result.status, test_result.detail)

       # Read messages until the timeout.
       async for response in connector.read(config, params):
           print("Received message:", response.dataframe)
           print("Checkpoint:", response.checkpoints)

   if __name__ == "__main__":
       asyncio.run(run_test())
   ```

   **Note:**

   - Replace `"your_connector_module"` with the actual Python module name where your MQTT connector is implemented.
   - Ensure your connector file (with the modifications for incremental reading) is available in your Python path.

2. **Run the Test Script:**

   With your Mosquitto container running, open a terminal in the directory containing your test script and run:

   ```bash
   python test_mqtt_connector.py
   ```

3. **Publish Some Messages:**

   While your test script is running, open another terminal and publish some messages:

   ```bash
   mosquitto_pub -h localhost -p 1883 -t "test/topic" -m '{"message": "Test Message 1"}'
   mosquitto_pub -h localhost -p 1883 -t "test/topic" -m '{"message": "Test Message 2"}'
   ```

   You should see these messages printed in the terminal running your test script.

4. **Observe the Timeout Behavior:**

   After publishing a few messages, stop publishing and wait for the `MESSAGE_TIMEOUT` period (e.g., 10 seconds). The connector should log that no new messages were received and then exit the reading loop, stopping the MQTT client.

---

### **Step 3: Debug and Iterate**

- **Logs and Output:**  
  Check the printed logs in your test script for messages and checkpoints. This will help you verify that your connector is processing messages correctly and stops reading after the timeout.

- **Adjust Timeout:**  
  If needed, you can adjust the `MESSAGE_TIMEOUT` value in your connector code to better suit your testing needs.

- **Additional Testing:**  
  Try sending messages in different formats (JSON, string, avro) to ensure your connector handles them as expected.

---

By following these steps, you'll be able to test your MQTT connector with a Mosquitto broker running in Docker. This will help you verify that your connector successfully reads messages until no new ones arrive (i.e., until the timeout is reached) and then gracefully stops.
