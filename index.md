---
layout: default
title: "Home"
permalink: /
---
<a id="top"></a>

{% include nav.html %}

# Welcome

I am a Computer Science student at Southern New Hampshire University, and this ePortfolio is my CS 499 capstone. It shows how I took one Android app from my CS 360 course and improved it in three areas: software design and engineering, algorithms and data structures, and databases. Each page has the code, a narrative explaining what I learned, and the evidence that it works.

**Where to go from here**

1. [Professional Self-Assessment](#professional-self-assessment) (below, on this page)
2. [Code Review]({{ '/code-review/' | relative_url }}): my video walkthrough of the original code and my plan
3. [Enhancement One: Software Design and Engineering]({{ '/software-design/' | relative_url }})
4. [Enhancement Two: Algorithms and Data Structures]({{ '/algorithms-data-structures/' | relative_url }})
5. [Enhancement Three: Databases]({{ '/databases/' | relative_url }})
6. [Code and Branches]({{ '/artifacts/' | relative_url }}): the original and enhanced code on GitHub

## Professional Self-Assessment {#professional-self-assessment}

I am nearing the completion of my Bachelor of Science in Computer Science, and my goal is to work as a software developer while designing and publishing my own mobile games with the Godot engine. I completed the program while serving full time on active duty in the U.S. Air Force, which has strengthened my time management. This portfolio shows what the program prepared me to do. All three enhancements build on one Android app, so a reader can follow a single project from its original weaknesses to a more organized, more secure, and better-tested result.

### How the program shaped my strengths and goals {#strengths-and-goals}

The program gave me breadth. I trained a reinforcement learning agent to solve a maze and analyzed how its reward structure affected whether its policy converged. I programmed an interactive 3D scene in C++ and OpenGL that recreated a physical room from photographs, working with object geometry, lighting, and camera controls. I built an Android weight tracker in Java and SQLite. None of the first two are artifacts in this portfolio, but together with the third they showed me where my interest lies: how a program is organized, not only whether it runs. That is why I am aiming at software engineering and application development, and it is why I value code that other people can read, that is secure by default, and that is tested.

Developing this ePortfolio sharpened those goals. Going back to my own CS 360 code with a critical eye, I found plain-text passwords, screens that talked straight to the database, and hardcoded dates. Those are the kinds of flaws an employer expects a developer to catch, and fixing them taught me more than writing the app the first time. It also left me with something I can show: a public repository with the original and each enhanced version, tests, and narratives, so I can walk an interviewer through my decisions.

### Collaborating in a team environment {#collaboration}

My capstone work was individual, so I focused on making it easy for others to join. I recorded a code review for a peer or manager audience, wrote comments and documentation so another developer could continue the project, kept the original and each enhancement on separate Git branches, and wrote tests that a teammate can run in seconds. Outside of coding, I led a team of four in the Air Force to analyze how people used our unit's SharePoint site to request Public Address (PA) equipment, such as speakers and microphones. We started by mapping out the main questions the request form needed to answer. I then built the form myself, with questions such as the event name, the date, the number of attendees, and the highest-ranking person attending, and I set up SharePoint to add each request and its answers to a calendar where our members could review and accept or deny it. Because most of my teammates did not know SharePoint, I did most of the SharePoint work while the team worked on the analysis. That experience taught me that leading a team means noticing who has which skills and covering the gaps, and that a form is only as useful as the questions it asks, which is the same thinking behind choosing what data an app collects.

### Communicating with stakeholders {#communication}

In this program my stakeholders were my instructor and my peers. I explained my plans and findings in a code review video, wrote a narrative for each enhancement for both technical and non-technical readers, and used feedback to change the work. After my Module One plan, my instructor pointed out that assigning old weight entries to the earliest user during a database migration could attach one person's data to another account. I changed my design to keep those entries in a separate legacy table instead. Listening, restating the requirement, and adjusting the design is the process I expect to follow with clients and teammates.

### Data structures and algorithms {#algorithms}

For the Weight Tracker's history screen, I loaded entries into an ArrayList and used Comparators to sort them (O(n log n)), a single-pass filter for date ranges (O(n)), and a HashMap to group entries by week (O(n) on average). I chose each structure for the question it answers, and I weighed doing the work in memory against letting SQL do it. In my reinforcement learning project I reasoned about algorithms in a different way, asking how an agent's rewards shape what it learns.

### Software engineering and databases {#engineering-databases}

I separated the screens from the data with a repository class, tracked the logged-in user with a session manager, and kept logic in plain Java classes that I could unit test. On the database side, I redesigned the schema with foreign keys and unique constraints, wrote a migration that upgrades an existing database without losing or misassigning data, and used parameterized queries. I test as I build: the project has 37 unit tests and a manual checklist for each enhancement.

### Security {#security}

I treat security as part of the design, not a final step. In the Weight Tracker I replaced plain-text passwords with salted PBKDF2 hashes, separated every user's data at the database level, validated input before using it, and made the data layer refuse to delete another user's entries. I also learned to state limits honestly. The app still has no password reset, and entries saved before accounts owned their data are kept but not shown to anyone.

### Skills at a glance {#skills}

| **Technical skills** | **Professional skills** |
|---|---|
| Java and Android development | Clear written and spoken communication |
| SQL and SQLite | Attention to detail |
| C++ and OpenGL | Problem solving |
| Data structures and algorithm analysis | Time management |
| Secure coding (hashing, validation) | Applying feedback |
| Unit testing with JUnit | |
| Git and GitHub | |
| Reinforcement learning basics | |

### Looking ahead {#looking-ahead}

I plan to keep building. I want to publish mobile games with Godot, and I intend to bring the same habits to that work: structure first, tests alongside, and security designed in. Two trends I studied in this course, autonomous AI and virtual and augmented reality, both connect to games, so I expect to keep learning in those directions.

## How my artifacts fit together {#artifacts-fit-together}

All three enhancements improve one app, the Weight Tracker, and each one builds on the one before it. The first gave the app a clean structure and a way to know who is logged in. The second used that structure and the dates it stored to add sorting, filtering, and summaries. The third used the structure again to give every user their own protected data.

| **Category** | **What I did** | **Page** |
|---|---|---|
| Software design and engineering | Repository class, session handling, date picker, tests | [Enhancement One]({{ '/software-design/' | relative_url }}) |
| Algorithms and data structures | ArrayList, Comparators, date filter, HashMap weekly summaries | [Enhancement Two]({{ '/algorithms-data-structures/' | relative_url }}) |
| Databases | Schema redesign, password hashing, validation, safe migration | [Enhancement Three]({{ '/databases/' | relative_url }}) |

### Where each course outcome is shown {#outcomes}

| **Course outcome** | **Where it is shown** |
|---|---|
| 1. Collaborative environments | Code comments and documentation, the code review video, separate Git branches, and the collaboration example above |
| 2. Professional communication | The code review video, the narratives, and this site |
| 3. Algorithmic design and trade-offs | Enhancement Two |
| 4. Well-founded techniques and tools | All three enhancements |
| 5. Security mindset | Enhancement Three, with the session design in Enhancement One |


[Back to top](#top)
