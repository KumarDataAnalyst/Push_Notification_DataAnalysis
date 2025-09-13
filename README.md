# Instagram-Push-Notifications_DataAnalysis
# Project summary

This repo demonstrates a small push-notification system using Firebase Cloud Messaging (browser client + service worker), a Flask backend that stores incoming notifications in MySQL, and a Python script to send FCM notifications. The repository contains:

* index.html — Browser client that requests permission, obtains an FCM token, registers the service worker, handles foreground messages and posts received notifications to the backend.
* firebase-messaging-sw.js — Service worker that handles background messages and forwards them to the backend. 
* app_n.py — Flask backend exposing /store-notification to save notifications into a MySQL database. 
* send_notification.py — Python script using the Firebase Admin SDK to send a test notification to a target FCM token. 
* Push-Notification-Analysis.ipynb — (analysis notebook included in the repo).


 # Features

* Request browser notification permission and retrieve FCM token. 

* Receive and display foreground & background notifications.

* Store incoming notification metadata in a MySQL database via Flask. 

* Send test notifications with Firebase Admin from Python.

# Prerequisites

* Python 3.8+ installed.

* A Firebase project with Cloud Messaging enabled. You will need:

* firebaseConfig object for client (web).


# Configure Firebase (web client + service worker)

In the Firebase Console → Project settings → Web apps → copy the firebaseConfig object and paste it into index.html and firebase-messaging-sw.js where a comment says "Replace with your Firebase configuration". Both files must share the same configuration so the SW and page initialize the same Firebase app.

Example firebaseConfig snippet (replace the placeholders):


# Prepare service account (Admin SDK)

Download a Firebase service account JSON from Firebase Console → Project Settings → Service accounts.

Move it into the repo (recommended: do not commit it). Update send_notification.py:

      cred = credentials.Certificate("path/to/serviceAccountKey.json")


Put the target device/browser FCM token into fcm_token (or modify script to accept a token via CLI/env).
* Firebase service account JSON key for the Admin SDK.

* MySQL server and credentials (or MariaDB).


# Create MySQL database & notifications table

Log into MySQL and run:

     CREATE DATABASE notifications_db;
     USE notifications_db;

     CREATE TABLE notifications (
     id INT AUTO_INCREMENT PRIMARY KEY,
     title VARCHAR(255) NOT NULL,
     body TEXT NOT NULL,
     type VARCHAR(32),
     source VARCHAR(100),
     received_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
     UNIQUE KEY uniq_content (title(255), body(255), source) -- optional, used for ON DUPLICATE KEY
     );


  # Update Flask backend config

Edit the DB connection settings inside app_n.py to match your MySQL host, user, password and database. Then run:

     python app_n.py
     Flask will start at http://127.0.0.1:5000 (default)


  # Get an FCM token from the browser

Open the served index.html in your browser.

Click Enable Notifications. The page requests permission and then prints/alerts an FCM token (and logs it to console). Save that token. 

index

# Send a test notification from Python

Add the saved token to send_notification.py (variable fcm_token) and make sure your service account path is set. Then run:

     python send_notification.py


The script sends a notification to the token via the Firebase Admin SDK. Verify you receive it in the browser (foreground or background, depending on focus). 

send_notification

# Verify storage

The browser and SW both POST notification data to your Flask backend which should insert rows into the notifications table. Check DB contents with:

     SELECT * FROM notifications ORDER BY received_at DESC LIMIT 20;


The Flask endpoint storing notifications is defined in app_n.py.

* A browser that supports Service Workers (and HTTPS or localhost).

Note: Service Workers require HTTPS when served from a remote host. localhost is allowed for development.
