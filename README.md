#!/bin/bash

# Log file path
LOG_FILE="$HOME/Downloads/bot_runtime.log"

while true; do
  # Testing settings: Quick start and short runtime
  START_DELAY=$((RANDOM % 60))   # Random delay (0-60 seconds) before starting
  RUNTIME=$((RANDOM % 120 + 30)) # Random runtime (30-150 seconds)

  # Get the current time and calculate start/stop times
  CURRENT_TIME=$(date '+%s') # Current time as epoch seconds
  START_TIME=$(date -d "+$START_DELAY seconds" '+%s')
  STOP_TIME=$(($START_TIME + $RUNTIME))

  # Log start and stop times
  echo "========================" >> "$LOG_FILE"
  echo "Start Time: $(date -d @$START_TIME '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
  echo "Stop Time: $(date -d @$STOP_TIME '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
  echo "Total Runtime: $((RUNTIME / 60)) minutes $((RUNTIME % 60)) seconds" >> "$LOG_FILE"

  # Sleep until the start time
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Waiting $START_DELAY seconds until start time..." >> "$LOG_FILE"
  sleep "$START_DELAY"

  # Start the bot
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting bot for testing..." >> "$LOG_FILE"
  Xvfb :99 -screen 0 1024x768x24 &
  XVFB_PID=$!
  export DISPLAY=:99
  java -Xmx255M -jar $HOME/Downloads/DBLauncher.jar -script 'P2P Master AI' -world world -username dbname -password dbpass -account zezima -params default >> "$LOG_FILE" 2>&1 &
  JAVA_PID=$!

  # Run for the allotted runtime
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Bot running for $RUNTIME seconds..." >> "$LOG_FILE"
  sleep "$RUNTIME"

  # Stop the bot and clean up
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Stopping bot..." >> "$LOG_FILE"
  kill "$JAVA_PID" 2>/dev/null
  kill "$XVFB_PID" 2>/dev/null

  # End the loop for testing
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Testing complete. Exiting script..." >> "$LOG_FILE"
  break
done
