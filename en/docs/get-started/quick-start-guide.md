# Quick Start Guide

WSO2 Integrator: ICP is a fully open-source, interface that enables developers to monitor and manage deployments effortlessly from a unified control plane, simplifying operations and improving efficiency.
In this quick start guide, you will learn how to download, install, and start using WSO2 Integrator: ICP in a few simple steps.

## Prerequisites

Before you begin, ensure that you have the following prerequisites installed and configured:

- **Java Runtime Environment (JRE)**: Version 21 or higher
- **Operating System**: Windows, macOS, or Linux
- **Memory**: Minimum 1 GB RAM (2 GB or more recommended)
- **Disk Space**: Minimum 100 MB of free disk space
- **Web Browser**: A modern web browser (Chrome, Firefox, Safari, or Edge)
- **Network**: Internet connection for downloading the distribution

## Step 1: Download the Distribution 
1. Download and unzip [WSO2 Integrator: ICP](https://github.com/wso2/integration-control-plane/releases/download/v2.0.0-M4/wso2-integrator-icp-2.0.0-SNAPSHOT.zip) distribution.


## Step 2: Start the ICP server
1. Navigate to `<ICP_HOME>/bin` folder.
2. Execute one of the commands given below.
=== "On macOS/Linux"

    ``` bash
    ./icp.sh
    ```

=== "On Windows"

    ``` bash
    icp.bat
    ``` 

## Step 3: Access the Dashboard
1. Navigate to `https://localhost:9445/` from the browser.
2. Enter **`username`** as `admin` and **`password`** as `admin`.
3. You should be able redirected to the `Home` page of the ICP console.
<a href="{{base_path}}/assets/img/get-started/quick-start-guide/home.png"><img src="{{base_path}}/assets/img/get-started/quick-start-guide/home.png" alt="introduction" width="70%"></a>
