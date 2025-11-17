# Connect Integration Runtimes to WSO2 Integrator: ICP

WSO2 Integrator: ICP allows you to connect BI and MI runtimes to the ICP server for centralized management and monitoring. 
This guide will walk you through the steps to connect your integration runtime to the ICP server.

- [Connecting WSO2 Integrator: BI Runtime to ICP](#connecting-wso2-integrator-bi-runtime-to-icp)
- Connecting WSO2 Integrator: MI Runtime to ICP


## Prerequisites
Before you begin, ensure you have the following prerequisites:

- ICP server up and running. If you haven't set up the ICP server yet, please refer to the [Quick Start Guide]({{base_path}}/get-started/quick-start-guide/) to set up the ICP server.
- Integration project created in ICP. If you haven't created a project yet, please refer to the [Create an Integration Project]({{base_path}}/get-started/create-project/) guide.
- BI/MI runtime installed. You can download the WSO2 Integrator: BI Runtime or WSO2 Integrator: MI Runtime from the [WSO2 website](https://wso2.com/integrator/).


## Connecting WSO2 Integrator: BI Runtime to ICP

### Step 1: Extract BI runtime Config from ICP
1. Navigate to the ICP dashboard at `https://localhost:9445/`.
2. Log in using your credentials. Default username and password are `admin`.
3. Go to the project dashboard where you created your integration project.
4. Click on the integration you want to connect.
<a href="{{base_path}}/assets/img/get-started/connect-integration/integration_details.png"><img src="{{base_path}}/assets/img/get-started/connect-integration/integration_details.png" alt="Integration Details" width="70%"></a>
5. In the integration details page, click on the **`Configure Runtime`** button located in the relevant environment card (e.g., Development, Production).
6. Copy the BI runtime configuration snippet provided.

### Step 2: Configure BI Runtime
1. Open the `Config.toml` file in the BI project. 
2. Paste the copied configuration snippet into the `Config.toml` file under the appropriate section.
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

