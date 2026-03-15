# Follow-up Ticket (Fill in)

**Title:** 500 errors, task submission slowness, duplicate entries on refresh
**Priority:** (P0/P1/P2/P3) P2
**Owner:** Carson Crooks

## Description
What should be improved after the immediate incident is resolved? Several improvements are needed now that the title issues have been resolved. 
    - First the testing script needs to be fixed (done) so that we do not commit issues like this to prod in the future.
        - Also we should add more test cases to the test to make it more robust and help us keep a stable application.
    - Next we need to make updates that allow end-users to sort the list of tasks by task number or status so that the order is less confusing (users thought that things were out of order but they were just sorted by date not id).
    - Last I noticed that tasks with the same name are not accepted. We should either allow duplicate names or if that will cause downstream issues we should display a warning or notice to the end-user to let them know that the task name is already taken and to choose a new one.
    

## Acceptance criteria
- [ ] ... 
- [ ] ... 

## Notes / context
- Links to relevant code/areas

  More test cases
  - tests/SupportEngineerChallenge.Tests/TaskApiTests.cs — add new [Fact] methods. Good candidates: creating a task
   without a timestamp header, creating a task with missing fields, listing tasks for a user with no tasks,
  verifying the limit parameter is respected.

  ---
  Sorting by task number or status
  - src/SupportEngineerChallenge.Api/Endpoints/TaskEndpoints.cs — add a sortBy query parameter to the GET
  /api/tasks endpoint and branch the OrderBy logic accordingly.
  - src/SupportEngineerChallenge.Api/wwwroot/index.html — add a sort dropdown control.
  - src/SupportEngineerChallenge.Api/wwwroot/main.js — pass the selected sort value to the API request.

  ---
  Duplicate task name handling
  - First, I don't actually see any duplicate prevention in the current code — no unique constraint in
  AppDbContext.cs and no check in TaskEndpoints.cs. Can you reproduce it? It's possible the behavior is coming from
   somewhere unexpected.
  - If confirmed, the check would go in TaskEndpoints.cs in the POST handler, and the user-facing warning would go
  in main.js where errors are already surfaced via setError().

- Any monitoring/alerting suggestions
    - Set up scripts that send an alert to Support Engineers when errors in the platform happen (eg 500 errors in logs).
    - When a bug report from a customer comes in with P2 or higher immediately notify the on-call engineer for investigation.

