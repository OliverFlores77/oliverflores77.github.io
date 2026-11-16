---
layout: default
title: "Enhancement Two: Algorithms and Data Structures"
permalink: /algorithms-data-structures/
---
<a id="top"></a>

{% include nav.html %}

# Enhancement Two: Algorithms and Data Structures

**At a glance**

- **Artifact:** Weight Tracker, an Android app written in Java with SQLite (built in CS 360)
- **Code:** [enhancement-2 branch]({{ site.enhancement2_url }}), compared with the [original code]({{ site.original_url }})
- **Course outcomes:** 3 and 4
- **Evidence:** 22 unit tests and a 16-step manual checklist, all passing
- **Word version of the narrative:** [Download]({{ '/assets/docs/CS_499_Enhancement_Two_Narrative.docx' | relative_url }})

## Narrative {#narrative}

### 1. The Artifact {#artifact}

The artifact is the Weight Tracker, an Android app I built in CS 360: Mobile Architecture and Programming in {{ site.cs360_term }}. It is written in Java with an SQLite database and was built and tested in Android Studio. A user can create an account, log in, record daily weights, view their history, set a goal weight, and turn on an SMS alert for reaching the goal. This is the same artifact I used for Enhancement One, where I added a repository class, session handling, and a date picker. I made the enhancements described here during CS 499 in fall 2026.

### 2. Why I Included It and How It Improved {#why-included}

I chose to use the Weight Tracker for this category because its weight history is a natural data problem. Before this enhancement, the app showed the history exactly as the database returned it. The order was fixed by a SQL query, there was no data structure in the app that I could work with, and the user had no way to filter entries or see how a week went. My code review also found that the history loaded twice every time the screen opened and that each refresh built a new adapter without closing the old database cursor.

The table below summarizes what changed.

| **Area**            | **Before**                                                                          | **After**                                                                                                                                         |
|---------------------|-------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| **Data in the app** | A database cursor went straight into the grid.                                      | The repository loads an ArrayList of WeightEntry objects, and the cursor is closed right away.                                                    |
| **Sorting**         | Fixed in SQL, newest first.                                                         | Comparators let the user switch between newest first and oldest first. Entries on the same day keep the order they were saved.                    |
| **Filtering**       | None.                                                                               | A one-pass filter keeps entries between two chosen dates, inclusive. Dates picked in the wrong order are swapped, and an empty result is handled. |
| **Summaries**       | None.                                                                               | A HashMap groups entries by week (Monday through Sunday), and each week shows its entry count and average weight.                                 |
| **Efficiency**      | Loaded twice on startup, with a new adapter and an unclosed cursor on each refresh. | Loaded once each time the screen resumes, with one reusable adapter.                                                                              |
| **Testing**         | Five unit tests for the date helper.                                                | 22 unit tests (10 for the date helper, 12 for the new logic) and a 16-step manual checklist, all passing.                                         |

The components that best show my skills are in a class called WeightAnalyzer. Sorting with a Comparator costs O(n log n), filtering with a single loop costs O(n), and grouping with a HashMap costs O(n) on average and O(n) extra space, where n is the number of entries. Sorting the weekly summaries afterward costs O(w log w), where w is the number of weeks, which is much smaller than n. I chose each structure for the job it does. An ArrayList keeps entries in a displayable order, and a HashMap collects entries by key in one pass without searching a list each time.

The design involved one main trade-off. The database could sort the entries itself, but doing it in the app lets the user change the view without re-querying the database, and it let me build and analyze these structures directly. The cost is holding every entry in memory, which is reasonable for a personal weight log. I also kept all of the logic in plain Java classes with no Android code, so it could be tested quickly on my computer without an emulator.

### 3. Course Outcomes {#outcomes}

In Module One I planned for this enhancement to support Outcomes 3 and 4.

**Outcome 3 (algorithmic design and trade-offs):** I believe I met this. I chose a data structure for each task, wrote the sorting, filtering, and grouping logic, analyzed the time and space cost of each, and weighed the in-memory approach against letting SQL do the work. I also handled edge cases such as an empty history, a range with no entries, reversed dates, and entries with a malformed date, which are skipped instead of crashing the summary.

**Outcome 4 (techniques and tools):** I believe I met this as well. I wrote a custom adapter to replace the cursor adapter, used date pickers for the filter, and tested the logic with unit tests and a manual checklist.

I have two notes on my plans. First, I changed one design detail from the Module One plan: instead of keying each week by year and week number, I key it by the date of the week’s Monday. Week numbers can be ambiguous around New Year’s Day, and the Monday date reads clearly on screen. I wrote tests for the year boundary to check it. Second, Enhancement Three still follows my plan, including my instructor’s advice to keep old weight records in a separate table instead of assigning them to a user. I would also like to save the goal weight per user there, which I will confirm with my instructor first.

### 4. Reflection {#reflection}

The biggest lesson was how much an earlier decision helped. In Enhancement One I stored dates as zero-padded text, so that plain string comparison matches calendar order. That made sorting and filtering simple, because I could compare dates directly without converting them. Matching the data structure to the question also became clearer to me. A list answers “what order do I show these in,” and a HashMap answers “which group does this belong to.” Trying to do both with one structure would have been slower and harder to read.

Several details took more thought than I expected. Two entries on the same day need a second sorting rule, or their order is arbitrary, so I used the order they were saved. Week boundaries needed care, which is why I tested a Sunday, a Monday, and a week that crosses into a new year. Keeping the filter, the sort order, and the full list of entries as separate pieces of state meant that deleting an entry or changing the sort never lost the user’s current view. I also had to decide what the filter should do when nothing matches, and I chose to show a clear message and an empty list instead of leaving the screen unchanged. Replacing the cursor adapter with my own adapter was a larger change than the sorting itself, since each row still needs a working delete button.

Some limitations remain. The sort order and filter reset when the user logs out or rotates the screen, all users still share the same entries, the goal weight resets when the app closes, and a lone decimal point as a weight still crashes the app. The shared entries and the crash are addressed by the Databases enhancement, and I have listed each one in my test checklist.

## Key code {#key-code}

Filtering keeps the entries between two dates, in a single pass over the list. Text dates in yyyy-MM-dd form compare correctly, so no conversion is needed.

```java
/**
 * Returns a NEW list holding only the entries dated from startDate through
 * endDate, inclusive. The original list is not changed. O(n): one pass.
 *
 * If the two dates are given in the wrong order they are swapped, so a range
 * is never "empty" just because the user picked the end date first.
 * If nothing falls in the range, the result is an empty list (never null).
 */
public static List<WeightEntry> filterByDateRange(List<WeightEntry> entries,
                                                  String startDate, String endDate) {
    if (startDate.compareTo(endDate) > 0) {
        String swap = startDate;
        startDate = endDate;
        endDate = swap;
    }

    List<WeightEntry> result = new ArrayList<>();
    for (WeightEntry entry : entries) {
        String date = entry.getDate();
        if (date.compareTo(startDate) >= 0 && date.compareTo(endDate) <= 0) {
            result.add(entry);
        }
    }
    return result;
}
```

Grouping by week uses a HashMap keyed by each week's Monday, so every entry is placed in one step.

```java
/**
 * Groups entries by week (Monday through Sunday) and returns one summary per
 * week, newest week first.
 *
 * A HashMap keyed by the week's starting Monday collects each week's entries in a
 * single pass, so grouping is O(n) on average. Sorting the (much smaller) list of
 * weeks afterward costs O(w log w), where w is the number of weeks.
 * Entries whose date is not in yyyy-MM-dd form are skipped.
 */
public static List<WeekSummary> summarizeByWeek(List<WeightEntry> entries) {
    Map<String, List<WeightEntry>> weeks = new HashMap<>();

    for (WeightEntry entry : entries) {
        if (!DateHelper.isValidIsoFormat(entry.getDate())) {
            continue;
        }
        String key = DateHelper.weekStart(entry.getDate());
        List<WeightEntry> group = weeks.get(key);
        if (group == null) {
            group = new ArrayList<>();
            weeks.put(key, group);
        }
        group.add(entry);
    }

    List<WeekSummary> summaries = new ArrayList<>();
    for (Map.Entry<String, List<WeightEntry>> week : weeks.entrySet()) {
        double total = 0;
        for (WeightEntry entry : week.getValue()) {
            total += entry.getWeight();
        }
        int count = week.getValue().size();
        summaries.add(new WeekSummary(week.getKey(), count, total / count));
    }

    Collections.sort(summaries, NEWEST_WEEK_FIRST);
    return summaries;
}
```

The helper that finds a week's Monday:

```java
/**
 * Returns the Monday that starts the week containing the given date, as yyyy-MM-dd.
 * Weeks run Monday through Sunday. Naming a week by its Monday avoids the
 * confusion of week numbers around New Year's Day.
 *
 * @param isoDate a date in yyyy-MM-dd form
 * @throws IllegalArgumentException if the text is not in yyyy-MM-dd form
 */
public static String weekStart(String isoDate) {
    if (!isValidIsoFormat(isoDate)) {
        throw new IllegalArgumentException("Expected yyyy-MM-dd but got: " + isoDate);
    }
    int year = Integer.parseInt(isoDate.substring(0, 4));
    int month = Integer.parseInt(isoDate.substring(5, 7));
    int day = Integer.parseInt(isoDate.substring(8, 10));

    Calendar date = Calendar.getInstance();
    date.clear();
    date.set(year, month - 1, day);

    // Calendar numbers Sunday=1 ... Saturday=7. This turns that into
    // "days since Monday": Monday=0, Tuesday=1, ... Sunday=6.
    int daysSinceMonday = (date.get(Calendar.DAY_OF_WEEK) + 5) % 7;
    date.add(Calendar.DAY_OF_MONTH, -daysSinceMonday);

    return fromPicker(
            date.get(Calendar.YEAR),
            date.get(Calendar.MONTH),
            date.get(Calendar.DAY_OF_MONTH));
}
```

Files on the branch: [WeightAnalyzer.java]({{ site.repo_url }}/blob/enhancement-2/{{ site.source_path }}/WeightAnalyzer.java), [WeightEntry.java]({{ site.repo_url }}/blob/enhancement-2/{{ site.source_path }}/WeightEntry.java), [WeekSummary.java]({{ site.repo_url }}/blob/enhancement-2/{{ site.source_path }}/WeekSummary.java), [WeightEntryAdapter.java]({{ site.repo_url }}/blob/enhancement-2/{{ site.source_path }}/WeightEntryAdapter.java), [WeightAnalyzerTest.java]({{ site.repo_url }}/blob/enhancement-2/{{ site.test_path }}/WeightAnalyzerTest.java), and the [manual_test_checklist_enhancement2.md]({{ site.repo_url }}/blob/enhancement-2/docs/manual_test_checklist_enhancement2.md).

## Update after finishing the capstone {#update}

Enhancement Three resolved three of the limitations listed above: each user now sees only their own entries, the goal weight is saved, and a lone decimal point shows a message instead of crashing. The sort order and filter still reset when the screen is recreated. See the [Databases page]({{ '/databases/' | relative_url }}).


[Back to top](#top)
