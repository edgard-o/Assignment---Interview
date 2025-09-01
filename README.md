# Assignment---Interview



******Part 1 – System Journeys and Architecture

User Journey 1 – Campaign Creation and User Journey 2 – Agent Flyer Scan

![Strategy and planning](https://github.com/user-attachments/assets/5e293e5e-8bef-4d67-9ade-0070bfc8a805)


******Part 2 – API Testing

- Overview
This project contains a set of automated API tests for the Open Charge Map API.
The purpose is to validate Oppizi’s integration with location-based infrastructure data, focusing on:

- Endpoints tested
GET /poi/ → Retrieves Points of Interest (charging stations).
GET /referencedata/ → Provides static reference values for charging stations.

- Test coverage
Response time (< 1000ms)
Status code (200)

- Response schema validation
Business logic checks (e.g., maxresults, country codes, valid coordinates, non-empty reference data)

- Tool used: Postman (Collection + Environment variables).
Prerequisites
Postman Desktop or Postman Web.

- A valid Open Charge Map API key. To obtain one: open Open Charge Map , sign in to openchargemap.org and browse to ‘my profile > my apps’ and click register an application.

To use your API key include it as the value of the ‘X-API-Key’ header (case sensitive) or key= query string parameter (also case sensitive).
Setup & Import

Download the provided Postman Collection JSON file: Assigment.postman_collection.json

Open Postman → click Import → select the JSON file.

Create a new Postman Environment (optional, recommended):

Variable: API_KEY → set your key value.

Use {{API_KEY}} in requests.

Verify that requests are configured with:

?key={{API_KEY}}

How to Run Tests
Option 1 – Single Request

Open a request (GET /poi/ or GET /referencedata/).

Click Send.

Check Tests tab for assertion results.

Option 2 – Collection Runner

Open Collection Runner.

Select OpenChargeMap_Tests collection.

Select environment with API_KEY.

Run collection → results will show pass/fail for each assertion.

Expected Results

GET /poi/

Status 200

Response < 1000ms

Returns max N results (maxresults)

Valid coordinates and AddressInfo present

GET /referencedata/

Status 200

Response < 1000ms

Contains expected country codes (US, GB, BR)

Non-empty ConnectionTypes

Sample Report (from Postman Runner)

✅ All tests passed (status, response time, schema, business rules).

⏱ Average response time: ~700ms.

⚠️ If API key is invalid → response will return 401 Unauthorized.

Check the Json file.

******Part 3 – Manual Testing Assignment

1. Test Design
Functional Scenarios
Successful reassignment before the campaign starts


Precondition: Campaign has not started, receiving agent is free, and the same location type.


Expected: Reassignment succeeds, audit log created, email sent, route locked for 24h.


Reassignment blocked after campaign start.


Precondition: Campaign already active.


Expected: Reassignment not allowed, error message shown, no audit/email triggered.


Reassignment blocked due to overlapping routes.e


Precondition: Receiving agent already assigned to overlapping time slot.


Expected: Reassignment fails with conflict error.


Reassignment blocked due to a mismatched location type.
An indoor route cannot be assigned to an agent who only handles outdoor routes.


Audit log validation


Each reassignment must appear in the audit with a timestamp, route ID, old agent, and new agent.


Email validation


Both old and new agents receive confirmation emails.


Route lock validation


After reassignment, the route cannot be reassigned again within 24h.


Negative Scenarios
Unauthorized user attempts reassignment


A non-admin user tries to reassign the route.


Expected: Permission denied.


Invalid route ID provided


Expected: Error message, no audit/email.


Network failure during reassignment


Expected: Transaction rollback, no partial audit/email.


Edge Scenarios
Reassignment exactly at campaign start time (boundary)


Expected: Blocked as the campaign is already active.


Reassignment scheduled within 1 min of another overlapping route


Check strict time conflict validation.


Reassigning the same route immediately after the 24-hour lock expires


Expected: Allowed.

2. Test Data Table

Test Case ID	  Campaign Name	  Route ID	Source Agent	  Target Agent	Schedule	                Location Type	    Expected Outcome
TC01	          Campaign A	    R001	    Alice	          Bob	          01/09 09:00–11:00	        Indoor	          Success, audit entry, 2 emails, lock applied
TC02	          Campaign A	    R002	    Alice	          Charlie	      01/09 09:00–11:00	        Indoor → Outdoor	Fail: mismatch location
TC03	          Campaign B	    R010	    David	          Emma	        02/09 14:00–16:00 	      Outdoor	          Fail: overlapping
TC04	          Campaign C	    R020	    Grace	          Henry	        Past campaign (start 25/08)	Indoor	        Fail: campaign started
TC05	          Campaign D	    R030	    Ian	            Jane	        05/09 10:00–12:00	         Outdoor	        Success first time; second reassignment within 24h fails


3. Audit and Email Verification
Audit Log Verification
Step 1: After reassignment, open the admin dashboard → Audit section or call the audit API endpoint.


Step 2: Validate fields logged: {routeID, campaignID, sourceAgent, targetAgent, timestamp, performedBy}.


Step 3: Ensure “lock applied until” timestamp is included.

Email Verification
Step 1: Open test inbox (or mock service like MailHog / MailTrap).


Step 2: Verify 2 emails received:

Source Agent: “Your route has been reassigned.”

Target Agent: “A new route has been assigned to you.”

Step 3: Check timestamps, subject, body content, and campaign/route details.


4. Bug Reporting Sample
Title: Route reassignment allowed despite overlapping schedule
 Environment: Staging, Chrome v117, Build 1.0.5
 Preconditions: Campaign B active, Agent Emma already assigned to Route R011 (02/09 14:00–16:00).
 Steps to Reproduce:

Log in as Manager.
Reassign Route R010 (02/09 14:00–16:00) from David → Emma.

Confirm reassignment.
 Expected Result: System should block reassignment with the error “Target agent has overlapping route.”
 Actual Result: Reassignment succeeds, audit log created, email triggered.
 Severity: High – creates a double-booked agent and undermines scheduling integrity.
 Attachments: Screenshot of overlapping schedule, API log.
 
6. Assumptions & Risks
Assumptions
System auto-detects overlaps based on exact datetime ranges (not location proximity).

“Lock for 24h” means calendar time, not “24 business hours.”

Emails are mandatory, and the system retries if the initial send fails.

Risks
Overlapping detection might miss near-boundary cases (e.g., 1-minute gaps).

Locks may prevent urgent reassignment fixes in real campaigns.

If audit logging fails silently, compliance tracking is compromised.
