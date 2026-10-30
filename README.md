# CloseCore Plugins

Official CloseCore plugins for Claude, ChatGPT, and Codex.

## Install in Claude

The CloseCore plugin bundles the CloseCore connector and skills. These steps work in Claude on the
web and Claude Desktop. Pick the section that matches your Claude plan and how you want to install
the plugin: from this GitHub repository, or from a zip file when you do not have a GitHub account.

- Marketplace repository: `https://github.com/closecore/closecore-plugins`
- Latest plugin zip: `https://production-api.closecore.com/api/mcp/plugin`
- MCP server URL: `https://production-api.closecore.com/mcp`

### Team or Enterprise plan, from this repository

Members cannot add custom connectors themselves. An Owner or Primary Owner (on Enterprise, also a
custom role that can manage Libraries) sets up the connector once for the organization.

Owner steps:

1. Open **Organization settings → Connectors**, select **Add**, then **Custom**. If Claude asks for
   the connector type, choose **Web**.
2. Enter the MCP server URL `https://production-api.closecore.com/mcp`.
3. Under **Authentication**, select **Sign in now**. Under **OAuth client**, select
   **Register automatically**. Leave request headers blank and select **Add**.

Steps for each member:

4. Open **Customize → Connectors**, find **CloseCore** (labeled **Custom**), and select **Connect**.
5. Complete the CloseCore sign-in, choose your organization, and approve the requested access.
6. Open **Customize → Plugins → Add → Add marketplace**, select **Add from a repository**, enter
   `https://github.com/closecore/closecore-plugins`, and select **Sync**. This requires your
   organization to allow members to add marketplaces, and Claude may ask you to connect a GitHub
   account. If either is a problem, an Owner can use the zip file instead.
7. Open the **CloseCore** plugin and select **Add**.
8. Start a chat and type `/closecore` to use CloseCore.

### Team or Enterprise plan, from a zip file

Members cannot add custom connectors themselves. An Owner or Primary Owner (on Enterprise, also a
custom role that can manage Libraries) sets up CloseCore once for the organization.

Owner steps:

1. Open **Organization settings → Connectors**, select **Add**, then **Custom**. If Claude asks for
   the connector type, choose **Web**.
2. Enter the MCP server URL `https://production-api.closecore.com/mcp`.
3. Under **Authentication**, select **Sign in now**. Under **OAuth client**, select
   **Register automatically**. Leave request headers blank and select **Add**.
4. Download the latest plugin zip from `https://production-api.closecore.com/api/mcp/plugin`.
5. Open **Organization settings → Plugins & skills**, select **Add → Upload a plugin**, and choose
   the downloaded file. Set **Default access** to **Installed by default** so members get the
   plugin without extra steps. When CloseCore releases a new version, download it again and select
   **Upload new version** from the plugin's menu. Members get the new version automatically.

Steps for each member:

6. Open **Customize → Connectors**, find **CloseCore** (labeled **Custom**), and select **Connect**.
7. Complete the CloseCore sign-in, choose your organization, and approve the requested access.
8. If the plugin is not already installed, open **Customize → Plugins → Discover**, select
   **CloseCore**, and select **Add**.
9. Start a chat and type `/closecore` to use CloseCore.

### Personal plan (Free, Pro, Max), from this repository

1. Open **Customize → Plugins → Add → Add marketplace**.
2. Select **Add from a repository**, enter `https://github.com/closecore/closecore-plugins`, and
   select **Sync**. Claude may ask you to connect a GitHub account at this step. If you do not have
   one, use the zip file instead.
3. Open the **CloseCore** plugin and select **Add**.
4. Open the installed plugin's **Connectors** tab and select **Connect** next to the CloseCore
   connector.
5. On **Add custom connector**, wait for **Server found**, then select **Continue**.
6. Under **Authentication**, select **Sign in now**. Under **OAuth client**, select
   **Register automatically**. Leave request headers blank and continue.
7. Complete the CloseCore sign-in, choose your organization, and approve the requested access.
8. Start a chat and type `/closecore` to use CloseCore.

### Personal plan (Free, Pro, Max), from a zip file

1. Download the latest plugin zip from `https://production-api.closecore.com/api/mcp/plugin`.
2. Open **Customize → Plugins → Add → Upload plugin** and choose the downloaded file.
3. Open the installed plugin's **Connectors** tab and select **Connect** next to the CloseCore
   connector.
4. On **Add custom connector**, wait for **Server found**, then select **Continue**.
5. Under **Authentication**, select **Sign in now**. Under **OAuth client**, select
   **Register automatically**. Leave request headers blank and continue.
6. Complete the CloseCore sign-in, choose your organization, and approve the requested access.
7. Start a chat and type `/closecore` to use CloseCore.

Uploaded plugins do not update on their own. When CloseCore releases a new version, remove the
plugin and upload the new file.

The ChatGPT and Codex manifests are included for compatibility testing. CloseCore currently only
provides customer setup instructions for Claude.
