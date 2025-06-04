# Telegram Message Scheduler Bot

## Overview

This is a Python-based Telegram bot designed for group administrators to schedule messages. It offers features like:

*   **Named Schedules:** Assign a unique name to each schedule.
*   **Flexible Content:** Include text, media (photos/videos), and inline buttons.
*   **Timing Options:**
    *   Repeating intervals (e.g., every 300 seconds).
    *   Specific one-time dates and times (YYYY-MM-DD HH:MM:SS).
*   **Auto-Deletion/Overwrite & Cleanup:** Creating a new schedule with the same name as an existing one in the same group will automatically replace the old schedule. Importantly, the bot will also attempt to **delete any messages previously sent by the old, overwritten schedule** from the Telegram group. This ensures that only messages from the latest version of a named schedule remain visible.
*   **Persistence:** Schedules are stored in a MongoDB database and are reloaded when the bot restarts.
*   **Admin Only:** Only group administrators (including anonymous admins) can schedule or delete messages.

## Technologies Used

*   **Python:** Core language for the bot.
*   **Telethon:** Python library for interacting with the Telegram API.
*   **Schedule:** Library for scheduling jobs in Python.
*   **Pymongo:** Python driver for MongoDB.
*   **MongoDB:** NoSQL database used to store message schedules.
*   **python-dotenv:** For managing environment variables.

## Setup Instructions

1.  **Clone the repository (if applicable) or ensure `bot.py` is present.**

2.  **Install Dependencies:**
    Make sure you have `pip` (Python package installer) available. Install the required Python packages using the `requirements.txt` file:
    ```bash
    pip install -r requirements.txt
    ```

3.  **Set up MongoDB:**
    *   Install MongoDB on your system. For Ubuntu, you can follow the official MongoDB installation guide: [https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)
    *   Ensure the MongoDB service is running. Typically, this can be started with:
        ```bash
        sudo systemctl start mongod
        ```
    *   Verify its status:
        ```bash
        sudo systemctl status mongod
        ```

4.  **Create `.env` File:**
    *   In the same directory as `bot.py`, create a file named `.env`.
    *   Add the following configuration, replacing placeholder values with your actual credentials:
        ```env
        API_ID=your_api_id
        API_HASH=your_api_hash
        BOT_TOKEN=your_bot_token
        MONGODB_URI=mongodb://localhost:27017/
        ```
    *   `API_ID` and `API_HASH`: Obtain these from [https://my.telegram.org/apps](https://my.telegram.org/apps).
    *   `BOT_TOKEN`: Get this from BotFather on Telegram.
    *   `MONGODB_URI`: This is the connection string for your MongoDB instance. The default `mongodb://localhost:27017/` should work for a local installation.

5.  **Bot Permissions:**
    *   Add the bot to your Telegram group.
    *   Ensure it has permissions to:
        *   Send messages.
        *   Send photos and videos (if you plan to use media).
        *   **Delete messages** (for the auto-cleanup feature of overwritten schedules to work). If the bot lacks this permission, it will log an error when trying to delete old messages but will continue to function for scheduling.

6.  **Run the Bot:**
    ```bash
    python bot.py
    ```

## Bot Commands

*   `/start`: Displays a welcome message and basic information.
*   `/help`: Provides detailed instructions on how to use the bot and its commands.
*   `/schedule_message`: Initiates a conversational process to schedule a new message. The bot will prompt you step-by-step for:
    *   **Schedule name:** A unique identifier for your schedule (e.g., "Daily Reminder").
    *   **Message text:** The content of your message.
    *   **Media (optional):** Send a photo or video, or type `skip`.
    *   **Inline buttons (optional):** Add buttons using the format `ButtonText|button_url`. Add multiple buttons one by one, then type `skip`.
    *   **Time interval:** Enter seconds for a repeating schedule (e.g., `300` for every 5 minutes) or a specific future date/time for a one-time message (e.g., `2024-12-31 23:59:00`).
    The bot provides feedback and validation at each step.
*   `/list`: Shows all currently scheduled (unsent) messages for the chat, including their IDs.
*   `/delete <id>`: Deletes a scheduled message by its unique ID (obtained from `/list`). Only admins can use this.
*   `/cancel`: Cancels the current message scheduling process.

## Notes

*   **Logging:** The bot logs its activities to `bot.log` in the same directory as `bot.py`. Check this file for debugging if you encounter issues.
*   **Admin Privileges:** Operations like `/schedule_message` and `/delete` are restricted to group administrators.
*   **First Run Authentication:** The first time you run the bot, Telethon might prompt for authentication (e.g., phone number and code).
*   **Schedule Overwriting:** Remember that creating a new schedule with a name that already exists in that chat will replace the old one.
*   **Schedule Reloading & Robustness:** On restart, the bot reloads active (unsent) schedules from MongoDB.
    *   It attempts to reconnect to MongoDB multiple times if the initial connection fails.
    *   It validates schedules during loading and will skip (and log) any that are invalid (e.g., missing crucial data, invalid time format, or scheduled for a time that has already passed for one-time messages).
    *   It also skips schedules for chats that the bot can no longer access.

## Configuration Details

While the `.env` file is primarily for essential API credentials, the bot also uses other internal configuration settings defined in `bot.py`. These have sensible defaults and generally do not need changing, but are listed here for completeness:

*   `MONGODB_DATABASE`: Name of the database to use (default: `telegram_scheduler`).
*   `MONGODB_COLLECTION`: Name of the collection for messages (default: `messages`).
*   `MONGODB_TIMEOUT_MS`: MongoDB connection timeout in milliseconds (default: `5000`).
*   `LOG_FILE`: Name of the log file (default: `bot.log`, stored in the script's directory).
*   `SCHEDULE_CHECK_INTERVAL_SECONDS`: How often the bot checks for pending schedules to send (default: `1`).
*   `SESSION_NAME`: Name for the Telethon session file (default: `bot_session`).
*   `MONGODB_RETRIES`: Number of retries if MongoDB connection fails during startup (default: `3`).
*   `MONGODB_RETRY_DELAY_SECONDS`: Delay between MongoDB connection retries (default: `2`).

## Development

(This section can be expanded with details on contributing, project structure, etc., if needed in the future.)
