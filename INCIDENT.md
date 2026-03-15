# Incident Summary (Fill in)

**Title:**  500 errors, Slowness, and tasks not organized
**Date:**  3/15/26
**Severity:**  High (not emergency)

## Impact
- Who/what was impacted? All users impacted. May slow down users when using the app and cause frustration but workarounds exits to continue use.
- Symptoms observed by customers/internal users
    - 500 errors when submitting tasks.
    - Task list is slow for some users.
    - Tasks appear duplicated or out of order after refresh.

## Detection
- How did we learn about this? (customer reports, monitoring, etc.)
    - Customer reported the problems Symptoms observed under Impact.
    - I was able to reproduce the 500 errors on an intermittent basis by submitting tasks and seeing 500s about 1/3 of the time.
    - Slowness was observed. It was taking a good amount of time for a task to show up after a refresh if at all.
    - I was able to reproduce the duplicate tasks showing up. Although the tasks were ordered by date correctly. They may have appeared out of order if the user was looking at other columns.

## Timeline (UTC)
- HH:MM — ... 10:00am PST
- HH:MM — ... 

## Root cause
- What happened and why? 
1. TaskEndpoints.cs:17 — GET /api/tasks loads the entire table into memory before filtering. With 5 × 12,000 =
  60,000 rows, this is a full table scan on every request and could cause some slowness.
2. TaskEndpoints.cs:41 — DateTime.Parse(clientTimestamp) is called before the null-check. The frontend sends an
  empty X-Client-Timestamp ~35% of the time (Math.random() > 0.35 is false), causing a FormatException → 500 on
  those requests.
3. main.js:46 — state.tasks = state.tasks.concat(items) accumulates duplicates on every refresh instead of
  replacing.

## Mitigation / resolution
- What did we change to stop the bleeding? 
- What was the final fix?
  ---
  Bug 1 — Full table scan on GET /api/tasks
                                                                                                                   
  All 60,000 rows are loaded into memory before filtering in C#. Fix: push the WHERE, ORDER BY, and TAKE into the
  SQL query.
                  
  // Before
  var all = await db.Tasks.AsNoTracking().ToListAsync();
  var filtered = all.Where(t => t.UserId == userId)
      .OrderByDescending(t => t.CreatedAt)
      .Take(Math.Clamp(limit ?? 50, 1, 200))
      .ToList();

  // After
  var filtered = await db.Tasks.AsNoTracking()
      .Where(t => t.UserId == userId)
      .OrderByDescending(t => t.CreatedAt)
      .Take(Math.Clamp(limit ?? 50, 1, 200))
      .ToListAsync();

  ---
  Bug 2 — 500 crash on POST /api/tasks when timestamp header is empty

  DateTime.Parse("") throws a FormatException ~35% of the time because the frontend randomly sends an empty header.
   Fix: fall back to DateTime.UtcNow.

  // Before
  var createdAt = DateTime.Parse(clientTimestamp);

  // After
  var createdAt = DateTime.TryParse(clientTimestamp, out var parsed)
      ? parsed
      : DateTime.UtcNow;

  ---
  Bug 3 — Tasks duplicate on every refresh in the frontend

  concat appends new results to the existing list instead of replacing it. Fix: replace instead.

  // Before
  state.tasks = state.tasks.concat(items);

  // After
  state.tasks = items;

## Verification
- How did we verify the fix worked?
    - Committed these changes locally. 
    - Reran the app and cleared the cache to ensure I'm testing the new changes.
    - Spot checked in the app and found that 500's were no longer appearing when tasks were submitted.
    - Used the refresh button for all the users and no tasks were duplicated.
    - While using the refresh button to see tasks they show up promptly without delay.
    
    - After adding some fixes to the test script I was able to run it and it showed that both tests were passing.
    


## Follow-ups / action items
- [X] ... Initially I was unable to use the included test script. Which may have been why these issues were published to production in the first place. Two fixes were needed to get the tests running:
                                         
  1. Missing Microsoft.NET.Test.Sdk package — The test project was missing this package, which is required for dotnet test to discover and run tests. Added it to SupportEngineerChallenge.Tests.csproj.
  2. Missing using Xunit; — TaskApiTests.cs was using xunit types ([Fact], IClassFixture) without importing the    
  namespace. Added using Xunit; at the top of the file.
- [ ] ... Todo: While testing I noticed that if I names a task something that had already been taken nothing happened. No task was created which at first was confusing and a bit of a red herring when troubleshooting. Going forward we should either allow duplicate names if that does not cause any issues or if we need unique task names we should display some sort of warning for the user so they know whey their task was not added.
- [ ] ... Todo: Allow ordering of the list via task number or status so that the organization of the list is less confusing to the end-user.
- [ ] ... Todo: add more test cases to the test script to ensure nothing is missed.
