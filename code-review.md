---
layout: default
title: "Code Review"
permalink: /code-review/
---
<a id="top"></a>

{% include nav.html %}

# Code Review

My code review is a recorded walkthrough of the original Weight Tracker code, the problems I found, and the changes I planned. I recorded it with OBS Studio for a peer or manager audience, and it follows the same three steps for each category: what the existing code does, what the checklist reveals, and what I plan to change.

<div style="position:relative; padding-bottom:56.25%; height:0; overflow:hidden; max-width:100%; margin:16px 0;">
  <iframe src="{{ site.code_review_embed }}" title="CS 499 code review video" style="position:absolute; top:0; left:0; width:100%; height:100%; border:0;" allowfullscreen></iframe>
</div>

[Watch the code review on YouTube]({{ site.code_review_video }})

## What I found and what I planned {#findings}

I used the CS 499 code review checklist to review the code. The table shows the main findings in each category and the enhancement each one led to.

| **Category** | **Main findings** | **Planned enhancement** |
|---|---|---|
| Software design and engineering | Every screen created its own database helper and called it directly. The app did not track who was logged in. The entry date, phone number, and goal weight were hardcoded. A leftover demo screen and a copy of it sat in the test folder. Comments were sparse and there were no tests of my own. | A repository class, session handling, a date picker, named constants, dead code removed, comments, and tests |
| Algorithms and data structures | The history was shown exactly as the database returned it. The sort order was fixed in the SQL query. There was no filtering or summary. The data loaded twice at startup, and each refresh built a new adapter without closing the old cursor. | An ArrayList of entries, Comparator sorting, a date-range filter, and HashMap weekly summaries |
| Databases | Passwords were stored as plain text. Weights were not linked to users, so every account saw every entry. Usernames could repeat. A lone decimal point crashed the app. The upgrade method dropped every table. | A new schema with foreign keys, salted password hashing, input validation, and a safe migration |

## How the plan changed {#plan-changes}

The plan I presented in the video changed in three ways as I worked:

1. **Migration.** I planned to assign existing weight entries to the earliest account. My instructor advised against it because it could attach one person's data to another account, so the final migration keeps those entries in a separate legacy table.
2. **Goal weight.** I added saving the goal weight for each user, which was not in the original plan.
3. **Week summaries.** I named each week by its Monday date instead of a year and week number, because week numbers are confusing around New Year's Day.

The three enhancement pages show how each finding was fixed:

- [Enhancement One: Software Design and Engineering]({{ '/software-design/' | relative_url }})
- [Enhancement Two: Algorithms and Data Structures]({{ '/algorithms-data-structures/' | relative_url }})
- [Enhancement Three: Databases]({{ '/databases/' | relative_url }})


[Back to top](#top)
