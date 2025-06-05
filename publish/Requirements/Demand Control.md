## Requirements

### 1st Prompt: 

Create for me a comprehensive prompt to ask a LLM like Claude to make a demand control  lambda code as well as the database tables (in Postgresql) needed to implement the Demand Control Check system and lambda to work. The lambda will run in AWS Lambda and the details of the lambda are as follows: 

Overview and purpose
Demand Control is used to check if a device is exceeding the set threshold for the check. It will first receive all the demand control schedules that contains the device id and the set threshold for the device. Then it executes the 6 check conditions accordingly while following the process flow to see if the device triggers any of the check conditions and responds accordingly. The check is every 30 minute block, it should also account for the marks +20 minute after the 30th minute mark which is [50, 51, 52, 53, 54, 55,  60(0 minute in the next hour)]. Calculate the difference and give the correct data to the child lambda to calculate for the checks (Ex Check condition 1 should check the data at 20th minute and 50th minute mark (50-30=20))


Demand Control Logic:
1. Demand Calculation:
- 30minute block = [kWh (end of 30min) - kWh (start of 30minute block)] x 2
- 1minute block = [kWh (end of 1min) - kWh (start of 1min)] x 2
2. Demand Target = x kW (30minutes value)
3. Demand Alert Threshold = 2/3 x = y	(20minutes value)
4. Demand per minute = x/30 = z (1minute value)

Check Conditions:

1. Check demand at 20 minutes, demand value = a:  
- Check: If a > y, automatic alert is sent to customer requesting them to reduce their connected load by "a1" kW
- Calculate: a1 = (a-y)/2 - 7.5 

2. Check demand at 21 minutes, demand value = b: 
- Check: If c > y+2z, automatic alert is sent to customer requesting them to reduce their connected load by c1 kW
- Calculate: b1 = (b-(y+z))/2 * 8.57

3. Check demand at 22 minutes, demand value = c: 
- Check: If c > y+2z, automatic alert is sent to customer requesting them to reduce their connected load by c1 kW
- Calculate: c1 = (c-(y+2z))/2 - 10

4. Check demand at 23 minutes, demand value = d: 
- Check: If d > y+3z, automatic alert is sent to customer requesting them to reduce their connected load by d1 kW
- Calculate: d1 = (d-(y+3z))/2 - 12

5. Check demand at 24 minutes, demand value = e: 
- Check: If e > y+4z, automatic alert is sent to customer requesting them to reduce their connected load by e1 kW
- Calculate: e1 = (e-(y+4z))/2 - 15

6. Check demand at 25 minutes, demand value = f: 
- Check: If f > y+5z, automatic alert is sent to customer requesting them to reduce their connected load by f1 kW
- Calculate: f1 = (f-(y+5z))/2 - 20


Check Condition Process flow: 

Execute Check Conditions in order 1 to 6 with the appropriate time for each event. 

If Check Condition 1 is true, send an alert to the recipient using the notification api immediately and skip Check Condition 2-4,6 and execute Check Condition 5 

If Check Condition 2 is true, skip Check Condition 2-5 and execute Check Condition 6

If any Check Condition from Check Condition 3 and onwards is true, skip the following Check Condition 

If there is any Check Condition that is true; at 30minutes send an alert to the recipient using the notification api notifying them of final demand for the period and that new demand block has started

Lambda Implementation:

Using chalice and python code, create the Demand Control Lambda that is able to execute the Check Conditions for each Demand Control Schedule in the database and follow the process flow above to execute the necessary steps. This implementation must also make a master lambda and child lambda that can be created and executed from the master lambda. The master lambda will handle getting all the Demand Control Schedule, executing the child lambdas for each schedule, getting the result from the execution from each child lambda, responding accordingly and saves the results into the database. The child lambda handles the Check Conditions for each Demand Control Schedule given to it and follow the process flow for the Check Conditions above accordingly and also notifies the user using the notification api url provided below. 

The notification api url used to notify the recipient is https://notificationapi.com/api/notification with the required json:
{
    "client_id": "Insert client id here",
    "recipient_ids": [
        "Insert recipient id here"
    ],
    "subject": "Insert subject here",
    "message": "Insert message here",
    "notification_category": "DC"
}

Make sure to fill up the relevant "Insert xxxx here" respectively where xxxx means client id, recipient id, subject or message when using the api.

Also an important note is that the database has a latency of 3 minutes for the reading from the device to be inserted into the database. To account for this, make sure to run every check 4 minutes after the minute mark (Ex: Run the Check for 20th minute mark at 24 minute mark) so that the value will be in the database and not have missing values. Also give the lambda the ability to increase or decrease this latency value for future scaling.

### 2nd Prompt:

You are a software architect and AWS expert. You are tasked with developing a Demand Control Check System using AWS Lambda with Python (via the Chalice framework) and PostgreSQL. Please deliver:

1. PostgreSQL schema only create Tables and relationships for:
  - analytics_data.demand_control_schedules with columns (schedule_id, client_id, device_id, demand_target, recipient_ids, timezone)
  - analytics_data.demand_control_history with columns (history_id, schedule_id, device_id, minute, condition_triggered, demand_value, reduction_amount, timezone, alert_sent)

2. A Chalice-based AWS Lambda architecture, consisting of:

   - A Master Lambda that:

     - Runs every minute and keeps track of the processes inside the 30 minute demand block
     - Queries all active Demand Control Schedules
     - Dispatches child lambdas for each schedule and gives the current minute
     - Collects results, logs outcomes, and sends final alerts (if any) for that particular minute
     - Send final alert at the end of the 30-minute block saying next block is starting (if there was any conditions true inside the 30 minute block)

   - A Child Lambda that:
     - Accepts a single demand control schedule and the current minute
     - Executes the respective needed for that minute (Check Conditions 1 through 6 detailed below with their respective execution minute)
     - Follows the specified process flow logic
     - Calculates whether an alert should be sent and sends it using the given Notification API
     - Returns the outcome back to the Master Lambda for each particular minute

 System Overview

- Each demand control check runs in a 30-minute block
- Each 30-minute block includes special minute checkpoints at: 20, 21, 22, 23, 24, 25
- Also include corresponding 50-55th minute values for the demand difference calculations
- Make sure the time used for checking is in the respective timezone (given in the Demand Control Schedule)
- Due to Data latency from device ingestion to database:
  - All checks must be run 4 minutes after the intended minute mark due to the latency
  - Make latency value configurable for future scaling
  - Latency means that the minute to check will be (minute to check = current minute - latency time)

 Demand Control Logic

1. Demand Calculation:

   - 30-minute block demand: (kWh_at_end - kWh_at_start) - 2
   - 1-minute block demand: (kWh_at_end_of_minute - kWh_at_start_of_minute) - 2

2. Thresholds:

   - Target (x): max allowed kW for 30-minute block
   - Alert Threshold (y): (2/3)*x
   - Per-minute threshold (z): x/30

 Check Conditions (run sequentially)

| Minute | Check if demand exceeds | Formula to calculate load reduction |
| ------ | ----------------------- | ----------------------------------- |
| 20     | a > y                   | a1 = (a - y) / 2 - 7.5              |
| 21     | b > y + z               | b1 = (b - (y + z)) / 2 - 8.57       |
| 22     | c > y + 2z              | c1 = (c - (y + 2z)) / 2 - 10        |
| 23     | d > y + 3z              | d1 = (d - (y + 3z)) / 2 - 12        |
| 24     | e > y + 4z              | e1 = (e - (y + 4z)) / 2 - 15        |
| 25     | f > y + 5z              | f1 = (f - (y + 5z)) / 2 - 20        |

 Process Flow Rules

- Run Check Conditions 1 to 6 in order
- If Check 1 is triggered:
  - Send alert
  - Skip Check 2, 3, 4, 6
  - Run Check 5 only

- If Check 2 is triggered:
  - Skip Check 3, 4, 5
  - Run Check 6 only

- If any of Check 3, 4, 5, or 6 are triggered:
  - Skip subsequent checks

- If any check is triggered:
  - At the 30-minute mark, send a final demand calculated for the check and send notification

 Notification API Spec

Send alerts using:

POST https://notificationapi.com/api/notification

With JSON body:

json
{
  "client_id": "Insert client id here",
  "recipient_ids": ["Insert recipient id here"],
  "subject": "Insert subject here",
  "message": "Insert message here",
  "notification_category": "DC"
}


> Populate the client_id, recipient_ids, subject, and message based on schedule and check outcomes.

 🛠️ Settings & Scalability

- The latency buffer (default: 4 minutes) must be configurable via env variable.
- Must support parallel execution of child lambdas via async invocation.
- Design the system to allow additional checks in future.

 Requirements Summary

Please provide:

 1. PostgreSQL DDL:

- Only create Tables and relationships for:
  - analytics_data.demand_control_schedules with columns (schedule_id, client_id, device_id, demand_target, recipient_ids, timezone)
  - analytics_data.demand_control_history with columns (history_id, schedule_id, device_id, minute, condition_triggered, demand_value, reduction_amount, timezone, alert_sent)

 2. Python Chalice Lambda Code:

- Master Lambda:

  - Runs every minute
  - Keep track of the processes inside the 30 minute demand block
  - Loads all active schedules
  - Dispatches child lambdas nd gives the current minute
  - Log results (with app.log.info) and sends summary notification (if any check conditions was true inside the demand block)

- Child Lambda:

  - Accepts a single schedule
  - Queries energy readings at minute marks (accounting for latency)
  - Performs demand calculations needed for that particular minute mark (if any)
  - Runs check conditions using process flow for that particular minute mark (if any)
  - Sends notifications if needed
  - Returns results to Master Lambda for tracking the 30 minute demand block

- Include:

  - Clear modular structure
  - Comments explaining logic
  - Error handling & logging with app.log.info
  - Production-grade
  - Functions for calculating demand, thresholds, and check values
  - Retry or fallback logic if readings are missing
  - Configurable latency value

Also for reference on implementation, the recipient info is obtained from the client_data.recipients table using the recipient_id and the demand value for the device is obtained from iot_data.timescale_measurements with the respective device id and the parameter id for demand which is parameter_id = 7a9b18ad-3510-4e39-9f19-87fd173a1fdf (set this value as an environment variable that can be changed). Note that the iot_data.timescale_measurements has the time saved in UTC timezone, hence it needs to be converted into the appropriate timezone (obtained from Demand Control Schedule) when getting the data from the database.

> Please write clean, well-documented code and DDL for the PostgreSQL schema. Do not skip any part of the implementation — provide complete code for both orchestration and checking logic. You can the lambda split code between different functions but it should be in the same app.py, and simulate the environment with stubbed sample data if necessary. Include example records or helper functions to illustrate how things connect if relevant.
