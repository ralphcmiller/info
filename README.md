#!/bin/bash

# Log file path
LOG_FILE="$HOME/Downloads/bot_runtime.log"

# Check for the --run-now flag
RUN_NOW=false
if [[ "$1" == "--run-now" ]]; then
  RUN_NOW=true
fi

while true; do
  # Define time boundaries
  MORNING_HOUR=8     # Earliest start time (8:00 AM)
  AFTERNOON_HOUR=12  # Latest start time (12:00 PM)
  NIGHT_HOUR=22      # Latest possible stop time (10:00 PM)
  MAX_RUNTIME=36000  # Maximum runtime (10 hours in seconds)

  # Get the current date
  CURRENT_DATE=$(date '+%Y-%m-%d')

  if [ "$RUN_NOW" == true ]; then
    # Skip delay and start immediately
    START_TIME=$(date '+%s') # Current time as epoch seconds
    STOP_TIME=$((START_TIME + MAX_RUNTIME))
    # Ensure stop time does not exceed 10 PM
    NIGHT_EPOCH=$(date -d "$CURRENT_DATE $NIGHT_HOUR:00" '+%s')
    if [ "$STOP_TIME" -gt "$NIGHT_EPOCH" ]; then
      STOP_TIME=$NIGHT_EPOCH
    fi
  else
    # Calculate random start time between 8 AM and 12 PM
    START_HOUR=$((RANDOM % (AFTERNOON_HOUR - MORNING_HOUR + 1) + MORNING_HOUR))
    START_MIN=$((RANDOM % 60))
    START_TIME=$(date -d "$CURRENT_DATE $START_HOUR:$START_MIN" '+%s') # Start time as epoch seconds

    # Calculate random runtime (up to 10 hours)
    RUNTIME=$((RANDOM % MAX_RUNTIME + 3600)) # At least 1 hour, up to 10 hours
    STOP_TIME=$((START_TIME + RUNTIME))

    # Ensure stop time does not exceed 10 PM
    NIGHT_EPOCH=$(date -d "$CURRENT_DATE $NIGHT_HOUR:00" '+%s')
    if [ "$STOP_TIME" -gt "$NIGHT_EPOCH" ]; then
      STOP_TIME=$NIGHT_EPOCH
    fi
  fi

  # Calculate sleep durations
  CURRENT_TIME=$(date '+%s') # Current time as epoch seconds
  SLEEP_BEFORE_START=$((START_TIME - CURRENT_TIME))
  RUNTIME=$((STOP_TIME - START_TIME))

  # Skip to the next day if the calculated start time is in the past
  if [ "$SLEEP_BEFORE_START" -lt 0 ] && [ "$RUN_NOW" == false ]; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Calculated start time is in the past. Waiting until tomorrow..." >> "$LOG_FILE"
    sleep $((24 * 60 * 60 - (CURRENT_TIME - $(date -d "$CURRENT_DATE" '+%s'))))
    continue
  fi

  # Log start and stop times
  echo "========================" >> "$LOG_FILE"
  echo "Start Time: $(date -d @$START_TIME '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
  echo "Stop Time: $(date -d @$STOP_TIME '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
  echo "Total Runtime: $((RUNTIME / 3600)) hours $(((RUNTIME % 3600) / 60)) minutes" >> "$LOG_FILE"

  # Sleep until the start time (if not running immediately)
  if [ "$RUN_NOW" == false ]; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Waiting until start time..." >> "$LOG_FILE"
    sleep "$SLEEP_BEFORE_START"
  fi

  # Start the bot
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting bot..." >> "$LOG_FILE"
  java -Xmx255M -jar $HOME/Downloads/DBLauncher.jar -script 'P2P Master AI' -world world -username dbname -password dbpass -account zezima -params default >> "$LOG_FILE" 2>&1 &

  # Run for the allotted runtime
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Bot running for $RUNTIME seconds..." >> "$LOG_FILE"
  sleep "$RUNTIME"

  # Stop the bot and clean up
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Stopping bot..." >> "$LOG_FILE"
  killall java

  # Reset RUN_NOW to false after the first cycle
  RUN_NOW=false

  # Wait until tomorrow to calculate the next cycle
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Cycle complete. Waiting until tomorrow..." >> "$LOG_FILE"
  sleep $((24 * 60 * 60 - (CURRENT_TIME - $(date -d "$CURRENT_DATE" '+%s'))))
done
