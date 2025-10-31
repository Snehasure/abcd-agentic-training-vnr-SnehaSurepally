# 📅 Automated Meeting Reminder System (Gmail + Twilio + Google Sheets)

## 🧠 Overview

This project automates the process of detecting upcoming meetings from Gmail, storing them into Google Sheets, and sending personalized WhatsApp/SMS using Twilio — all inside an n8n workflow.

The workflow runs on a defined schedule, reads your meeting invites, extracts date/time and link information, and ensures you never miss an important event.

## 🏷️ Project Title

Meeting Reminder Automation using Gmail, Google Sheets, and Twilio in n8n

## 💡 Problem Statement

Manually tracking meeting emails and sending reminders is time-consuming and error-prone.
Professionals often miss meetings due to notification delays or cluttered inboxes.

Goal: Create an end-to-end automated reminder system that:

Reads meeting details from Gmail automatically

Saves meetings to a central database (Google Sheets)

Sends WhatsApp  reminders before the meeting

## 🎯 Objectives

Automatically extract upcoming meeting details (title, date, link).

Store meetings in Google Sheets for centralized tracking.

Send automatic WhatsApp  reminders before each meeting.

Schedule the workflow to run periodically.

## Tools and Technologies Used

<img width="462" height="247" alt="image" src="https://github.com/user-attachments/assets/bbdc4549-8bfd-4ea2-9f42-a4584ff34cec" />

## High-Level Architecture Diagram

flowchart TD

  A[Gmail Inbox] --> B[Gmail Node - Fetch Emails]
    
  B --> C[Code Node - Extract Meeting Details]
    
   C --> D[Google Sheets Node - Save Data]
   
   D --> E{Meeting within 24 Hours?}
  
  E -- Yes --> F[Twilio Node - Send WhatsApp Reminder]

 <img width="467" height="270" alt="image" src="https://github.com/user-attachments/assets/829a6058-4ce5-49bd-8ee1-a86cde7c56d9" />

## ⚙️ Pre-Workflow Setup — API and Environment Configuration

Before building and running the Automated WhatsApp Meeting Reminder workflow in n8n, a few important configurations and API setups must be completed. These ensure that every service — Gmail, Google Sheets, and Twilio (for WhatsApp) — can securely connect and exchange data.

1. Set Up Google Cloud Project for Gmail and Google Sheets APIs

Both Gmail and Google Sheets require authorization from Google Cloud.

Follow these steps carefully:

🧩 Step 1: Create Google Cloud Project

Go to Google Cloud Console


Click “New Project” → Name it Meeting Reminder Automation.

Save the project ID for later use.

🧩 Step 2: Enable Required APIs

Inside your new project, navigate to APIs & Services → Library, then enable:

Gmail API – for fetching meeting emails.

Google Sheets API – for storing and updating meeting details.

🧩 Step 3: Configure OAuth Consent Screen

Go to APIs & Services → OAuth consent screen.

Choose External, enter app details (name, support email, etc.).

Add your Gmail address under Test Users.

Save and continue.

🧩 Step 4: Create OAuth Client Credentials

Go to APIs & Services → Credentials → Create Credentials → OAuth Client ID.

Select Web application.

Add http://localhost:5678/rest/oauth2-credential/callback as an authorized redirect URI (this is n8n’s default callback URL).

Save it — copy your Client ID and Client Secret.

🧩 Step 5: Connect Gmail and Google Sheets to n8n

Open your n8n dashboard → Credentials.

Create:

Gmail OAuth2 credential using your Client ID and Secret.

Google Sheets OAuth2 credential with the same details.

Authorize both by logging into your Google Account when prompted.

2. Set Up Twilio WhatsApp API

This enables WhatsApp messages to be sent automatically.

🧩 Step 1: Create a Twilio Account

Go to https://www.twilio.com/

 and sign up.

Verify your mobile number.

In the Twilio Console, note down:

Account SID

Auth Token

WhatsApp Sandbox Number

🧩 Step 2: Join WhatsApp Sandbox

In your Twilio dashboard → Messaging → Try it Out → Send a WhatsApp message.

Send the provided code (e.g., join bright-dream) from your phone to the Twilio sandbox number using WhatsApp.

Once connected, you can send messages to yourself using Twilio’s WhatsApp API.

🧩 Step 3: Create Twilio Credential in n8n

In n8n → Credentials → Twilio, enter:

Account SID

Auth Token

Save and test the connection.

3. Install and Configure n8n Environment
   
🧩 Step 1: Local Setup (Optional)

If you are running locally:

npm install -g n8n

n8n start


Access the editor at http://localhost:5678
 
 ## Implementation details

 ### 🕒 Node 1 — Schedule Trigger

Purpose:

This node automatically starts the workflow every day at a specific time without manual intervention.

Configuration Details:

Trigger Type: Schedule Trigger

Trigger Interval: Daily

Days Between Triggers: 1

Trigger Time: 7:00 AM

Timezone: America/New_York (UTC-04:00)

Functionality:

This node ensures that the meeting reminder automation runs once a day at 7:00 AM.

Each execution generates a timestamp and readable date-time details (such as year, month, day, and time) which are passed to the next node in the workflow.

It serves as the starting point for fetching and processing upcoming meetings from Gmail.

### 📧 Node 2 — Gmail (Get Many Messages)

Purpose:

This node connects to Gmail and fetches all incoming meeting-related emails from your inbox. It acts as the data collection step of the automation.

Configuration Details:

Credential: Gmail Account 2

Resource: Message

Operation: Get Many

Return All: Disabled (Limited to 10 messages per run)

Simplify: Enabled

Filters Applied:

Label Names or IDs: INBOX

Search Query:

subject:(meeting) OR has:invite OR "Google Meet"


Read Status: Unread and Read emails (to include all meeting-related emails)

Functionality:

This node fetches the latest meeting invitations or reminders from the inbox.

It searches for emails that contain specific meeting-related keywords (like meeting, invite, or Google Meet).

The output includes essential metadata such as message IDs, snippets, MIME types, and labels.

The data is then passed to the next nodes for date extraction, storage in Google Sheets, and sending reminders.

### 🧠 Node 3 — Code in JavaScript (Extract Meeting Details)

Purpose:

This node parses the Gmail messages to extract meeting details, such as the subject, date, time, meeting links, and filters out irrelevant or outdated emails.

It ensures that only upcoming meetings within the next 24 hours are considered for reminders.

Configuration Details:

Mode: Run Once for All Items

Language: JavaScript

Core Functionality:

Extracts meeting-related emails by scanning both the subject and body for keywords like meeting, invite, schedule, zoom, Google Meet, or teams.

Parses date and time from email content using multiple regex patterns that support various formats (e.g., 17/10/2025, October 17, 2025, 2025-10-17, etc.).

Converts extracted date and time into a standard JavaScript Date object for comparison.

Filters meetings that are scheduled within the next 24 hours from the current time.

Extracts meeting links for Google Meet, Zoom, or Microsoft Teams using URL patterns.

Generates clean and structured output for each upcoming meeting.

If no valid meeting is found for the day, it returns:

{ "message": "No upcoming meetings found today." }

Sample Code Snippet:

const results = [];

function parseDateTime(dateStr, timeStr) {

  if (!dateStr) return null;
  
  let full = dateStr;
  
  if (timeStr && timeStr !== "Not Found") full += " " + timeStr;
  
  let parsed = new Date(full);
  
  if (isNaN(parsed)) {
  
  parsed = new Date(Date.parse(full.replace(/(\d+)(st|nd|rd|th)/, "$1")));
    
  }
  
  return isNaN(parsed) ? null : parsed;
  
}

// Extract meetings from Gmail messages

for (const item of items) {

  const json = item.json;
  
  // ... (extract subject, body, links, and parse date/time)
  
}

Example Output (When meetings found):

{

  "title": "Project Review Call",
  
  "date": "October 31, 2025",
  
  "time": "11:15 AM",
  
  "meetingDate": "2025-10-31T11:15:00.000Z",
  
  "meetingLink": "https://meet.google.com/abc-defg-hij",
  
  "body": "Discussion on upcoming sprint planning...",
  
  "messageId": "199f093549123532"
  
}

Example Output (When no meetings):

{

  "message": "No upcoming meetings found today."
  
}

## 📊 Node 4 — Google Sheets (Append Meeting Logs)

Purpose:

This node records all processed meeting information into a Google Sheet called MeetingLogs.

It helps you maintain a historical record of all upcoming meetings or logs a “no meetings found” message when applicable.

Configuration:

Credential: Connected with Google Sheets account

Resource: Sheet Within Document

Operation: Append Row

Document: MeetingLogs

Sheet: Sheet1

Mapping Column Mode: Map Automatically

Functionality:

Takes structured JSON output from the previous JavaScript Code node.

Automatically maps fields like:

title

date

time

meetingLink

message (if no meetings found)

Appends each new entry as a new row in the selected Google Sheet.

Ensures duplicate entries are avoided in subsequent executions (by using message IDs or unique timestamps).

## 🧩 Node 5 — Code in JavaScript (Filter & Prepare Meeting Reminders)

Purpose:

This node reads the meeting data fetched from Google Sheets and filters upcoming meetings within the next 24 hours (1440 minutes).

It then prepares structured reminder data that will be used to send WhatsApp or email notifications.

Configuration:

Language: JavaScript

Execution: Run Once for All Items

Output: A list of upcoming meetings ready for reminder delivery

Code Explanation:

const now = new Date();

const reminders = [];

for (const item of items) {

  const row = item.json;

  // Identify the date field (meetingDate / startTime / date)
  
  const meetingTime = new Date(row.meetingDate || row.startTime || row.date);

  if (isNaN(meetingTime)) continue;

  // Calculate time difference in minutes
  
  const diffMinutes = (meetingTime - now) / (1000 * 60);

  // Select meetings happening within next 24 hours (1440 min)
  
  if (diffMinutes <= 1440 && diffMinutes > 0) {
  
  reminders.push({
  
  json: {
  
  title: row.title || row.subject || "Upcoming Meeting",
  
  meetingTime: meetingTime.toLocaleString(),
  
  meetingLink: row.meetingLink || row.link || "",
  
  email: row.email || "snehasurapally@gmail.com",
  
  phone: row.phone || "+91xxxxxxxxxx",
  
  }
  
  });
  
  }
  
}

return reminders;


Logic:

Reads meeting rows from the Google Sheet (via previous node).

Converts the date or start time into a valid Date object.

Calculates time difference between now and the meeting time.

Keeps only those meetings that start within the next 24 hours.

Creates an array (reminders) with key meeting details:

title — meeting name or subject

meetingTime — formatted date and time

meetingLink — URL to join the meeting

email — recipient email for reminders

phone — WhatsApp number for notification

Example Output:

[

  {
  
  "title": "Team Standup",
  
  "meetingTime": "10/31/2025, 10:00:00 AM",
  
  "meetingLink": "https://meet.google.com/xyz-1234",
  
  "email": "snehasurapally@gmail.com",
  
  "phone": "+91xxxxxxxxxx"
  
}

]

Outcome:

This node acts as a bridge between data and reminders — transforming stored meeting data into ready-to-send reminder payloads.

The output feeds directly into your WhatsApp and Gmail reminder nodes, ensuring timely notifications for all scheduled meetings.

## 📱 Node 6 — Send WhatsApp Reminder via Twilio

Purpose:

This node sends automated WhatsApp meeting reminders using the Twilio API.

It takes the filtered meeting details (from the JavaScript node) and sends a personalized WhatsApp message to the recipient’s phone number.

Configuration:

Credential: Twilio account

Resource: SMS

Operation: Send

From: Your registered Twilio WhatsApp number (+14155238886)

To: Dynamic phone number from Google Sheet (+91XXXXXXXXXX)

To WhatsApp: ✅ Enabled

Message Body:

⏰ Reminder: Your meeting "{{ $json["title"] }}" 

## 🏁 Final Conclusion

The Automated Meeting Reminder System successfully demonstrates the power of agentic automation using n8n with seamless integrations between Gmail, Google Sheets, and Twilio WhatsApp API.

Through a well-structured, autonomous workflow, the system:

Detects upcoming meetings from Gmail automatically.

Extracts essential details like meeting title, date, and link through intelligent parsing.

Stores and tracks all meeting data in Google Sheets for transparency and monitoring.

Sends timely WhatsApp reminders to ensure users never miss important meetings.


This project effectively combines low-code automation with API-based communication, showcasing real-world integration of productivity tools. It minimizes manual effort, ensures consistency, and enhances personal and team productivity.

In summary, the project achieves its main objective —

To build a fully automated, reliable, and intelligent meeting reminder system using n8n that leverages WhatsApp for instant and efficient notifications.

The implementation can easily be extended in the future to include:

Email and calendar synchronization,

Multi-user reminder support, and

Integration with other communication platforms like Slack or Telegram.

This system stands as a practical example of how AI-driven automation and workflow orchestration can transform repetitive digital tasks into efficient, autonomous processes.

    

