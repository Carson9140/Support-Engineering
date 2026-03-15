# Runbook — SupportEngineerChallenge

> Update this file as part of the exercise.

## Service overview
- **Service:** SupportEngineerChallenge.Api
- **Purpose:** Minimal task tracker (create + list tasks)
- **Data store:** SQLite (`app.db` in the API working directory)

## Common commands

**Run locally**
```bash
cd src/SupportEngineerChallenge.Api
dotnet run
```

**Run tests**
```bash
dotnet test
```
- Spot check in the app to verify.
    - submit tasks with various names and users
    - use refresh for each user


## Key endpoints
- `GET /api/tasks?userId={id}&limit={n}`
- `POST /api/tasks`

## Using log artifacts

- **Create-task 500:** Inspect `artifacts/sample_api_log.txt` (or production logs). Look for the `CreateTask request` line — `X-Client-Timestamp present=False` or `length=0` indicates missing/invalid header. The stack trace shows `FormatException` at `DateTime.Parse`.
- **Slow list:** Look for `ListTasks completed` lines with high `elapsedMs` (e.g. `artifacts/sample_slow_list_log.txt`). Correlate `userId` and `limit` with slow requests.

## Troubleshooting checklist (starter)
- Test UI by using all button and unsure its functioning (I do this first because this is what the end user sees)
- Run test script to ensure all tests are passing.

### “Create task fails with 500”
- Check API logs in console.
    - Example failure:
fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HNK2MP2T8D24", Request id "0HNK2MP2T8D24:0000000E": An unhandled exception was thrown by the application.
      System.FormatException: String '' was not recognized as a valid DateTime.
         at System.DateTime.Parse(String s)
         at SupportEngineerChallenge.Api.Endpoints.TaskEndpoints.<>c.<<MapTaskEndpoints>b__0_1>d.MoveNext() in /home/carson/dev/SupportEngineerDebugAssignment/src/SupportEngineerChallenge.Api/Endpoints/TaskEndpoints.cs:line 41
      --- End of stack trace from previous location ---
         at Microsoft.AspNetCore.Http.RequestDelegateFactory.ExecuteTaskResult[T](Task`1 task, HttpContext httpContext)
         at Microsoft.AspNetCore.Http.RequestDelegateFactory.<>c__DisplayClass102_2.<<HandleRequestBodyAndCompileRequestDelegateForJson>b__2>d.MoveNext()
      --- End of stack trace from previous location ---
         at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|7_0(Endpoint endpoint, Task requestTask, ILogger logger)
         at Swashbuckle.AspNetCore.SwaggerUI.SwaggerUIMiddleware.Invoke(HttpContext httpContext)
         at Swashbuckle.AspNetCore.Swagger.SwaggerMiddleware.Invoke(HttpContext httpContext, ISwaggerProvider swaggerProvider)
         at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.Http.HttpProtocol.ProcessRequests[TContext](IHttpApplication`1 application)

- Verify request payload and headers.
- Look for unhandled exceptions in `POST /api/tasks`.

### “Tasks list is slow”
- Confirm dataset size (seed can be large).
- Inspect how the list endpoint fetches and filters data.
- Review query patterns and database usage.

### “Duplicates / wrong order after refresh”
- Compare API response vs UI rendering.
- Check the UI state update logic during refresh.
- Verify how the list is merged and ordered.

## Verification steps (starter)
- Create tasks from UI and via Swagger.
- Refresh tasks repeatedly; confirm no duplicates and ordering is correct.
- Validate list endpoint returns only requested user's tasks.

## Rollback / mitigation ideas (starter)
- Roll back to last known good version.
    - git revert HEAD
- Temporarily disable problematic client behavior (feature flag / UI change).
- Add guardrails (e.g. input validation, error handling) to prevent unhandled exceptions.
