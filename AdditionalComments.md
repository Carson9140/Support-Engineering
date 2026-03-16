# Additional Comments

I was able to confirm and fix all three issues.
- 500 errors upon task submission.
- Slowness in the app and the time it took for a task to show upon refresh.
- Duplicate task entries upon refresh.

Tradeoffs:
- In the interest of time and expediency I pushed all three fixes to my repository together. Ideally I would have put all three fixes in their own PR's and tested them independently. This would make rolling any one of these issues back easier and less risky.
- I used AI to to help me parse the code and help me code the fixes correctly. Since I have not used C# in quite a while I feared looking up syntax would have taken up a lot of my time. (The current application I support is coded in Clojure which is quite a different language).

What I would do next with more time:
- First I would add more test cases to the test to make it more robust and help us keep a stable application.
- Next I would make updates that allow end-users to sort the list of tasks by task number or status so that the order is less confusing (users thought that things were out of order but they were just sorted by date not id).
- Last I noticed that tasks with the same name are not accepted. We should either allow duplicate names or if that will cause downstream issues we should display a warning or notice to the end-user to let them know that the task name is already taken and to choose a new one.
