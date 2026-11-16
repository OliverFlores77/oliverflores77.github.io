---
layout: default
title: "Enhancement One: Software Design and Engineering"
permalink: /software-design/
---
<a id="top"></a>

{% include nav.html %}

# Enhancement One: Software Design and Engineering

**At a glance**

- **Artifact:** Weight Tracker, an Android app written in Java with SQLite (built in CS 360)
- **Code:** [enhancement-1 branch]({{ site.enhancement1_url }}), compared with the [original code]({{ site.original_url }})
- **Course outcomes:** 4, 2, and 1
- **Evidence:** 5 unit tests and a 17-step manual checklist, all passing
- **Word version of the narrative:** [Download]({{ '/assets/docs/CS_499_Enhancement_One_Narrative.docx' | relative_url }})

## Narrative {#narrative}

### 1. The Artifact {#artifact}

The artifact is the Weight Tracker, an Android app I built in CS 360: Mobile Architecture and Programming in {{ site.cs360_term }}. It is written in Java with an SQLite database, and I built and tested it in Android Studio on an emulator. A user can create an account, log in, record daily weights, view their history in a grid, set a goal weight, and turn on an SMS alert for when the goal is reached. I made the enhancements described here during CS 499 in fall 2026.

### 2. Why I Included It and How It Improved {#why-included}

My instructor approved using the Weight Tracker for all three enhancement categories. I chose it because one app lets me show software design, data handling, and security as connected work, and because mobile apps are the kind of development I want to do as a career, working toward publishing my own mobile games. It also gave me real problems to solve. My code review found that every screen talked straight to the database, the app did not know who was logged in, and the entry date was hardcoded, along with several other hardcoded values and some leftover code.

The table below summarizes what changed.

| **Area**            | **Before**                                                                    | **After**                                                                                                                      |
|---------------------|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| **Database access** | Each screen created its own DatabaseHelper.                                   | All screens call WeightRepository, the only class that touches the database.                                                   |
| **Logged-in user**  | Not tracked.                                                                  | SessionManager saves the user’s id. Screens send the user back to login if nobody is logged in, and logout clears the session. |
| **Login**           | Returned only true or false.                                                  | Returns the user’s id, using a parameterized query.                                                                            |
| **Entry date**      | Hardcoded as 2026-06-13.                                                      | A date picker defaults to today and blocks future dates. DateHelper stores dates as zero-padded yyyy-MM-dd.                    |
| **Code quality**    | Hardcoded numbers, a dead MainActivity, a leftover note, and sparse comments. | Named constants, dead code removed, and comments on every changed class.                                                       |
| **Testing**         | None.                                                                         | Five JUnit tests for DateHelper (all pass) and a 17-step manual checklist (all pass on a Pixel 10 emulator).                   |

These changes showcase several skills: separation of concerns, managing state across screens, handling user input, writing readable and documented code, and testing as I build. I also designed the changes with the next two enhancements in mind. The repository methods already accept a user id, so when the Databases enhancement adds ownership to the weights table, the database layer can change without rewriting the screens.

### 3. Course Outcomes {#outcomes}

In Module One I planned for this enhancement to support Outcomes 4, 2, and 1. Here is where I think each one stands:

**Outcome 4 (techniques and tools):** I believe I met this for this enhancement. I used a repository pattern, SharedPreferences for session state, and a date picker, and I checked my work with both unit tests and a manual checklist.

**Outcome 2 (communication):** This is in progress. I wrote comments and documentation for each changed class, and this narrative and the test checklist explain the work, but I expect to refine them with feedback.

**Outcome 1 (collaboration):** This is partly met. I wrote the code and comments so another developer could pick up my work and continue it, but I will cover the rest of this outcome in my professional self-assessment.

I have two updates to my plans. First, my instructor advised against assigning existing weight records to the earliest user during the database migration, since that could attach one person’s data to another account. In Enhancement Three, I will keep old records in a separate legacy table instead. Second, I would like to save the goal weight per user in Enhancement Three, and I will confirm that addition with my instructor first. I also went slightly beyond the written plan by adding named constants and removing dead code, because both answered findings from my code review.

### 4. Reflection {#reflection}

The biggest lesson was designing for the next enhancement, not just the current one. Putting the user id into the repository methods before the database could use it meant I did not have to rework the screens later. My testing also taught me something I had blurred together. The session works, so the app knows who is logged in, but user1 and user2 still see each other’s weights, because the weights table has no owner column. Knowing who the user is and having the database enforce who owns what are separate problems, and the second one belongs to the Databases enhancement.

I also learned several practical details. Android’s date picker counts months from zero, so January is 0. Zero-padding dates matters because text sorting only matches calendar order when every date has the same format, and the sorting and filtering in Enhancement Two depend on it. Keeping the date logic in plain Java made it testable on my computer without an emulator. And calling finish on the login screen changed how the Back button behaves, which I confirmed in testing.

My main challenges were with the tools, not the code. The emulator crashed intermittently, my project folder had two copies of the app, and Android Studio reported that no module was specified for my run configuration until I synced Gradle. Each cost me time, but they taught me to check the project setup before assuming the code is wrong. I also had to resist adding extra features, such as input validation and saving the goal weight, which are planned for later. Some limitations remain: all users still share the same entries, the goal weight resets when the app closes, a lone decimal point as a weight still crashes the app, and passwords are still stored as plain text. I have planned each for a later enhancement and listed them in my test checklist.

## Key code {#key-code}

The session manager remembers who is logged in. Only the user's id is stored, never the password.

```java
/**
 * Remembers which user is logged in so every screen knows whose data it is
 * working with.
 *
 * Only the user's database id is stored (never the password). The id is kept in
 * SharedPreferences, which is private to this app.
 *
 * Typical flow:
 *   LoginActivity    -> saveUserId(id) after a successful login
 *   Other Activities -> isLoggedIn() / getUserId() to find the current user
 *   Logout           -> clear()
 */
public class SessionManager {

    /** Returned by getUserId() when nobody is logged in. */
    public static final int NO_USER = -1;

    private static final String PREFS_NAME = "weight_tracker_session";
    private static final String KEY_USER_ID = "user_id";

    private final SharedPreferences prefs;

    public SessionManager(Context context) {
        // Application context avoids holding on to an Activity.
        prefs = context.getApplicationContext()
                .getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
    }

    /** Saves the id of the user who just logged in. */
    public void saveUserId(int userId) {
        prefs.edit().putInt(KEY_USER_ID, userId).apply();
    }

    /** Returns the logged-in user's id, or NO_USER if nobody is logged in. */
    public int getUserId() {
        return prefs.getInt(KEY_USER_ID, NO_USER);
    }

    /** True when a user is currently logged in. */
    public boolean isLoggedIn() {
        return getUserId() != NO_USER;
    }

    /** Ends the session (logout). */
    public void clear() {
        prefs.edit().remove(KEY_USER_ID).apply();
    }
}
```

The date helper turns the date picker's values into a text date that sorts correctly. The picker counts months from zero, so one is added.

```java
/**
 * Builds a yyyy-MM-dd string from the values Android's DatePickerDialog and
 * Calendar give us. Both count months from 0 (January) to 11 (December),
 * so this adds 1 before formatting.
 */
public static String fromPicker(int year, int monthZeroBased, int day) {
    return format(year, monthZeroBased + 1, day);
}
```

Files on the branch: [WeightRepository.java]({{ site.repo_url }}/blob/enhancement-1/{{ site.source_path }}/WeightRepository.java), [SessionManager.java]({{ site.repo_url }}/blob/enhancement-1/{{ site.source_path }}/SessionManager.java), [DateHelper.java]({{ site.repo_url }}/blob/enhancement-1/{{ site.source_path }}/DateHelper.java), [DateHelperTest.java]({{ site.repo_url }}/blob/enhancement-1/{{ site.test_path }}/DateHelperTest.java), and the [manual_test_checklist.md]({{ site.repo_url }}/blob/enhancement-1/docs/manual_test_checklist.md).

## Update after finishing the capstone {#update}

The four limitations listed above were all resolved in Enhancement Three: each user now sees only their own entries, the goal weight is saved for each user, a lone decimal point shows a message instead of crashing, and passwords are stored as salted hashes. See the [Databases page]({{ '/databases/' | relative_url }}).


[Back to top](#top)
