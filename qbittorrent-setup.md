# qBittorrent Setup Guide for WEEB-bot

This guide walks you through preparing a standard qBittorrent installation to work with WEEB-bot, from enabling the Web UI to entering your credentials into the Excel file.

---

## Step 1 — Enable the qBittorrent Web UI

WEEB-bot communicates with qBittorrent through its built-in Web UI. This is disabled by default and must be turned on manually.

1. Open qBittorrent
2. Go to **Tools → Preferences** (or press `Alt + O`)
3. Click the **Web UI** tab on the left
4. Check the box labelled **"Enable Web User Interface (Remote control)"**
5. Leave the **IP address** set to `127.0.0.1` (this means only programs running on the same computer can connect, which is what you want)
6. Leave the **port** set to `8080` unless that port is already in use by something else on your system
   - If you change the port, make sure to update `QB_PORT` in the `USER CONFIGURATION` section of `WEEB-bot.py` to match
7. Click **Save**

---

## Step 2 — Create a Username and Password

Still in the **Web UI** tab:

1. Under **Authentication**, uncheck **"Bypass authentication for clients on localhost"** if it is checked — WEEB-bot uses the username and password to log in and this bypass would interfere
2. Set a **username** of your choice (for example: `WEEB-bot`)
3. Set a **password** of your choice — pick something reasonably secure since the Web UI is a live login endpoint
4. Click **Save**

> Keep a note of the username and password you chose — you will need to enter them into the Excel file in Step 4.

---

## Step 3 — Create a Download Category

WEEB-bot tags every torrent it adds with a category. This makes it easy to see at a glance which torrents were added by WEEB-bot, and lets you apply category-specific rules (like a dedicated save path) in qBittorrent if you want to.

1. In the qBittorrent main window, right-click anywhere in the **Categories** panel on the left sidebar
2. Select **"Add category..."**
3. Enter a category name — for example: `WEEB-bot`
4. Optionally set a **save path** for this category if you want WEEB-bot downloads to go to a specific folder
5. Click **OK**

> The category name you choose here must be entered into the Excel file in Step 4 and must match exactly, including capitalisation.

---

## Step 4 — Enter Your Credentials into the Excel File

WEEB-bot reads your qBittorrent username, password and category from a dedicated sheet in `Downloadlist.xlsx` called **`QB_Credentials`**. This keeps your credentials out of the Python script, so the script can be shared or stored in a repository without exposing sensitive information.

1. Open `Downloadlist.xlsx`
2. Navigate to the sheet named **`QB_Credentials`**
3. The sheet has three columns with the following headers in row 1:

   | QB_Username | QB_Password | QB_Category |
   |---|---|---|

4. In **row 2**, enter the values you chose in Steps 2 and 3:

   | QB_Username | QB_Password | QB_Category |
   |---|---|---|
   | `WEEB-bot` | `your_password_here` | `WEEB-bot` |

5. Save and close the Excel file

> **Important:** `Downloadlist.xlsx` contains your password and should **not** be committed to a public repository. If you are using Git, add `Downloadlist.xlsx` to your `.gitignore` file to prevent it from being uploaded accidentally. The repository includes a blank template version of the file for reference — use that as your starting point but keep your personal filled-in copy local only.

---

## Step 5 — Verify the Connection

Before running WEEB-bot for the first time, you can verify the Web UI is reachable:

1. Make sure qBittorrent is running
2. Open a browser and go to: `http://127.0.0.1:8080`
3. You should see the qBittorrent Web UI login page
4. Log in with the username and password you set in Step 2

If the page loads and you can log in, WEEB-bot will be able to connect. If the page does not load, double-check that:
- qBittorrent is running
- The Web UI is enabled (Step 1)
- The port in your browser matches `QB_PORT` in `WEEB-bot.py` (default: `8080`)

---

## Summary of Values to Note

| Setting | Where to configure it |
|---|---|
| Web UI port (default `8080`) | qBittorrent → Tools → Preferences → Web UI, and `QB_PORT` in `WEEB-bot.py` if changed |
| Username | qBittorrent → Tools → Preferences → Web UI → Authentication, and `QB_Credentials` sheet in `Downloadlist.xlsx` |
| Password | qBittorrent → Tools → Preferences → Web UI → Authentication, and `QB_Credentials` sheet in `Downloadlist.xlsx` |
| Category name | qBittorrent → Categories panel (right-click → Add category), and `QB_Credentials` sheet in `Downloadlist.xlsx` |
