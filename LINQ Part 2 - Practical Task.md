# LINQ Practical Tasks

**1) Display the total number of students registered in the system.**

**2) Display the total number of faculty members guiding projects.**

**3) Display the total number of projects available in the system.**

**4) Show how many tasks belong to each status category.**

| Status    | Tasks |
| --------- | ----- |
| Completed | 120   |
| Pending   | 45    |
| Ongoing   | 30    |
| Cancelled | 10    |

**5) Show priority wise task count**

| Priority | Tasks |
| -------- | ----- |
| Critical | 20    |
| Moderate | 80    |
| Low      | 50    |

**6) Show how many projects are assigned to each faculty member.**

| Faculty   | Projects |
| --------- | -------- |
| Dr. Shah  | 15       |
| Dr. Patel | 12       |

**7) Show how many tasks have been assigned to each student.**

| Student | Tasks |
| ------- | ----- |
| Amit    | 15    |
| Ravi    | 12    |

**8) Display the top 10 students having the highest average earned score.**

| Student | Avg Score |
| ------- | --------- |
| Amit    | 92        |
| Neha    | 90        |


**9) Display the bottom 10 students based on average earned score.**

| Rank | Student | Total Tasks | Average Score |
| ---: | ------- | ----------: | ------------: |
|    1 | Rahul   |          15 |         45.20 |
|    2 | Karan   |          18 |         48.50 |
|    3 | Priya   |          20 |         50.10 |
|    4 | Riya    |          16 |         52.40 |
|    5 | Aman    |          19 |         54.00 |

**10) Display all tasks whose due date has passed but are not completed.**

| Task ID | Task Title      | Student | Faculty   | Due Date    | Days Overdue |
| ------- | --------------- | ------- | --------- | ----------- | -----------: |
| 201     | API Security    | Amit    | Dr. Shah  | 20-Jul-2026 |           15 |
| 202     | Role Management | Ravi    | Dr. Patel | 25-Jul-2026 |           10 |
| 203     | Dashboard UI    | Neha    | Dr. Mehta | 28-Jul-2026 |            7 |

**11) Display tasks having follow-up dates within the next 7 days.**

| Task Title         | Student | Faculty   | Follow-Up Date |
| ------------------ | ------- | --------- | -------------- |
| JWT Authentication | Amit    | Dr. Shah  | 06-Aug-2026    |
| Dashboard API      | Ravi    | Dr. Patel | 08-Aug-2026    |
| Testing Module     | Neha    | Dr. Mehta | 10-Aug-2026    |

**12) Show how many students have obtained each grade.**

| Grade | Students |
| ----- | -------- |
| A     | 30       |
| B     | 45       |
| C     | 20       |

**13) Show month-wise completed task count.**

| Year | Month    | Completed Tasks |
| ---: | -------- | --------------: |
| 2026 | January  |              45 |
| 2026 | February |              52 |
| 2026 | March    |              68 |
| 2026 | April    |              81 |
| 2026 | May      |              94 |
| 2026 | June     |             105 |

**14) Display Role Wise Active User Count.**

| Role        | Active Users |
| ----------- | -----------: |
| Student     |          285 |
| Faculty     |           17 |
| Admin       |            3 |
| Coordinator |            2 |

**15) Display each role with users assigned to it.**

| Role    | User Name |
| ------- | --------- |
| Admin   | Madhuresh |
| Admin   | Ravi      |
| Faculty | Dr. Shah  |
| Faculty | Dr. Patel |
| Student | Amit      |
| Student | Neha      |

**16) List Roles Having More Than 10 Users.**

| Role    | Total Users |
| ------- | ----------: |
| Student |         285 |
| Faculty |          17 |

**17) Display role statistics.**

| Role    | Total Users | Active Users | Inactive Users |
| ------- | ----------- | ------------ | -------------- |
| Student | 250         | 240          | 10             |
| Faculty | 15          | 14           | 1              |
| Admin   | 2           | 2            | 0              |

**18) Show tasks due within next 7 days.**

| Task ID | Task Title         | Project    | Student | Due Date    | Days Remaining |
| ------- | ------------------ | ---------- | ------- | ----------- | -------------: |
| 101     | JWT Authentication | ERP System | Amit    | 10-Aug-2026 |              6 |
| 102     | Dashboard UI       | LMS Portal | Ravi    | 08-Aug-2026 |              4 |
| 103     | API Testing        | CRM System | Neha    | 06-Aug-2026 |              2 |

**19) Display each project with total tasks, completed tasks, pending tasks, and average task progress.**

| Project | Tasks | Completed | Pending | Avg Progress |
| ------- | ----- | --------- | ------- | ------------ |
| ERP     | 20    | 15        | 5       | 78%          |

**20) Display project-wise total assigned score, earned score, and score percentage.**

| Project             | Total Assigned Score | Total Earned Score | Score Percentage |
| ------------------- | -------------------: | -----------------: | ---------------: |
| ERP System          |                  500 |                450 |           90.00% |
| LMS Portal          |                  400 |                360 |           90.00% |
| Inventory System    |                  300 |                240 |           80.00% |
| Hospital Management |                  600 |                510 |           85.00% |

**21) Display Top 10 projects based on average earned score.**

| Rank | Project             | Average Score |
| ---: | ------------------- | ------------: |
|    1 | ERP System          |         95.50 |
|    2 | LMS Portal          |         93.25 |
|    3 | Inventory System    |         91.75 |
|    4 | Hospital Management |         90.20 |
|    5 | Library Management  |         89.50 |

**22) Show project count, task count, and average progress for each faculty.**

| Faculty     | Total Projects | Total Tasks | Avg Progress (%) |
| ----------- | -------------: | ----------: | ---------------: |
| Dr. Shah    |             12 |         180 |            84.50 |
| Dr. Patel   |             10 |         150 |            79.20 |
| Dr. Mehta   |              8 |         120 |            76.80 |
| Dr. Trivedi |              6 |          95 |            72.10 |

**23) Display task completion statistics and average score for each student.**

| Student | Total Tasks | Completed Tasks | Pending Tasks | Avg Score |
| ------- | ----------- | --------------- | ------------- | --------- |
| Amit    |          20 |              18 |             2 |     92.50 |
| Ravi    |          18 |              15 |             3 |     88.20 |
| Neha    |          22 |              20 |             2 |     94.10 |
| Priya   |          16 |              12 |             4 |     84.75 |

**24) Display projects whose expected completion date has passed but are still incomplete.**

| Project    | Student | Faculty   | End Date    | Progress (%) |
| ---------- | ------- | --------- | ----------- | -----------: |
| ERP System | Amit    | Dr. Shah  | 15-Jul-2026 |           85 |
| LMS Portal | Ravi    | Dr. Patel | 10-Jul-2026 |           78 |
| CRM System | Neha    | Dr. Mehta | 20-Jul-2026 |           72 |

**25) Show month-wise completed task count.**

| Year | Month | Completed Tasks |
| ---: | ----: | --------------: |
| 2026 |     1 |              45 |
| 2026 |     2 |              52 |
| 2026 |     3 |              60 |
| 2026 |     4 |              71 |
| 2026 |     5 |              80 |
| 2026 |     6 |              92 |

**26) Rank faculties based on average project progress.**

| Rank | Faculty     | Avg Progress (%) |
| ---: | ----------- | ---------------: |
|    1 | Dr. Shah    |            89.50 |
|    2 | Dr. Patel   |            85.75 |
|    3 | Dr. Mehta   |            81.20 |
|    4 | Dr. Trivedi |            78.90 |

**27) Display task statistics for every project.**

| Project          | Total Tasks | Completed | Pending | Overdue |
| ---------------- | ----------: | --------: | ------: | ------: |
| ERP System       |          20 |        18 |       2 |       0 |
| LMS Portal       |          25 |        20 |       5 |       1 |
| CRM System       |          18 |        12 |       6 |       2 |
| Inventory System |          22 |        20 |       2 |       0 |
