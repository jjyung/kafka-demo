# KAFKA DEMO

這是一個使用 Spring Boot 和 Apache Kafka 的範例專案，旨在展示 Kafka 的基本生產者 (Producer) 和消費者 (Consumer) 功能。

## 專案架構

此專案採用 Spring Boot 框架，並整合了 Spring for Apache Kafka。

- **`src/main/java/my/app`**: 專案的主要程式碼
  - **`config`**: Kafka 的設定，包括 Broker 位址、Topic 的建立。
    - `KafkaTopicConfig.java`: 設定了兩個 Topic: `baeldung` (1 個 partition) 和 `weather` (2 個 partition)。
  - **`controller`**: Spring MVC 控制器，提供 HTTP 端點來觸發 Kafka 訊息的發送與接收。
    - `HelloController`: 提供一個通用的發送訊息端點。
    - `WeatherStationController`: 模擬氣象站，發送天氣觀測資料。
    - `NewsDeskController`, `NewsDeskIIController`, `AgricultureAndFoodAgencyController`: 模擬不同的新聞台和機構，消費天氣資料。
  - **`service`**: 處理業務邏輯，例如產生和處理天氣資料。
  - **`manager`**: 封裝了 Kafka 的操作。
  - **`model`**: 資料模型，如 `MessageDTO` 和 `WeatherData`。
- **`src/main/resources/application.properties`**: 應用程式設定檔，其中設定了 Kafka Broker 的位址 (`localhost:29092`)。

## 環境準備

1. **安裝並執行 Kafka:**
    請確保您已經在本機安裝並啟動了 Kafka 和 Zookeeper。Broker 應監聽在 `localhost:29092`。
    如果您使用 Docker，可以使用類似以下的指令：

    ```bash
    # 啟動 Zookeeper
    docker run -d --name zookeeper -p 2181:2181 confluentinc/cp-zookeeper:latest

    # 啟動 Kafka
    docker run -d --name kafka -p 29092:29092 
      -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 
      -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:29092 
      -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 
      --link zookeeper:zookeeper 
      confluentinc/cp-kafka:latest
    ```

2. **執行 Spring Boot 應用程式:**
    使用 Gradle 包裝器來執行應用程式。

    ```bash
    ./gradlew bootRun
    ```

## 測試內容

應用程式啟動後，您可以透過 `curl` 或其他 HTTP 工具來測試以下端點：

### 1. 通用訊息發送

您可以發送任何訊息到任何已建立的 Topic。

- **發送訊息到 `baeldung` Topic:**

  ```bash
  curl -X POST http://localhost:8080/baeldung 
       -H "Content-Type: application/json" 
       -d '{"data": "Hello from Baeldung!"}'
  ```

  您可以在應用程式的日誌中看到消費者接收到此訊息。

### 2. 天氣觀測模擬

這個流程模擬一個氣象站 (Producer) 發布天氣資訊，並由多個新聞台 (Consumers) 接收。

- **步驟 1: 觸發氣象站發送觀測資料**
  執行以下指令，`WeatherStationController` 將會發送一筆模擬的天氣資料到 `weather` Topic。

  ```bash
  curl -X POST http://localhost:8080/weather-station/observation
  ```

- **步驟 2: 從不同的消費者獲取最新的天氣資訊**
  由於 `weather` Topic 有兩個 Partition，不同的 Consumer Group 可以同時消費。此專案中的 `NewsDeskService`, `NewsDeskIIService`, 和 `AgricultureAndFoodAgencyService` 代表了不同的消費者。
  
  您可以透過以下端點查詢它們各自收到的最後一筆天氣資料：

  - **新聞台 1:**

    ```bash
    curl http://localhost:8080/news-desk/weather/last
    ```

  - **新聞台 2:**

    ```bash
    curl http://localhost:8080/news-desk-2/weather/last
    ```

  - **農業與食品局:**

    ```bash
    curl http://localhost:8080/afa/weather/last
    ```

  重複執行步驟 1 多次，您可能會觀察到不同的消費者獲取到的 "last" 資料是不同的，這展示了 Kafka 的訊息分發機制。
