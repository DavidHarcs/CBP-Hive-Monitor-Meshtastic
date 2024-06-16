# Meshtastic Message Logger

**By the Community Bee Project**

## Overview

The Meshtastic Message Logger is a Python program designed to read messages from a Meshtastic device and store them in a SQLite database, sorted by the sender. This project aims to facilitate easy logging and retrieval of messages received through Meshtastic devices.

## Features

- Reads messages from a Meshtastic device.
- Stores messages in a SQLite database.
- Logs messages with sender information and timestamp.
- Simple and easy to set up.

## Requirements

- Python 3.6 or higher
- `meshtastic` library
- `sqlite3` library (included with Python)

## Installation

1. **Clone the repository:**

    ```bash
    git clone https://github.com/DavidHarcs/CBP-Hive-Monitor-Meshtastic.git
    cd meshtastic-message-logger
    ```

2. **Install the required Python packages:**

    ```bash
    pip install meshtastic
    ```

## Usage

1. **Connect your Meshtastic device** to your computer.

2. **Run the program:**

    ```bash
    python meshtastic_logger.py
    ```

3. **Program Output:**
    - The program will listen for incoming messages and log them to the `messages.db` SQLite database.
    - Messages are printed to the console with the sender information and stored in the database with a timestamp.

## Database Schema

The SQLite database `messages.db` contains a single table `messages` with the following columns:

- `id`: Integer, primary key, auto-incremented.
- `sender`: Text, the sender's ID.
- `message`: Text, the message content.
- `timestamp`: Datetime, default is the current timestamp.

## Code Overview

The main script is `meshtastic_logger.py`:

```python
import meshtastic
import meshtastic.serial_interface
import sqlite3
import json
import time

# Initialize SQLite database
conn = sqlite3.connect('messages.db')
c = conn.cursor()

# Create a table to store messages
c.execute('''
CREATE TABLE IF NOT EXISTS messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    sender TEXT NOT NULL,
    message TEXT NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
)
''')
conn.commit()

# Function to save message to database
def save_message(sender, message):
    c.execute("INSERT INTO messages (sender, message) VALUES (?, ?)", (sender, message))
    conn.commit()

# Function to handle incoming messages
def on_receive(packet, interface):
    try:
        sender = packet['from']
        message = packet['decoded']['data']['text']
        print(f"Received message from {sender}: {message}")
        save_message(sender, message)
    except KeyError as e:
        print(f"KeyError: {e}")

# Initialize Meshtastic interface
interface = meshtastic.serial_interface.SerialInterface()

# Set the callback for received messages
interface.onReceive = on_receive

# Keep the program running
try:
    print("Listening for messages...")
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("Exiting...")
finally:
    interface.close()
    conn.close()
