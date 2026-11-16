---
layout: default
title: "Enhancement Three: Databases"
permalink: /databases/
---
<a id="top"></a>

{% include nav.html %}

# Enhancement Three: Databases

**At a glance**

- **Artifact:** Weight Tracker, an Android app written in Java with SQLite (built in CS 360)
- **Code:** [enhancement-3 branch]({{ site.enhancement3_url }}), compared with the [original code]({{ site.original_url }})
- **Course outcomes:** 5 and 4
- **Evidence:** 37 unit tests and a 35-step manual checklist, all passing
- **Word version of the narrative:** [Download]({{ '/assets/docs/CS_499_Enhancement_Three_Narrative.docx' | relative_url }})

## Narrative {#narrative}

### 1. The Artifact {#artifact}

The artifact is the Weight Tracker, an Android app I built in CS 360: Mobile Architecture and Programming in {{ site.cs360_term }}. It is written in Java with an SQLite database and was built and tested in Android Studio. A user can create an account, log in, record daily weights, view and analyze their history, set a goal weight, and turn on an SMS alert for reaching the goal. This is the same artifact I used for Enhancements One and Two. I made the enhancements described here during CS 499 in fall 2026.

### 2. Why I Included It and How It Improved {#why-included}

The database was the weakest part of the original app, which made it a good fit for this category. My code review found that passwords were stored as plain text, weight entries were not linked to users so every account saw every entry, usernames did not have to be unique, a lone decimal point crashed the app, and the upgrade method dropped every table and erased all data whenever the database version changed. I also found that the app’s manifest allowed the database to be included in device backups, which made the plain-text passwords worse.

The table below summarizes what changed.

| **Area** | **Before** | **After** |
|---|---|---|
| **Passwords** | Stored as plain text. | Stored as a salted PBKDF2 hash (SHA-256, a random salt for each account, 120,000 rounds). |
| **Data ownership** | No link between weights and users, so every account saw every entry. | Each weight has a user_id foreign key, and every query returns only the logged-in user’s entries. |
| **Usernames** | Repeats allowed, and failures were silent. | A UNIQUE rule, with a clear “username is already taken” message. |
| **Input** | A lone decimal point crashed the app. | An InputValidator checks accounts, weights (20 to 1,500 lbs), and the goal weight, and shows a message instead of crashing. |
| **Deleting** | Removed any entry by its id. | Removes an entry only if it belongs to the logged-in user. |
| **Goal weight** | Reset to 150 whenever the app closed. | Saved for each user in the database. |
| **Upgrades** | The upgrade method dropped every table and erased all data. | A version-aware migration keeps accounts and old entries. |
| **Testing** | 22 unit tests. | 37 unit tests (15 new, including published PBKDF2 test vectors) and a 35-step manual checklist that includes a migration test on seeded old data. |

The part that best shows my database skills is the schema and the migration. The new schema links each weight entry to its owner with a foreign key and requires unique usernames. For upgrades, I wrote a migration that sets the old weight entries aside in a separate table, copies each account into a new users table with its password converted to a salted hash, and skips repeated usernames, keeping the first one. The old version never recorded who owned an entry, so I did not assign old entries to anyone.

For passwords, each account gets a random salt and a PBKDF2 hash with SHA-256, which makes stolen hashes slow to guess, and logins compare hashes in a way that does not reveal how close a wrong guess was. Every query that uses a username or user id uses placeholders instead of pasting text into SQL, and the database methods return clear results, such as telling the user a username is taken, instead of failing silently. These changes showcase relational schema design, safe data migration, secure credential storage, input validation, and testing.

### 3. Course Outcomes {#outcomes}

In Module One I planned for this enhancement to support Outcomes 5 and 4.

**Outcome 5 (security mindset):** I believe I met this. I tried to anticipate how someone could attack or misuse the app: stealing the database file or a backup, reading another user’s entries, entering bad data, or deleting another user’s entries. The design answers each one with hashed passwords, ownership enforced in every query, input validation, and a delete that only works on the logged-in user’s own entries.

**Outcome 4 (techniques and tools):** I believe I met this as well. I designed a relational schema, wrote and tested a versioned migration, and checked my work with unit tests and a manual checklist.

I made two changes to my Module One plan. First, my instructor advised against assigning existing weight records to the earliest user during the migration, since that could attach one person’s data to another account. I changed the design to keep the old records in a separate legacy table, where nothing reads them. Second, I added saving the goal weight for each user, which was not in my original plan. I also made one small design change: deleting an entry now requires the user’s id, so the database enforces ownership instead of relying on the screen.

### 4. Reflection {#reflection}

The most important lesson was that a migration is where data is most likely to be lost or mixed up, so it needs the most care. SQLite runs an upgrade as one transaction, so if a step fails, the old database stays untouched. The order of the steps matters too. I created the new weights table last, after the users table had its final form, so no foreign key ever pointed at a table that was being replaced. To test it, I installed the old version, created accounts (including a repeated username), added entries, and ran the new version over it.

I also learned how much a small design decision can help later. Because the repository and session code from Enhancement One already passed a user id around, the screens changed very little: the login messages, the input checks, the goal weight, and one delete call. The database layer did the heavy work. For hashing, I learned the reason behind each choice, a salt so identical passwords look different and a slow function so guessing is expensive, and I checked my implementation against published PBKDF2 test vectors, which gave me independent proof that it is correct.

The hardest part was deciding what not to guess. The old data had no owner and some usernames repeated, and every shortcut risked putting someone’s data in the wrong account, so I chose to keep and set aside instead of assuming. Some limitations remain: there is no way to change or reset a password, old entries from before accounts owned their data are kept but not shown to anyone, usernames are case-sensitive, the sort order and filter reset when the screen is recreated, and the password hashing needs Android 8.0 or newer. If I continued the project, those would be my next steps.

## Key code {#key-code}

Logging in looks the user up by username with a placeholder, then hashes the typed password with that user's salt and compares it with the stored hash. The password is never put into the SQL.

```java
/**
 * Checks a username and password. The password is never put into SQL: the user is
 * looked up by username (with a ? placeholder), then the typed password is hashed
 * with that user's salt and compared with the stored hash.
 *
 * @return the user's _id, or NO_USER if the credentials do not match
 */
public int getUserId(String username, String password) {
    Cursor cursor = null;
    try {
        SQLiteDatabase db = this.getReadableDatabase();
        String query = "SELECT " + COL_USER_ID + ", " + COL_SALT + ", " + COL_PASSWORD_HASH +
                " FROM " + TABLE_USERS + " WHERE " + COL_USERNAME + " = ?";
        cursor = db.rawQuery(query, new String[]{username});
        if (cursor.moveToFirst()) {
            String salt = cursor.getString(1);
            String storedHash = cursor.getString(2);
            if (PasswordHasher.verify(password, salt, storedHash)) {
                return cursor.getInt(0);
            }
        }
    } catch (SQLException e) {
        Log.e(TAG, "Could not look up user", e);
    } finally {
        if (cursor != null) {
            cursor.close();
        }
    }
    return NO_USER;
}
```

The hash and the comparison:

```java
// Package-private so unit tests can check the algorithm against published
// test vectors, which use a small iteration count.
static String hash(String password, String saltHex, int iterations) {
    PBEKeySpec spec = new PBEKeySpec(password.toCharArray(), fromHex(saltHex), iterations, KEY_BITS);
    try {
        SecretKeyFactory factory = SecretKeyFactory.getInstance(ALGORITHM);
        return toHex(factory.generateSecret(spec).getEncoded());
    } catch (NoSuchAlgorithmException | InvalidKeySpecException e) {
        throw new IllegalStateException("Password hashing is not available on this device", e);
    } finally {
        spec.clearPassword();
    }
}
```

```java
/**
 * Checks a password against a stored salt and hash.
 * The comparison takes the same time whether or not the hashes match,
 * so timing cannot be used to guess a hash.
 */
public static boolean verify(String password, String saltHex, String expectedHashHex) {
    if (password == null || saltHex == null || expectedHashHex == null) {
        return false;
    }
    String actual = hash(password, saltHex);
    return MessageDigest.isEqual(
            actual.getBytes(StandardCharsets.US_ASCII),
            expectedHashHex.getBytes(StandardCharsets.US_ASCII));
}
```

The migration from the old database to the new one:

```java
/**
 * Upgrades a version 1 database without losing anything and without guessing.
 *
 *  1. Old weight entries are kept, untouched, in legacy_weights. Version 1 never
 *     recorded who owned an entry, so they are NOT assigned to any account. That
 *     way no one's data can end up in someone else's account.
 *  2. Every account is copied into the new users table with its plain-text password
 *     replaced by a salted hash. Ids are kept. If a username appears more than once
 *     (version 1 allowed that), the first account is kept and the later ones are skipped.
 *  3. A new, empty weights table is created that is linked to users.
 */
private void migrateToVersion2(SQLiteDatabase db) {
    // 1. Set the old entries aside.
    db.execSQL("ALTER TABLE " + TABLE_WEIGHTS + " RENAME TO " + TABLE_LEGACY_WEIGHTS);

    // 2. Build the new users table beside the old one, then swap them.
    String newUsersTable = "users_v2";
    db.execSQL(createUsersSql(newUsersTable));

    Cursor oldUsers = db.rawQuery(
            "SELECT " + COL_USER_ID + ", " + COL_USERNAME + ", " + LEGACY_COL_PASSWORD +
                    " FROM " + TABLE_USERS + " ORDER BY " + COL_USER_ID + " ASC", null);
    try {
        while (oldUsers.moveToNext()) {
            String username = oldUsers.getString(1);
            String password = oldUsers.getString(2);
            if (username == null || username.isEmpty() || password == null) {
                continue; // not a usable account
            }
            String salt = PasswordHasher.generateSalt();

            ContentValues values = new ContentValues();
            values.put(COL_USER_ID, oldUsers.getInt(0));
            values.put(COL_USERNAME, username);
            values.put(COL_SALT, salt);
            values.put(COL_PASSWORD_HASH, PasswordHasher.hash(password, salt));

            // CONFLICT_IGNORE skips a repeated username, so the first account wins.
            db.insertWithOnConflict(newUsersTable, null, values, SQLiteDatabase.CONFLICT_IGNORE);
        }
    } finally {
        oldUsers.close();
    }

    db.execSQL("DROP TABLE " + TABLE_USERS);
    db.execSQL("ALTER TABLE " + newUsersTable + " RENAME TO " + TABLE_USERS);

    // 3. Create the new weights table last, after users is in its final form.
    db.execSQL(CREATE_WEIGHTS_SQL);
    db.execSQL(CREATE_WEIGHTS_INDEX_SQL);
}
```

Files on the branch: [DatabaseHelper.java]({{ site.repo_url }}/blob/enhancement-3/{{ site.source_path }}/DatabaseHelper.java), [PasswordHasher.java]({{ site.repo_url }}/blob/enhancement-3/{{ site.source_path }}/PasswordHasher.java), [InputValidator.java]({{ site.repo_url }}/blob/enhancement-3/{{ site.source_path }}/InputValidator.java), [WeightRepository.java]({{ site.repo_url }}/blob/enhancement-3/{{ site.source_path }}/WeightRepository.java), [PasswordHasherTest.java]({{ site.repo_url }}/blob/enhancement-3/{{ site.test_path }}/PasswordHasherTest.java), [InputValidatorTest.java]({{ site.repo_url }}/blob/enhancement-3/{{ site.test_path }}/InputValidatorTest.java), and the [manual_test_checklist_enhancement3.md]({{ site.repo_url }}/blob/enhancement-3/docs/manual_test_checklist_enhancement3.md).


[Back to top](#top)
