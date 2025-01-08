# Mobile Game Analysis Real-Time Pipeline with Pub/Sub and Dataflow

This project demonstrates how to build a real-time analytics pipeline for mobile game data using Google Cloud Pub/Sub and Apache Beam (Dataflow). The pipeline processes game events, analyzes player and team scores, and calculates weapon performance in battles, all in real time.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Pipeline Process](#pipeline-process)
  - [1. Upload Source File](#1-upload-source-file)
  - [2. Set Up Pub/Sub](#2-set-up-pubsub)
  - [3. Run `publish.py`](#3-run-publishpy)
  - [4. Set Up Dataflow](#4-set-up-dataflow)
  - [5. Analyze Player and Team Scores](#5-analyze-player-and-team-scores)
  - [6. Analyze Weapon Performance](#6-analyze-weapon-performance)
- [Windowing, Trigger, and Watermark Explained](#windowing-trigger-and-watermark-explained)
  - [Windowing](#windowing)
    - [Tumbling Window](#tumbling-window)
    - [Sliding Window](#sliding-window)
    - [Session Window](#session-window)
    - [Global Window](#global-window)
  - [Trigger](#trigger)
  - [Watermark](#watermark)
- [License](#license)

---

## Overview
This pipeline enables real-time analysis of mobile game data by leveraging Pub/Sub for data ingestion and Dataflow for streaming processing. Key features include:
- Analyzing player and team scores.
- Calculating weapon performance metrics.
- Writing results to Pub/Sub topics for further use.

## Prerequisites
- Google Cloud account.
- GCP SDK installed.
- `gcloud` CLI configured.
- Python 3.x installed locally.

## Pipeline Process

### 1. Upload Source File
Upload the source file `mobile_game.txt` to a Google Cloud Storage bucket:
```bash
gsutil cp mobile_game.txt gs://<your-bucket-name>/
```

### 2. Set Up Pub/Sub
1. Enable the Pub/Sub API:
   ```bash
   gcloud services enable pubsub.googleapis.com
   ```
2. Create a topic:
   ```bash
   gcloud pubsub topics create mobile-game-events
   ```
3. Create a subscription:
   ```bash
   gcloud pubsub subscriptions create mobile-game-subscription --topic=mobile-game-events
   ```

### 3. Run `publish.py`
Run the `publish.py` script to publish events from the `mobile_game.txt` file to the Pub/Sub topic.

### 4. Set Up Dataflow
Enable the Dataflow API:
```bash
gcloud services enable dataflow.googleapis.com
```

### 5. Analyze Player and Team Scores
Run the `score.py` script to process player and team scores:
```bash
python score.py
```
The script reads events from Pub/Sub, calculates scores using global windows with custom triggers, and writes results back to a Pub/Sub topic.

### 6. Analyze Weapon Performance
Run the `weapon.py` script to calculate weapon performance:
```bash
python weapon.py
```
The script reads battle data from Pub/Sub, calculates average battle points using session windows, and writes results back to a Pub/Sub topic.

## Windowing, Trigger, and Watermark Explained

### Windowing
Windowing allows data to be grouped based on temporal bounds. Here are the commonly used window types:

#### Tumbling Window
- Fixed-size, non-overlapping windows.
- Each event belongs to exactly one window.
- Example: A 1-minute tumbling window processes events from `00:00` to `00:01`.

#### Sliding Window
- Fixed-size windows that overlap.
- Each event can belong to multiple windows.
- Example: A 1-minute window with a 30-second slide processes events in overlapping intervals (`00:00-00:01`, `00:00:30-00:01:30`, etc.).

#### Session Window
- Captures events that occur within a defined gap duration.
- New events extend the session if they arrive within the gap.
- Example: A session window with a 5-minute gap groups events until there is a 5-minute period of inactivity.

#### Global Window
- Processes all data in a single, unbounded window.
- Useful for continuous aggregations with custom triggers.

### Trigger
Triggers define when results are emitted for a window. Common triggers include:
1. **Event-time trigger (default):** Emits results based on the event timestamp and watermark.
2. **Processing-time trigger:** Emits results based on the system clock.
3. **AfterWatermark:** Emits results when the watermark passes the end of the window.
4. **AfterCount:** Emits results after a specified number of elements arrive.
5. **Repeatedly:** Re-applies a trigger after the initial trigger fires.

### Watermark
A watermark tracks the progress of event time in a pipeline. It defines the point in time at which all earlier events are expected to have arrived. Late events can be handled using:
- **Allowed lateness:** Specifies how late events are accepted.
- **Dropping or updating results:** Determines how late events are treated.

## License
This project is licensed under the MIT License. See the LICENSE file for details.

