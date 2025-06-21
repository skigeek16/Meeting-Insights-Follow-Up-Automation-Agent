# Meeting Insights & Follow-Up Automation Agent

This repository contains the n8n workflow for an AI-powered agent designed to automate the extraction of key insights from meeting recordings or transcripts, generate actionable follow-ups, and integrate with essential productivity tools. It transforms raw meeting data into organized, actionable intelligence, significantly reducing manual effort and improving meeting efficiency.

## Table of Contents

* [Features](#features)
* [How It Works](#how-it-works)
* [Setup and Configuration](#setup-and-configuration)
    * [Prerequisites](#prerequisites)
    * [n8n Workflow Setup](#n8n-workflow-setup)
    * [Specific Node Configurations](#specific-node-configurations)
* [Use Cases](#use-cases)
* [Advantages](#advantages)
* [Future Enhancements](#future-enhancements)

## Features

  * **Automated Transcription:** Converts meeting audio recordings into accurate text transcripts using AI.
  * **Intelligent Summarization:** Condenses lengthy transcripts into concise summaries of key discussions and decisions.
  * **Personalized To-Do Lists:** Identifies individual action items and creates customized task lists for each participant.
  * **Automated Calendar Reminders:** Integrates with Google Calendar to set reminders for deadlines and follow-up events.
  * **Sentiment Analysis:** Analyzes the overall emotional tone of the meeting discussions.
  * **Centralized Documentation:** Automatically compiles all insights into organized, accessible Google Docs.
  * **Flexible Input Methods:** Supports audio files from Google Drive or direct text/audio via webhooks.

## How It Works

The workflow operates in a sequential, automated manner to process meeting data:

1.  **Input & Trigger:**
      * The workflow can be initiated by either a **Google Drive Trigger** detecting a new audio file upload in a specified folder, or by a **Webhook** receiving pre-transcribed text or raw audio data.
2.  **Audio Processing (if applicable):**
      * If triggered by a Google Drive audio file, the workflow first uses a **Google Drive (download\_file) node** to retrieve the audio content.
      * This audio is then sent to an **OpenAI (Transcribe recording) node** which leverages powerful AI models to convert the spoken content into a detailed text transcript.
3.  **Core Information Extraction:**
      * The complete meeting transcript (from OpenAI or direct Webhook input) is then forwarded to an external summarization service via an **HTTP Request3 node**. This service processes the text and returns a concise summary of the meeting's key points.
4.  **Intelligent AI Agent Processing:**
      * The summarized content is fed into the central **AI Agent**, powered by an **Azure OpenAI Chat Model**. This intelligent agent analyzes the summary to:
          * Identify specific action items and deliverables.
          * Assign these tasks to relevant participants.
          * Detect any mentioned deadlines or follow-up events.
          * Prepare structured output for documentation and calendar integration.
5.  **Documentation & Follow-up Integration:**
      * Leveraging its integrated tools, the AI Agent interacts with **Google Calendar** to automatically create event reminders for identified deadlines.
      * The generated summary, along with personalized to-do lists, is then prepared for documentation.
      * Concurrently, the meeting summary is also sent to another **OpenAI Message Model** for sentiment analysis, which assesses the overall emotional tone of the meeting.
      * Finally, all compiled insights (summary, to-dos, sentiment analysis) are automatically updated and stored in a designated **Google Doc**, creating a comprehensive and accessible record.

## Setup and Configuration

### Prerequisites

  * An active n8n instance.
  * API keys/credentials for:
      * OpenAI (for transcription and sentiment analysis).
      * Azure OpenAI (for the main AI Agent).
      * Access to your external summarization service (for HTTP Request3).
  * Google Cloud Project with APIs enabled for:
      * Google Drive
      * Google Calendar
      * Google Docs
      * Ensure appropriate OAuth credentials are set up in n8n for these Google services.

### n8n Workflow Setup

1.  Import the provided `.json` n8n workflow file into your n8n instance.
2.  Connect all necessary credentials for OpenAI, Azure OpenAI, Google Drive, Google Calendar, and Google Docs within n8n.
3.  Configure the specific settings for each node as detailed below.

### Specific Node Configurations

  * **Webhook Nodes (Webhook1, Webhook):**
      * Configure the desired HTTP method (POST is typical for receiving data).
      * **For direct audio file transcription (if sending raw audio via webhook):** Ensure your OpenAI (Transcribe recording) node is configured to access the audio data from `{{ $json.body.body }}` or the appropriate path where your webhook receives the raw audio binary content.
  * **Google Drive Trigger:**
      * Specify the Google Drive folder where meeting audio recordings will be uploaded.
  * **OpenAI (Transcribe recording):**
      * Select your OpenAI credentials.
      * Ensure the "File" field points to the output of the Google Drive `download_file` node, or the `{{ $json.body.body }}` (or equivalent) for webhook audio input.
  * **HTTP Request3 (Summarizer):**
      * **URL:** Set this to the endpoint of your external summarization service (e.g., `https://loggerwalkkartix...`).
      * **Method:** Typically `POST`.
      * **Body Parameters:** The body of your request should contain the transcribed text. Assuming the OpenAI transcription node outputs the text in a field like `data.text`, you would typically send it as JSON:
        ```json
        {
          "text": "{{ $json.data.text }}"
        }
        ```
        *As per custom instruction, ensure your summarizer expects the transcript in a `text` field within the JSON body if you are sending the output of the transcription node to it.*
  * **AI Agent:**
      * **Model:** Select your Azure OpenAI Chat Model credentials.
      * **Memory:** Keep as Simple Memory for initial setup.
      * **Tools:** Verify Google Calendar and Google Docs tools are correctly configured with your credentials.
      * **Agent Prompt:** This is critical. Craft a detailed prompt that instructs the AI Agent on how to identify action items, assign them, detect deadlines, and format the output for Google Docs and Google Calendar. Provide examples within the prompt for best results.
  * **Google Docs (update: document):**
      * Configure whether to create a new document or update an existing one. If updating, provide the Document ID.
      * Map the output from the AI Agent (summary, to-do lists) and the sentiment analysis (from the second OpenAI node) to the appropriate fields for your Google Doc content. Use n8n expressions to combine these data points into a well-formatted document.

## Use Cases

  * **Automated Meeting Summaries:** Instantly generate concise overviews from audio recordings.
  * **Personalized To-Do Lists:** Automatically identify and distribute specific tasks to each participant.
  * **Proactive Calendar Reminders:** Create Google Calendar events for deadlines extracted from discussions.
  * **Sentiment Analysis:** Gain insights into the overall tone and mood of meetings.
  * **Centralized Meeting Records:** Maintain an organized Google Doc with all meeting outcomes.
  * **Enhanced Team Collaboration:** Ensure team members are aligned on next steps.
  * **Streamlined Project Management:** Accelerate project cycles by automating documentation.
  * **Efficient Onboarding & Catch-up:** Quickly bring new team members or absentees up-to-speed.
  * **Client & Stakeholder Management:** Precisely document client needs and agreements.

## Advantages

  * **Elevates Efficiency & Saves Valuable Time:** Automatically summarizes discussions and generates tasks, eliminating manual note-taking and speeding up post-meeting follow-up.
  * **Drives Accountability & Enhances Collaboration:** Delivers personalized to-do lists and sets calendar reminders, ensuring everyone knows their responsibilities and improving task completion rates.
  * **Unlocks Deeper Insights & Supports Strategic Decisions:** Provides objective sentiment analysis and quick access to historical meeting data, aiding in better-informed strategic choices.
  * **Establishes Centralized, Reliable Records:** Automatically compiles all meeting outcomes (summaries, tasks, sentiment) into easily accessible and searchable Google Docs, ensuring consistency and accuracy.

## Future Enhancements

  * **Q\&A Capability via Vector Database:** Integrate a Vector Database (e.g., Pinecone, Weaviate) to store meeting data, enabling users to ask natural language questions about past meetings and receive precise answers (Retrieval Augmented Generation).
  * **Pre-Meeting Preparation Workflow:** Implement a scheduled workflow to analyze past meetings related to an upcoming agenda, generating a concise brief for participants to ensure optimal preparation.
  * **Advanced Human-in-the-Loop:** For complex summaries, implement a robust "Human-in-the-Loop" review process within the workflow to ensure maximum accuracy and quality control.
  * **Direct Google Meet API Integration:** Fully automate workflow initiation by directly integrating with the Google Meet API via OAuth to pull live transcripts as they are generated, removing any manual upload steps.
  * **Outbound Communication & Proactive Reminders:** Explore integration with external communication APIs (e.g., Twilio) to send personalized pre-meeting call/SMS reminders or post-meeting follow-ups directly to participants.
