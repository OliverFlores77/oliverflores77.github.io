---
layout: default
title: "Code and Branches"
permalink: /artifacts/
---
<a id="top"></a>

{% include nav.html %}

# Code and Branches

All of my code is in one GitHub repository. Each stage of the project is on its own branch, so you can see the app before and after each enhancement.

| **Branch** | **What it contains** | **Link** |
|---|---|---|
| Original | The Weight Tracker as I built it in CS 360 | [View the original code]({{ site.original_url }}) |
| enhancement-1 | The original plus Enhancement One (software design and engineering) | [View enhancement-1]({{ site.enhancement1_url }}) |
| enhancement-2 | Enhancement One plus Enhancement Two (algorithms and data structures) | [View enhancement-2]({{ site.enhancement2_url }}) |
| enhancement-3 | All three enhancements, the finished app | [View enhancement-3]({{ site.enhancement3_url }}) |

Each enhancement branch starts from the one before it, so enhancement-3 is the complete, final version. The [full repository]({{ site.repo_url }}) is the place to browse every file.

## About the app {#about-the-app}

The Weight Tracker is an Android app written in Java with an SQLite database. A user creates an account, logs in, records weights by date, sees their history, and sets a goal weight. The final version also lets a user sort the history, filter it by date, and see weekly summaries.

## Running it {#running-it}

1. Clone the repository and switch to the branch you want, for example `git checkout enhancement-3`.
2. Open the folder in Android Studio and let Gradle sync.
3. Run the app on an emulator. Android 8.0 or newer is recommended, because the final version's password hashing uses PBKDF2 with SHA-256.
4. To run the unit tests, right-click the `test` folder in Android Studio and choose Run Tests.

## Narratives as Word documents {#word-documents}

The self-assessment and the narratives are written on the pages of this site. Word copies are also available:

- [Professional self-assessment]({{ '/assets/docs/CS_499_Professional_Self-Assessment.docx' | relative_url }})
- [Enhancement One narrative]({{ '/assets/docs/CS_499_Enhancement_One_Narrative.docx' | relative_url }})
- [Enhancement Two narrative]({{ '/assets/docs/CS_499_Enhancement_Two_Narrative.docx' | relative_url }})
- [Enhancement Three narrative]({{ '/assets/docs/CS_499_Enhancement_Three_Narrative.docx' | relative_url }})


[Back to top](#top)
