# Colab-Powered Poe API & n8n Server with Ngrok Tunneling

This project provides a Google Colab notebook setup to simultaneously run:
1.  A **Poe API Wrapper server**: Exposes an OpenAI-compatible API for interacting with various models on Poe.com, leveraging the `poe-api-wrapper` library.
2.  An **n8n server**: A powerful workflow automation tool.

Both services are made accessible over the internet using **ngrok** tunnels, allowing you to integrate them with other applications or access them remotely.

**Disclaimer:** The `snowby666/poe-api-wrapper` library used in this project was archived by its owner on March 10, 2025, and is now read-only [1]. While it may function, it will not receive further updates or bug fixes, and its functionality may break if Poe.com changes its API.

## Features

*   **Simultaneous Operation:** Runs both the Poe API wrapper and n8n servers concurrently in a single Colab environment.
*   **Ngrok Integration:** Automatically creates public URLs for both services using `pyngrok`.
*   **OpenAI-Compatible Endpoint:** The Poe API wrapper provides an endpoint that mimics the OpenAI API, allowing you to use Poe models with tools designed for OpenAI.
*   **Workflow Automation:** Full access to your self-hosted n8n instance for creating and managing automation workflows.
*   **Colab Based:** Easy to set up and run without requiring local machine resources for the servers (beyond the browser for Colab).

## Prerequisites

Before you begin, ensure you have the following:

1.  **Google Account:** To use Google Colab.
2.  **Poe Account:**
    *   You'll need your `p-b` and `p-lat` cookie values from Poe.com.
3.  **Ngrok Account:**
    *   A free or paid ngrok account.
    *   Your ngrok Authtoken.
4.  **Familiarity with Colab:** Basic understanding of how to run cells in a Colab notebook.

## Setup and Usage

The entire setup is managed through a Google Colab notebook.

**1. Obtain Necessary Tokens:**

*   **Poe `p-b` and `p-lat` Cookies:**
    1.  Log in to [Poe.com](https://poe.com/) in your browser.
    2.  Open your browser's developer tools (usually F12 or Right-click -> Inspect).
    3.  Go to the `Application` (Chrome/Edge) or `Storage` (Firefox) tab.
    4.  Find Cookies -> `https://poe.com`.
    5.  Copy the `value` for the `p-b` cookie.
    6.  Copy the `value` for the `p-lat` cookie.
*   **Ngrok Authtoken:**
    1.  Log in to your [ngrok Dashboard](https://dashboard.ngrok.com/).
    2.  Navigate to "Your Authtoken" (usually under Get Started -> Your Authtoken).
    3.  Copy your authtoken.

**2. Configure the Colab Notebook:**

*   Open the `Poe_n8n_Colab_Server.ipynb` (or your chosen notebook name) in Google Colab.
*   Locate **Cell 3 (Combined Server Startup)**.
*   Find the section marked `--- IMPORTANT: SET YOUR TOKENS HERE ---`.
*   Paste your copied tokens into the respective variables:
    ```python
    POE_P_B_TOKEN = "YOUR_P-B_COOKIE_VALUE_HERE"
    POE_P_LAT_TOKEN = "YOUR_P-LAT_COOKIE_VALUE_HERE"
    NGROK_AUTH_TOKEN = "YOUR_NGROK_AUTHTOKEN_HERE"
    ```

**3. Run the Colab Cells in Order:**

*   **Cell 1: Initial Installations**
    *   This cell installs necessary Python packages like `poe-api-wrapper`, `pyngrok`, `uvicorn`, and `nest_asyncio`.
    *   Run this cell once per Colab session.
*   **Cell 2: Node.js, nvm, and n8n Installation**
    *   This cell sets up Node.js using nvm (Node Version Manager) and then installs n8n globally.
    *   This is a lengthy cell and might take a few minutes.
    *   Run this cell once per Colab session (or if the Colab instance resets and loses the n8n installation). Wait for it to complete fully and verify that n8n version is printed at the end.
*   **Cell 3: Combined Server Startup (Poe API + n8n)**
    *   This is the main cell that starts both the Poe API wrapper server and the n8n server, and then creates ngrok tunnels for them.
    *   Ensure your tokens are correctly set in this cell.
    *   Run this cell. It will perform cleanup, start the services, and then run indefinitely to keep the servers and tunnels active.

**4. Access Your Services:**

*   Once Cell 3 is running successfully, it will output:
    *   The public URL for the **Poe API Wrapper** (e.g., `https://<random-string>.ngrok-free.app`). This URL acts as your `base_url` for OpenAI-compatible clients.
    *   The public URL for your **n8n instance** (e.g., `https://<another-random-string>.ngrok-free.app`). Open this URL in your browser to access your n8n dashboard. You may need to set up an admin user on your first visit to n8n.

**5. Stopping the Servers:**

*   To stop both servers and close the ngrok tunnels, interrupt the execution of Cell 3 in Colab (usually by clicking the "Stop" button next to the cell). The `finally` block in the script will handle the shutdown of processes and ngrok tunnels.

## Colab Notebook Structure

*   **Cell 1: Initial Installations:** Installs Python dependencies.
*   **Cell 2: Node.js, nvm, and n8n Installation:** Prepares the environment for n8n by installing Node.js via nvm and then n8n itself.
*   **Cell 3: Combined Server Startup (Poe API + n8n):**
    *   Sets user-specific tokens (Poe cookies, Ngrok authtoken).
    *   Clones the `poe-api-wrapper` repository (if not already present).
    *   Creates the `secrets.json` file required by the Poe API wrapper.
    *   Performs an aggressive ngrok cleanup to ensure a fresh start.
    *   Configures `pyngrok` with the authtoken.
    *   Starts the Poe API server (Uvicorn) in a separate thread.
    *   Waits for the Poe API server to initialize.
    *   Starts an ngrok tunnel for the Poe API server.
    *   Starts the n8n server in a separate thread.
    *   Waits for the n8n server to initialize.
    *   Starts an ngrok tunnel for the n8n server.
    *   Enters a loop to keep the Colab cell running and the servers active.
    *   Includes a `finally` block for graceful shutdown of all processes and tunnels upon interruption.

## Important Notes & Troubleshooting

*   **`poe-api-wrapper` Archive Status:** As mentioned, `snowby666/poe-api-wrapper` is archived [1]. If Poe.com makes breaking changes to their platform, the API wrapper may stop working.
*   **Ngrok Authtoken:** The `ERR_NGROK_4018` error indicates an issue with your ngrok authtoken or account verification. Ensure your token is correct and your ngrok account (especially email) is fully verified.
*   **Colab Resource Limits:** Google Colab has usage limits. Long-running sessions might be disconnected. If the notebook is disconnected, you'll need to re-run the cells.
*   **Initialization Times:** n8n, in particular, can take some time to start up the first time. The script includes `time.sleep()` calls to wait for initialization; if you encounter "connection refused" errors for ngrok, these waits might need to be slightly adjusted.
*   **Output Logs:** Cell 3 will print stdout and stderr for both the Poe API and n8n processes, prefixed with `[POE_API_STDOUT]`, `[N8N_STDERR]`, etc. Check these logs for any errors if services fail to start.

## License

This project setup is provided as-is. The licenses for the tools used are:
*   **poe-api-wrapper:** GPL-3.0 license [1]
*   **n8n:** Sustainable Use License / n8n Community License (refer to n8n's official licensing)
*   **ngrok:** Ngrok License (refer to ngrok's terms of service)

## Acknowledgements

*   The `snowby666/poe-api-wrapper` project for enabling Poe API access.
*   The teams behind n8n, ngrok, and pyngrok.

---
[1] GitHub - snowby666/poe-api-wrapper: [https://github.com/snowby666/poe-api-wrapper](https://github.com/snowby666/poe-api-wrapper)
