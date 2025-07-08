## Model Class
```csharp
    public class Student
    {
        public int Rno { get; set; }
        public string Name { get; set; }
        public string Branch { get; set; }
        public int Sem { get; set; }
        public double CPI { get; set; }
        public int Age { get; set; }
        public override string ToString()
        {
            return $"Rno: {Rno}, Name: {Name}, Branch: {Branch}, Sem: {Sem}, CPI: {CPI}, Age: {Age}";
        }
    }

    public class Course
    {
        public int Rno { get; set; }
        public string CourseName { get; set; }
        public int Credits { get; set; }

        public override string ToString()
        {
            return $"Rno: {Rno}, Course: {CourseName}, Credits: {Credits}";
        }
    }
```
---
## Below is sample data (write inside main method)
```csahrp
static List<Student> GetStudents()
        {
            // Note: Using traditional List<Student> for compatibility. For C# 12, can use: [new Student { ... }, ...]
            return new List<Student>
            {
                new Student { Rno=101, Name="Ramesh", Branch="CE", Sem=3, CPI=9.2, Age=23 },
                new Student { Rno=102, Name="Mahesh", Branch="EC", Sem=3, CPI=8.8, Age=21 },
                new Student { Rno=103, Name="Suresh", Branch="ME", Sem=4, CPI=7.5, Age=22 },
                new Student { Rno=104, Name="Amit", Branch="EE", Sem=4, CPI=8.6, Age=20 },
                new Student { Rno=105, Name="Anita", Branch="CE", Sem=4, CPI=8.0, Age=22 },
                new Student { Rno=106, Name="Reeta", Branch="ME", Sem=3, CPI=7.35, Age=21 },
                new Student { Rno=107, Name="Rohit", Branch="EE", Sem=4, CPI=9.45, Age=24 },
                new Student { Rno=108, Name="Chetan", Branch="CE", Sem=3, CPI=8.6, Age=26 },
                new Student { Rno=109, Name="Rakesh", Branch="CE", Sem=4, CPI=9.10, Age=24 },
                new Student { Rno=110, Name="Alice", Branch="CSE", Sem=4, CPI=8.5, Age=20 },
                new Student { Rno=111, Name="Jay", Branch="CE", Sem=5, CPI=6.5, Age=21 },
                new Student { Rno=112, Name="Vijay", Branch="ME", Sem=4, CPI=5.5, Age=22 },
                new Student { Rno=113, Name="Rajvi", Branch="CSE", Sem=5, CPI=5.5, Age=19 },
                new Student { Rno=114, Name="Viru", Branch="ME", Sem=4, CPI=8.5, Age=24 },
                new Student { Rno=115, Name="Amee", Branch="ME", Sem=5, CPI=8.5, Age=24 }
            };
        }

        static List<Course> GetCourses()
        {
            return new List<Course>
            {
                new Course { Rno=101, CourseName="Data Structures", Credits=4 },
                new Course { Rno=101, CourseName="Algorithms", Credits=3 },
                new Course { Rno=102, CourseName="Circuit Design", Credits=4 },
                new Course { Rno=103, CourseName="Mechanics", Credits=3 },
                new Course { Rno=104, CourseName="Electronics", Credits=4 },
                new Course { Rno=105, CourseName="Database Systems", Credits=3 },
                new Course { Rno=106, CourseName="Thermodynamics", Credits=3 },
                new Course { Rno=107, CourseName="Power Systems", Credits=4 },
                new Course { Rno=108, CourseName="Operating Systems", Credits=4 },
                new Course { Rno=109, CourseName="Software Engineering", Credits=3 },
                new Course { Rno=110, CourseName="AI", Credits=4 },
                new Course { Rno=111, CourseName="Networking", Credits=3 },
                new Course { Rno=112, CourseName="Fluid Mechanics", Credits=3 },
                new Course { Rno=113, CourseName="Machine Learning", Credits=4 },
                new Course { Rno=114, CourseName="Robotics", Credits=3 }
            };
        }

```
---

## LINQ Query Questions

1. Total Number of Students
2. Select Student Name, Branch, CPI
3. Distinct Branches
4. Distinct Semesters
5. Students with CPI > 8.0
6. Students with CPI between 7 and 8
7. Students with Age > 22
8. Students with Branch 'CE'
9. Students with Semester < 5
10. Students with Name Contains 'Ramesh'
11. Students with Name Contains 'e'
12. Students with Name Starts with 'R'
13. Students with Name Ends with 'h'
14. Students Ordered by Name
15. Students Ordered by CPI Descending
16. Select Student Name, Branch, CPI with OrderBy
17. Students with Sem 3 and CPI > 8.0
18. Students with Sem 4 and Age < 22
19. Students with Name Contains 'e' and Sem is 3
20. Students with Name Starts with 'R' and Sem is 3
21. Students with Sem 3 or Branch 'ME' and Age > 21
22. Students Ordered by Branch then by Name
23. Students with Age > 22 Ordered by Name
24. Students with CPI > 8.0 Ordered by CPI Descending
25. Students with Branch 'CE' Ordered by Sem and Name
26. Students with CPI > 8 and Sem is 3, Ordered by Name
27. Take First 3 Students
28. Skip First 3 and Take Next 3 Students
29. Branch Wise Student Count
30. Group Students by Semester
31. Average CPI of All Students
32. Minimum CPI
33. Maximum CPI
34. Group Students by Branch
35. Flatten All Students from Grouped Data
36. Students Ordered by Branch and then by CPI
37. Students Ordered by Sem, Branch, then Name
38. Students in Reverse Order by Rno
39. Top 5 Students by CPI using Take
40. Skip First 3 and Take Next 5 Students
41. Group Students by Sem and Age
42. Group Students by Sem and Branch with Count
43. Group by Branch and Sem
44. Branch wise Semester Wise Highest CPI
45. Any Student with CPI > 9.0
46. All Students have CPI > 7.0
47. Aggregate to Calculate Total CPI
48. First Student with CPI > 8
49. Last Student with CPI > 8
50. Students with CPI > 8 and Sem is 3, Ordered by Rno then Name
51. Join Students with Courses
52. Group Students by Branch and Count High CPI (> 8.0)
53. Any Branch with More Than 5 Students
54. All Branches Have At Least One Student with CPI > 8
55. Join Students with Courses, Filter by Credits >= 4
56. Group Students by Age Range
57. Any Student in Sem 3 with Course Credits = 4
58. All Students in CE Branch Enrolled in Courses
59. Group Students by Branch and Average CPI
60. Join Students with Courses, Order by Course Credits
61. Group Students by CPI Range
62. Any Student with Multiple Courses
63. All Students in Sem 4 Have CPI > 6
64. Group Students by Branch and Sem with Minimum CPI
65. Join Students with Courses, Filter by Branch 'CSE'
66. Group Students by Age and Count in Sem 3
67. Any Branch with Average CPI > 8
68. All Students in Branch 'EE' Have Age >= 20
69. Group Students by Branch and Total Credits
70. Join Students with Courses, Select Distinct Course Names
71. Group Students by Sem and Count High Age (> 22)
72. Any Student with CPI > 9 and Enrolled in AI Course
73. All Students in Sem 5 Enrolled in Courses
74. Group Students by Branch and Max Age
75. Join Students with Courses, Filter by Sem 3
76. Group Students by Branch and Count Courses
77. Any Student with Name Starts with 'A' in Sem 4
78. All Students in Branch 'ME' Have CPI > 5
79. Group Students by Sem and Average Age
80. Join Students with Courses, Order by Name and Course
81. Group Students by Branch and Count Young Students (< 22)
82. Any Student with CPI < 6 in Sem 5
83. All Students in Sem 3 Enrolled in 3-Credit Courses
84. Group Students by Branch and List Names
85. Join Students with Courses, Filter by Course Name Contains 'Systems'
86. Group Students by Sem and Count with CPI > 7
87. Any Branch with All Students Age > 20
88. All Students with Courses Have CPI > 7
89. Group Students by Branch and Sem with Average CPI > 8
90. Join Students with Courses, Count Courses per Student
91. Group Students by Branch and Count with Name Contains 'a'
92. Any Student with CPI > 8 and Age < 21
93. All Students in Branch 'CSE' Have Courses
94. Group Students by Sem and List Course Names
95. Join Students with Courses, Filter by CPI > 8
96. Group Students by Branch and Count Enrolled in 4-Credit Courses
97. Any Student in Sem 4 with Course 'Robotics'
98. All Students with CPI > 8 Have Courses
99. Group Students by Branch and Sem with Total Course Credits
100. Join Students with Courses, Select Students with Multiple Courses
