# ENGN1931Z Midterm Exam

# Instructions

This examination is an individual take-home assignment.  You may not discuss the exam or its contents with other persons (including members of the class). 

This examination is an open-book, open-notes, open-web assessment.  You are welcome to use information you may find in books, in your notes, or on the internet; however, *you are **not** allowed to actively solicit information*. For example, you are welcome to look for relevant information on StackExchange, but you *cannnot* post on StackExchange requesting help.  

**Please remember that you may not solicit help from others in any way. This exam must be the product of your own independent work, and all exams will be compared for similarities**


# Duration and Submission

The exam will be due by 5pm on Friday December 2nd.

1. Please submit a single ZIP file via CANVAS containing archival versions of your work (e.g. \*.js files for javascript, \*.py files for python, \*.text or \*.pdf files for text explanations).  You are welcome to also submit any additional handwritten work for all questions; this work may be submitted as a part of the electronic files as a photo JPEG or scanned PDF.

2. Please also share with me a single Google Drive Folder that contains your submission files and relevant Google Docs/Sheets.  

# Goal and Context
You've been hired as an engineer at a new non-profit organization focused on promoting clean energy technology. Your first project is to develop an open-source [vehicle-to-grid](https://en.wikipedia.org/wiki/Vehicle-to-grid) system that can encourage local energy companies to subsidize electric vehicles, especially as a means to offset natural [intermittency of renewable energy sources](https://en.wikipedia.org/wiki/Intermittent_energy_source) (e.g., solar, tidal, wind). Of course, there will be many economic, political, social, and technical constraints that will determine whether this path is viable. However, as an engineer, you are excited by the challenge of creating such a solution yourself.... 

Building on your ENGN1931Z experience, you decide to prototype the essential components using three main components: (1) Google Apps Script to log data, (2) On-Board Diagnostic CAN Bus commands to query vehicle information, (3) Web APIs to query local energy pricing and sources, and (4) wireless Automated Meter Readings to monitor electricity flow. The following questions will lead you through several design challenges that you may encounter along the way.

# Probelm 1: Web-Based Data Logger

You decide to use Google Sheets to create a web-based data logger where you can pass relevant information using HTTP GET requests. To this end, please deploy a Google-Apps-Script web app that allows anyone on the web (even anonymous users) to post data to a Google Sheet that you have created in Drive.  The web app should have the following functionality:

* a. If a parameter key called `sheet` is included in the request, then the information should be added to the sheet with that name. (If this sheet does not exist yet, your script should programmatically create it.) Otherwise, if the `sheet` parameter is not included, it should post the information to the default sheet called "Sheet1".

* b. The top row of each sheet should include headers for the data. The first header should be "Timestamp", and the subsequent headers should be added programmatically from the parameter keys. For example, if you pass the parameter (`...?batteryPercentage=80&kwhPrice=15.41`), then you should have headers that include at least "Timestamp", "batteryPercentage", and "kwhPrice". If any of these headers does not yet exist, it should be added by your script to the first empty column.

* c. New data should be added to first empty row (i.e. just below the old data) with the values in the appropriate rows (as labeled by the headers above).

**Please include the web app URL in comments at the top of your Apps-Script, and please make sure to include both the Sheet and Apps-Script in your submission folder.**

# Problem 2: OBD-II Queries

You want to use a low-cost CAN-bus reader to query relevant vehicle information through [OBD-II PID](https://en.wikipedia.org/wiki/OBD-II_PIDs) requests. To make sure you understand the command syntax, you decide to first query the available [Mode 1 PIDs]() on your own car. 

To do this, you have setup a web app that can send CAN commands to your car, and then returns the CAN responses. To simplify the format, your web app commands and responses will always start with a 0-bit followed by an 11-bit address followed by 8 bytes of data. So all commands and responses will be (12+8\*8)=76 bits long. Note that the appropriate 11-bit address for all OBD-II queries is `7DF` in hexadecimal, and a single query may return zero, one, or more responses. (* Note that it may help to look at the [example PID queries and responses information here](https://en.wikipedia.org/wiki/OBD-II_PIDs#CAN_.2811-bit.29_bus_format)*)

**The URL for the simulated OBD system is: goo.gl/ , and you can base your queries as a 76-bit binary string (i.e. 76 zeros or ones)**

**Note that a single PID query may return zero, one, or more responses. Zero means the method is not supported; one means that only one ECU replied; more than one means that more than one ECU replied.**

Please write a python script that can performs the following actions:

* a. Finds the supported Mode 01 PIDs by sequentially sending the "supported PIDs" commands (e.g. `00`, `20`, `40`, ... `A0`, `C0`,`E0`) and processing the [bit-encoded responses](https://en.wikipedia.org/wiki/OBD-II_PIDs#Mode_1_PID_00). *NB: You can double check the validity of your code by seeing if the supported PIDs actually work, but you cannot brute force a solution - credit will only be given for correctly parsing the reply data from the Support PID responses.)

* b. Query the Ambient Air Temperature and print value in degrees Celsius

* c. Query the Control Module Voltage and print value in Volts for each responding module together with the responding module address. (*Hint: these values should be greater than 12 Volts.*)

# Problem 3: 

You also want to track the local price of electricity in real-time so that you know when to buy and sell electricity. Of course, there will be many practical limitations to when and how you can buy and sell, but the real-time locational marginal price (LMP) can provide a good baseline estimate for the rates you might get if you could negotiate a wholesale deal. (Note that we cannot access the real-time bid/ask pricing, because we are not market participants. However, we can see near real-time LMP as a useful metric.)

Note that the wholesale LMP varies quite dramatically throughout a day, as shown below: ![Example LMP Variations](exampleVariationsLMP.PNG?raw=true).

Please write a python script that queries the [ISO New England Web Services API](https://webservices.iso-ne.com/docs/v1.1/index.html) to return the following information:

* a. Query and print the Current Five-Minute LMP for the Rhode Island load zone.
* b. Query the current Fuel Generation Mix, and print out the current percentages of power from Hydro, Solar, and Winds sources. 

**Note that you will need to check the "ISO Data Feeds" when registering for an [ISO Express Account](https://www.iso-ne.com/isoexpress/web/guest/login), and then use HTTP Basic Authentication within the python requests library to access the API.**

# Problem 4: 

Finally, you would like to be able to track changes to your home electricity meter using a wireless [Automated Meter Reading](https://en.wikipedia.org/wiki/Automatic_meter_reading) technology. You find a low-cost software-defined radio (SDR) that can read AMR packets than can log data into a CSV (comma separated value file) such as ![this example](exampleAMR.csv). However, there are lots of smart meters near your home, and they each send out a packet every few seconds. Therefore, you want to filter the data to look only at the packets from your home meter.

Please write a python script that:

* a. Searches through an AMR CSV file such as ![exampleAMR.csv](exampleAMR.csv) using regular expressions to find the rows associated with a specific `MeterID` (e.g. ID# 14452472), and prints out changes in the `Consumption` together with the associated `TimeStamp` for the changes.
