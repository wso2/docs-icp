# Connect Integration Runtimes to WSO2 Integrator: ICP

WSO2 Integrator: ICP allows you to connect BI and MI runtimes to the ICP server for centralized management and monitoring. 
This guide will walk you through the steps to connect your integration runtime to the ICP server.

- [Connecting WSO2 Integrator: BI Runtime to ICP](#connecting-wso2-integrator-bi-runtime-to-icp)
- [Connecting WSO2 Integrator: MI Runtime to ICP](#connecting-wso2-integrator-mi-runtime-to-icp)


## Prerequisites
Before you begin, ensure you have the following prerequisites:

- ICP server up and running. If you haven't set up the ICP server yet, please refer to the [Quick Start Guide]({{base_path}}/get-started/quick-start-guide/) to set up the ICP server.
- Integration project created in ICP. If you haven't created a project yet, please refer to the [Create an Integration Project]({{base_path}}/get-started/create-project/) guide.
- BI/MI runtime installed. You can download the WSO2 Integrator: BI Runtime or WSO2 Integrator: MI Runtime from the [WSO2 website](https://wso2.com/integrator/).


## Connecting WSO2 Integrator: BI Runtime to ICP

### Step 1: Extract BI runtime config from ICP
1. Navigate to the ICP dashboard at `https://localhost:9445/`.
2. Log in using your credentials. Default username and password are `admin`.
3. Go to the project dashboard where you created your integration project.
4. Click on the integration you want to connect.
<a href="{{base_path}}/assets/img/get-started/connect-integration/integration_details.png"><img src="{{base_path}}/assets/img/get-started/connect-integration/integration_details.png" alt="Integration Details" width="70%"></a>
5. In the integration details page, click on the **`Configure Runtime`** button located in the relevant environment card (e.g., Development, Production).
6. Copy the BI runtime configuration snippet provided.
<a href="{{base_path}}/assets/img/get-started/connect-integration/bi_config_snippet.png"><img src="{{base_path}}/assets/img/get-started/connect-integration/bi_config_snippet.png" alt="BI Config Snippet" width="70%"></a>

### Step 2: Configure BI Runtime
1. Open the `Config.toml` file in the BI project. 
2. Paste the copied configuration snippet into the `Config.toml` file section.
```toml
[anuruddha.wso2.icp]
integration="my-first-bi-integration"
project="My ICP Project"
environment="dev"
heartbeatInterval=10
```
3. Save the `Config.toml` file.
4. Open the `Ballerina.toml` file in the BI project.
5. Add the following configuration to the `Ballerina.toml` file:
    ```toml
    [build-options]
    remoteManagement = true
    ```
6. Open the `<main>.bal` file of your BI integration project.
7. Add the following import statement at the top of the file:
    ```ballerina
    import anuruddha/wso2.icp as _;
    ```
8. Restart the BI runtime to apply the changes.
9. Following console logs indicate a successful connection to the ICP server:
    ```
    Running executable
    time=2025-11-17T14:22:24.624+05:30 level=INFO module=anuruddha/wso2.icp message="Starting ICP agent..."
    time=2025-11-17T14:22:24.635+05:30 level=INFO module=anuruddha/wso2.icp message="Loaded ICP configuration: {\"icp\":{\"serverUrl\":\"https://localhost:9445\", \"cert\":\"\", \"enableSSL\":false, \"heartbeatInterval\":10}, \"observability\":{\"opensearchUrl\":\"\", \"logIndex\":\"bi-client-logs\", \"metricsEnabled\":false}}"
    time=2025-11-17T14:22:25.190+05:30 level=INFO module=anuruddha/wso2.icp message="ICP agent initialized with server URL: https://localhost:9445"
    time=2025-11-17T14:22:25.231+05:30 level=INFO module=anuruddha/wso2.icp message="ICP agent started successfully with job ID: {\"id\":382801}"
    time=2025-11-17T14:22:25.391+05:30 level=INFO module=anuruddha/wso2.icp message="Sending delta heartbeat to ICP server"
    time=2025-11-17T14:22:25.554+05:30 level=INFO module=anuruddha/wso2.icp message="Delta heartbeat acknowledged by ICP server"
    time=2025-11-17T14:22:25.555+05:30 level=INFO module=anuruddha/wso2.icp message="Received control commands: []"
    time=2025-11-17T14:22:25.556+05:30 level=INFO module=anuruddha/wso2.icp message="Delta heartbeat acknowledged by ICP server"
    time=2025-11-17T14:22:25.557+05:30 level=INFO module=anuruddha/wso2.icp message="ICP server requested full heartbeat"
    time=2025-11-17T14:22:25.559+05:30 level=INFO module=anuruddha/wso2.icp message="Sending full heartbeat to ICP server"
    time=2025-11-17T14:22:25.619+05:30 level=INFO module=anuruddha/wso2.icp message="Full heartbeat acknowledged by ICP server"
    ```
10. The BI runtime is now connected to the ICP server. You can monitor and manage your BI integration from the ICP dashboard.
<a href="{{base_path}}/assets/img/get-started/connect-integration/bi_connected.png"><img src="{{base_path}}/assets/img/get-started/connect-integration/bi_connected.png" alt="BI Connected" width="70%"></a>

## Connecting WSO2 Integrator: MI Runtime to ICP

### Step 1: Extract MI runtime config from ICP
1. Navigate to the ICP dashboard at `https://localhost:9445/`.
2. Log in using your credentials. Default username and password are `admin`.
3. Go to the project dashboard where you created your integration project.
4. Click on the integration you want to connect.
5. In the integration details page, click on the **`Configure Runtime`** button located in the relevant environment card (e.g., Development, Production).
6. Copy the MI runtime configuration snippet provided.
<a href="{{base_path}}/assets/img/get-started/connect-integration/mi_config_snippet.png"><img src="{{base_path}}/assets/img/get-started/connect-integration/mi_config_snippet.png" alt="MI Config Snippet" width="70%"></a>

### Step 2: Configure MI Runtime
1. Open the `<MI_HOME>\conf\deployment.toml` file in the MI runtime.
2. Paste the copied configuration snippet into the `deployment.toml` file under the appropriate section.
3. Save the `deployment.toml` file.
4. Restart the MI runtime to apply the changes.
5. Following console logs indicate a successful connection to the ICP server:    
   ```
     [2025-11-17 20:54:18,766]  INFO {ICPHeartBeatComponent} - Starting connector collection for ICP heartbeat
     [2025-11-17 20:54:18,766]  INFO {ICPHeartBeatComponent} - No connectors/libraries found in SynapseConfiguration
     [2025-11-17 20:54:18,766]  INFO {ICPHeartBeatComponent} - Starting registry resources collection for ICP heartbeat
     [2025-11-17 20:54:18,766]  INFO {ICPHeartBeatComponent} - Registry root path: /Users/anuruddha/Downloads/wso2mi-4.5.0-SNAPSHOT/registry/../
     [2025-11-17 20:54:18,771]  INFO {ICPHeartBeatComponent} - Found 21 registry resources to process
     [2025-11-17 20:54:18,771]  INFO {ICPHeartBeatComponent} - Registry resources collection completed. Processed: 21, Errors: 0, Total registry resources collected: 21
     [2025-11-17 20:54:18,771]  INFO {ICPHeartBeatComponent} - Registry resources response summary - Count: 21, Root directory: /Users/anuruddha/Downloads/wso2mi-4.5.0-SNAPSHOT/registry/../
     [2025-11-17 20:54:18,775]  INFO {ICPHeartBeatComponent} - Full heartbeat payload: {"runtime":"f22d1d20-ae12-4904-9a89-14fd6cd48ea6","runtimeType":"MI","status":"RUNNING","environment":"dev","project":"My ICP Project","component":"first-mi-integration","version":"4.4.0","nodeInfo":{"platformName":"wso2-mi","platformVersion":"4.4.0","platformHome":"/Users/anuruddha/Downloads/wso2mi-4.5.0-SNAPSHOT","osName":"Mac OS X","osVersion":"15.3","javaVersion":"21.0.3"},"artifacts":{"apis":[],"proxyServices":[],"endpoints":[],"inboundEndpoints":[],"sequences":[{"name":"fault","type":"Sequence"},{"name":"main","type":"Sequence"}],"tasks":[],"templates":[],"messageStores":[],"messageProcessors":[],"localEntries":[{"name":"SERVER_IP","type":"LocalEntry","valueType":"Entry"},{"name":"fault","type":"LocalEntry","valueType":"SequenceMediator"},{"name":"main","type":"LocalEntry","valueType":"SequenceMediator"},{"name":"SERVER_HOST","type":"LocalEntry","valueType":"Entry"}],"dataServices":[],"carbonApps":[],"dataSources":[],"connectors":[],"registryResources":[{"name":"dropins","mediaType":"directory","properties":[]},{"name":"repository","mediaType":"directory","properties":[]},{"name":"diagnostics-tool","mediaType":"directory","properties":[]},{"name":"release-notes.html","mediaType":"text/plain","properties":[]},{"name":"wso2","mediaType":"directory","properties":[]},{"name":"tmlog2.log","mediaType":"text/plain","properties":[]},{"name":"tmlog.lck","mediaType":"text/plain","properties":[]},{"name":"bin","mediaType":"directory","properties":[]},{"name":"patches","mediaType":"directory","properties":[]},{"name":"updates","mediaType":"directory","properties":[]},{"name":"wso2carbon.pid","mediaType":"text/plain","properties":[]},{"name":"registry","mediaType":"directory","properties":[]},{"name":"lib","mediaType":"directory","properties":[]},{"name":"README.txt","mediaType":"text/plain","properties":[]},{"name":"192.168.1.4.tm2.epoch","mediaType":"text/plain","properties":[]},{"name":"dbscripts","mediaType":"directory","properties":[]},{"name":"LICENSE.txt","mediaType":"text/plain","properties":[]},{"name":"backup","mediaType":"directory","properties":[]},{"name":"dockerfiles","mediaType":"directory","properties":[]},{"name":"conf","mediaType":"directory","properties":[]},{"name":"tmp","mediaType":"directory","properties":[]}],"listeners":[{"protocol":"http","port":9201,"host":"0.0.0.0"},{"protocol":"https","port":9164,"host":"0.0.0.0"}],"systemInfo":{"serverName":"WSO2 Micro Integrator","version":"4.4.0","carbonHome":"/Users/anuruddha/Downloads/wso2mi-4.5.0-SNAPSHOT","javaVersion":"21.0.3","javaVendor":"Eclipse Adoptium","osName":"Mac OS X","osVersion":"15.3","osArch":"aarch64","memory":{"totalMemory":278921216,"freeMemory":117194640,"maxMemory":1073741824,"usedMemory":161726576}}},"runtimeHash":"mBKqftH+U2SkzU6yYbZDYg=="}

   ```
6. The MI runtime is now connected to the ICP server. You can monitor and manage your MI integration from the ICP dashboard.
<a href="{{base_path}}/assets/img/get-started/connect-integration/mi_connected.png"><img src="{{base_path}}/assets/img/get-started/connect-integration/mi_connected.png" alt="MI Connected" width="70%"></a>