#!/bin/bash

# Log file path
LOG_FILE="$HOME/Downloads/bot_runtime.log"

# Generate a random start time between 8 AM and 12 PM
START_HOUR=$((RANDOM % 4 + 8))  # Random hour between 8 AM and 12 PM
START_MIN=$((RANDOM % 60))      # Random minute

# Random runtime duration between 8 to 12 hours
MAX_RUNTIME=$((22 * 60))        # 10 PM in minutes
START_MINUTES=$((START_HOUR * 60 + START_MIN))
RUNTIME_MIN=$((RANDOM % 241 + 480))  # Random duration (8-12 hours in minutes)
STOP_MINUTES=$((START_MINUTES + RUNTIME_MIN))

# If the stop time hits 10 PM, add 1-30 random minutes
if [ "$STOP_MINUTES" -ge "$MAX_RUNTIME" ]; then
  EXTRA_MIN=$((RANDOM % 30 + 1))  # Randomly add 1-30 minutes
  STOP_MINUTES=$((MAX_RUNTIME + EXTRA_MIN))
fi

STOP_HOUR=$((STOP_MINUTES / 60))
STOP_MIN=$((STOP_MINUTES % 60))

# Format start and stop times
START_TIME=$(printf "%02d:%02d" "$START_HOUR" "$START_MIN")
STOP_TIME=$(printf "%02d:%02d" "$STOP_HOUR" "$STOP_MIN")

# Calculate total runtime in hours and minutes
TOTAL_RUNTIME_HOURS=$((RUNTIME_MIN / 60))
TOTAL_RUNTIME_MINUTES=$((RUNTIME_MIN % 60))
TOTAL_RUNTIME=$(printf "%02d:%02d" "$TOTAL_RUNTIME_HOURS" "$TOTAL_RUNTIME_MINUTES")

# Log start and stop times, and total runtime
echo "========================" >> "$LOG_FILE"
echo "Date: $(date '+%Y-%m-%d')" >> "$LOG_FILE"
echo "Start Time: $START_TIME" >> "$LOG_FILE"
echo "Stop Time: $STOP_TIME" >> "$LOG_FILE"
echo "Total Runtime: $TOTAL_RUNTIME" >> "$LOG_FILE"

# Wait until the current time is at or greater than START_TIME
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Waiting until $START_TIME to start the bot..." >> "$LOG_FILE"
while [[ "$(date +%H:%M)" < "$START_TIME" ]]; do
  sleep 30  # Check every 30 seconds
done

# Start the bot and log runtime
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting bot..." >> "$LOG_FILE"
java -Xmx255M -jar $HOME/Downloads/DBLauncher.jar -script 'P2P Master AI' -world world -username dbname -password dbpass -account zezima -params default >> "$LOG_FILE" 2>&1 &

# Schedule the bot stop
STOP_COMMAND="pkill -f $HOME/Downloads/DBLauncher.jar"
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Scheduling bot stop at $STOP_TIME..." >> "$LOG_FILE"
echo "$STOP_COMMAND" | at "$STOP_TIME"
