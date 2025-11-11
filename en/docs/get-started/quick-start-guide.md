# Quick Start Guide

WSO2 Integrator: ICP is a fully open-source, interface that enables developers to monitor and manage deployments effortlessly from a unified control plane, simplifying operations and improving efficiency.
In this quick start guide, you will learn how to download, install, and start using WSO2 Integrator: ICP in a few simple steps.

## Step 1: Download the Distribution 
1. Download and unzip [WSO2 Integrator: ICP](https://github.com/wso2/integration-control-plane/releases/download/v2.0.0-M1/wso2-integrator-icp-2.0.0-SNAPSHOT.zip) distribution.


## Step 2: Start the ICP server
1. Navigate to `<ICP_HOME>/bin` folder.
2. Execute one of the commands given below.
=== "On MacOS/Linux"

    ``` bash
    ./icp.sh
    ```

=== "On Windows"

    ``` bash
    icp.bat
    ``` 

## Step 3: Access the Dashboard
1. Navigate to `https://localhost:3000/` from browser.
2. Enter username as `admin` and password as `admin`.
>**Note:** If you get a "Network Error" when trying to log in, please open [https://localhost:9445/auth](https://localhost:9445/auth) in your browser and trust the certificate.
3. You should be able redirected to the `Home` page of the ICP console.
<a href="{{base_path}}/assets/img/get-started/quick-start-guide/home.png"><img src="{{base_path}}/assets/img/get-started/quick-start-guide/home.png" alt="introduction" width="70%"></a>

## Step 4: Create an integration in ICP
1. Create an integration in the ICP platform to establish the connection.
2. A reference project with a pre-configured integration is available to guide your setup. Create your own project and configure integrations based on your specific requirements.  
<a href="{{base_path}}/assets/img/get-started/quick-start-guide/sample_integration.png"><img src="{{base_path}}/assets/img/get-started/quick-start-guide/sample_integration.png" alt="Sample Inegration" width="70%"></a>