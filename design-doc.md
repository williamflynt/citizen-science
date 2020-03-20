# Citizen Science
###### 2020-03-20 // ch5

A mobile app to harness the power of crowdsourced science, starting with COVID-19.

### Idea

This app is being developed as a solution for scaling longitudinal studies of persons who have been tested for COVID-19.

It should allow tracking of: blood samples, analysis results, and self-reported survey data for tens/hundreds of thousands of users.

More broadly, it can also serve as a platform for any study that collects discrete samples from people, and optionally follows those people through time/space to collect more information.

### Scope Disclaimer

Not everything has to work all at once. We can hack out the biggest things first and go from there!

Also these ideas probably aren't perfect, so let's work on them together!

### Important Entities

* User - any user of the application (like a blood donor)
* Study - a distinct study with a name and scope (ex: COVID-19 Tacoma)
* Sample - a data collection event (like a blood draw)
* SampleProfile - some User information collected with the Sample (possibly also environment data or Sample metadata)
* SampleData - data about the sample, including analysis results (ex: positive for COVID-19 antigens, X ppm concentration of Y chemical, ...)
* QuestionSet - a study-linked set of questions for Users to answer on their app (ex: Have you had any symptoms today? Which ones? How often?)

### Possible Architecture

* User management with Firebase ecosystem
* Single `data` topic (stream) for all Study data input
* Data persistence layer can be cloud object storage (raw `data`), BigQuery (direct load from GCP PubSub), or any other storage option with an ETL process as a `data` subscriber

### Possible Stack

* Firebase Auth (WhatsApp phone auth model)
* Firestore (app data storage layer)
* Cloud SQL (relational entity storage)
* Firebase (all mobile app requirements, like push notification)
* GCP PubSub (data stream)
* BigQuery (data analysis/storage)

### Sample Flow 1

This is the flow of a User joining for the first time after giving blood for a COVID-19 study.

1. Person gives blood (Sample)
2. Technician (also a User) creates Sample in database (posted to `data` topic)
3. Technician generates QR for Sample
4. Person installs Citizen Science app and registers (User)
5. User scans QR with app
6. Sample posted to `data` topic (w/ User + metadata)
7. User presented Study consent form (if not already enrolled)
8. User enrolled in linked Study (if not already enrolled) - posted to `data`
9. User presented with SampleProfile (completes)
10. SampleProfile posted to `data`
11. User device periodically posts to `data` with device sensor info
12. Push notifications periodically with QuestionSet for User to self-report
13. User answers QuestionSet - posted to `data`

### Sample Flow 2

This is the flow for testing the Sample and logging results of analysis (SampleData).

1. Person downloads app and registers (User)
2. User is made a Technician by Study Admin
3. Sample goes to lab
4. Technician opens Study in Technician Mode
4. Technician scans QR with app - posted to `data`
5. Technician does not select SampleData button (just custody log)
6. Technician does the analysis
7. Technician scans QR with app - posted to `data`
8. Technician selects SampleData button
9. Technician inputs SampleData - posted to `data`

### Sample Flow 3

This is the flow for an organization that wants to replicate this COVID-19 study in their town.

1. Person downloads Citizen Science app, registers (User)
2. User finds public study with search (ex: "COVID-19 Tacoma")
3. User clicks clone button
4. User updates study information as needed
5. Study is replicated with new ID, updated info
6. Study is live and ready to go!

### Analyzing Data & Data Use

* Data collected by any Study should be private
* Admins must be able to grant data analysis rights to people on their team (otherwise the actual science doesn't get done)
* Initial data analysis can be done by allowing Users with rights to a Study to download unstructured `data` messages and other relevant information for data scientists to use directly.
* Addressing User privacy rights is important, and should be an area for investment following the initial basic functionality to allow us to quickly begin tracking COVID-19-related Samples and data.

### HIPAA

We could use some help on HIPAA compliance. Here is what we know.

* RAIN has an attorney-drafted user consent agreement for HIPAA waiver ready for this application.
* State of emergency plays here as well - but that's murkier.
* The PhDs in charge of the testing are in contact with the WA governor directly on this initiative.

### QuestionSet

This is a set of questions arranged in an (optional) branching format. For example, your answer to one question might prompt several other questions. For example:

* How are you feeling? (Perfect health / I may have some symptoms)
    - Do you have a cough?
        + How hard of a cough?
        + How often?
        + Productive?
    - Do you have a fever?
        + ...
* Have you been outside lately?
    - ...

We should be able to push out new questions to Users on demand.

### Data Topic

The `data` topic is where all Study data (ie: not User management stuff) is posted. It is a write-only log, ordered in time, and it can be compressed and stored. At any time an ETL pipeline can read the entire `data` topic and generate new forms of structured, related data.

### Create a Study

* Any User can create a Study (public or private)
* Any User can join any public Study
* Creating User can add other Study Admins
* Study Admins can invite to private study
* Standardized consent forms/data licenses for Study creation

### User Engagement

We should reward Users for participating in Studies by showing them what they're contributing to!

A "My Studies" dashboard with aggregated results (read: nice graphs) for each study would go a long way toward retaining Users for longitudinal studies.

---

# Alternatives

Instead of this mobile app, we might try:

### Text --> PWA

The main reasons to use a mobile app are:

* Push notifications for User engagement
* Access to device sensor data
* Camera access to QR/barcode data

We can replace some of this flow with:

* Automated text messaging (Twilio, others)
* Mobile-friendly map-based questions (where are you?)
* Manual barcode entry and encoding data into a barcode

It may be faster to develop and deploy, as well.
