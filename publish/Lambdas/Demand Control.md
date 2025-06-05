## Demand Control Messages

Condition Check Message (Old):
Demand Control Alert - Check 1
Demand control check 1 triggered for device 708caa1f-c206-4c3b-ad0b-b97911bdfc3d.
Current demand: 624.00 kW
Threshold: 200.00 kW
Recommended reduction: 204.50 kW

Condition Check Message (New):
Demand Control Alert - (20th minute)
Demand control triggered for Meter Name - Location Name
Recommended reduction: 204.50 kW

Demand Control Block Summary:
A 30-minute demand control block has ended for Meter Name - Location Name
Demand conditions were triggered during this block.
Actual demand value: 484.00
Target demand value: 300.00
*See IoTWatt Demand Control for recommended reduction history
A new 30-minute block is now starting.


## Demand Control Criteria

There is 6 Event Checks

1.Check 20 minutes = a: 
- Check: If a > y, automatic alert is sent to customer requesting them to reduce their connected load by "a1" kW
- Calculate: a1 = (a-y)/2 - 7.5 

2.Demand at 21 minutes = b:
- Check: If c > y+2z, automatic alert is sent to customer requesting them to reduce their connected load by c1 kW
- Calculate: b1 = (b-(y+z))/2 * 8.57

3.Demand at 22 minutes = c:
- Check: If c > y+2z, automatic alert is sent to customer requesting them to reduce their connected load by c1 kW
- Calculate: c1 = (c-(y+2z))/2 - 10

4.Demand at 23 minutes = d:
- Check: If d > y+3z, automatic alert is sent to customer requesting them to reduce their connected load by d1 kW
- Calculate: d1 = (d-(y+3z))/2 - 12

5.Demand at 24 minutes = e:
- Check: If e > y+4z, automatic alert is sent to customer requesting them to reduce their connected load by e1 kW
- Calculate: e1 = (e-(y+4z))/2 - 15

6.Demand at 25 minutes = f:
- Check: If f > y+5z, automatic alert is sent to customer requesting them to reduce their connected load by f1 kW
- Calculate: f1 = (f-(y+5z))/2 - 20


Conditions:

Execute event checks in order 1 to 6 with the appropriate time for each event.

If Event Check 1 is true, send an alert to the recipient using the notification api immediately and skip Event Check 2-4,6 and execute Event Check 5

If Event Check 2 is true, skip Event Check 2-5 and execute Event Check 6

If any Event Check from Event Check 3 and onwards is true, skip the following Event Check

If there is any Event Check that is true; at 30minutes send an alert to the recipient using the notification api notifying them of final demand for the period and that new demand block has started
