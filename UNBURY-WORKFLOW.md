# Unbury Workflow - GP Test User Removal with WonderPick

This document explains the workflow for the GP Test mode that removes specific users from your friend list while periodically checking WonderPick.

## Overview

The **Unbury** feature removes friends by name from a list you provide, checking WonderPick after every batch of removals. This is useful for cleaning up your friend list while ensuring you don't miss any WonderPick opportunities.

### Key Features
- Removes users by matching names from `remove_users.txt`
- Checks WonderPick **3 times** after every **4 users** removed
- Stops after **40 users** removed (configurable in code)
- Scans names directly from friend list view (no profile clicking needed)
- Re-enters friend list fresh after each removal to avoid list reordering issues

---

## Setup

### 1. Create the Remove List

Create a file called `remove_users.txt` in your PTCGPB root folder (same folder as `Settings.ini`).

Add one username per line:
```
PlayerToRemove1
TargetUser123
AnotherName
VictimX
```

### 2. Start GP Test Mode

1. Run `Main.ahk`
2. Press **Shift+F9** (or click the "GP Test" button)
3. The script will automatically detect `remove_users.txt` and start the removal process

---

## Detailed Workflow

### Phase 1: Initialization

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Load remove_users.txt                                       │
│     • Read all usernames into memory                            │
│     • Display: "Loaded X usernames to remove"                   │
│                                                                 │
│  2. Initialize counters                                         │
│     • totalRemoved = 0                                          │
│     • cycleRemoved = 0                                          │
└─────────────────────────────────────────────────────────────────┘
```

### Phase 2: Main Loop (Repeats for Each Removal)

```
┌─────────────────────────────────────────────────────────────────┐
│  STEP A: Check Exit Conditions                                  │
│  ─────────────────────────────                                  │
│                                                                 │
│  • If totalRemoved >= 40 → DONE (success)                       │
│  • If GPTest toggled off → EXIT                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP B: Check WonderPick Time                                  │
│  ─────────────────────────────                                  │
│                                                                 │
│  If cycleRemoved >= 4:                                          │
│    ┌─────────────────────────────────────┐                      │
│    │  WonderPick Check Loop (3 times)    │                      │
│    │  ─────────────────────────────────  │                      │
│    │  1. Navigate to home screen         │                      │
│    │  2. Click WonderPick button         │                      │
│    │  3. Select available pack           │                      │
│    │  4. Pick a card                     │                      │
│    │  5. Skip through reveal animation  │                      │
│    │  6. Repeat 2 more times             │                      │
│    └─────────────────────────────────────┘                      │
│    Then: cycleRemoved = 0                                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP C: Fresh Entry into Friend List                           │
│  ────────────────────────────────────                           │
│                                                                 │
│  1. Navigate to Social screen (click Social button)             │
│  2. Click "Add" to enter friends list                           │
│  3. Now viewing TOP of friend list                              │
│                                                                 │
│  WHY: After removing a friend, the list may reorder.            │
│       Starting fresh ensures we don't skip anyone.              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP D: Scroll Through List Looking for Match                  │
│  ─────────────────────────────────────────────                  │
│                                                                 │
│  SCROLL LOOP:                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  1. Take screenshot of friend list                      │    │
│  │                                                         │    │
│  │  2. OCR scan 3 visible friend slots:                    │    │
│  │     • Slot 1: Y=175 (click at Y=195)                    │    │
│  │     • Slot 2: Y=270 (click at Y=290)                    │    │
│  │     • Slot 3: Y=365 (click at Y=385)                    │    │
│  │                                                         │    │
│  │  3. For each scanned name:                              │    │
│  │     • Compare against ALL names in remove_users.txt     │    │
│  │     • Use fuzzy matching (75% similarity threshold)     │    │
│  │     • If MATCH → go to STEP E                           │    │
│  │                                                         │    │
│  │  4. No match in view → Scroll down                      │    │
│  │                                                         │    │
│  │  5. Check for end of list:                              │    │
│  │     • Compare current names with previous scan          │    │
│  │     • If SAME → reached bottom, exit loop               │    │
│  │                                                         │    │
│  │  6. Safety: Stop after 50 scrolls                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  If no match found after entire list → DONE (no more matches)   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP E: Remove the Matched Friend                              │
│  ─────────────────────────────────                              │
│                                                                 │
│  1. Click on the friend at their Y position                     │
│  2. Wait for profile to load                                    │
│  3. Click "Remove" button                                       │
│  4. Confirm removal (click "Send2" / OK)                        │
│  5. totalRemoved++                                              │
│  6. cycleRemoved++                                              │
│                                                                 │
│  → Loop back to STEP A (re-enter friend list fresh)             │
└─────────────────────────────────────────────────────────────────┘
```

---

## End of List Detection

The script detects when it has reached the bottom of the friend list by comparing consecutive scans:

```
Scroll 5:  Names = "Alice|Bob|Charlie|"
                        │
                   [SCROLL DOWN]
                        │
                        ▼
Scroll 6:  Names = "David|Eve|Frank|"     ← Different, keep scrolling
                        │
                   [SCROLL DOWN]
                        │
                        ▼
Scroll 7:  Names = "David|Eve|Frank|"     ← SAME! We're at bottom
                        │
                        ▼
              No more friends to check
              Exit scroll loop
```

---

## Name Matching

The script uses **fuzzy matching** to handle OCR errors:

```
FuzzyNameMatch(parsedName, targetName):
  1. Exact match (case insensitive) → MATCH
  2. Similarity score > 75% → MATCH
  3. Otherwise → NO MATCH
```

This allows for minor OCR misreads like:
- `Player123` vs `PIayer123` (I vs l)
- `TargetUser` vs `TargetUser1` (extra character)

---

## Configuration

Edit these values in `RemoveUsersFromListWithWonderPick()` (Main.ahk, line ~987):

```ahk
usersPerCycle := 4          ; Remove this many users before WonderPick
wonderpickChecks := 3       ; Number of WonderPick checks per cycle
maxRemovals := 40           ; Stop after this many total removals
```

---

## Example Run

```
[START]
Loaded 10 usernames to remove.
Target: 40 removals

[Entering friend list...]
Scanning... (scroll 0/50)
  → Found: Alice, Bob, Charlie
  → No matches

Scanning... (scroll 1/50)
  → Found: David, TargetUser, Frank
  → MATCH: TargetUser -> TargetUser
  → Removing... (1/40)

[Entering friend list...] (fresh start)
Scanning... (scroll 0/50)
  → Found: Alice, Bob, Charlie
  → No matches

Scanning... (scroll 1/50)
  → Found: David, Frank, Grace
  → No matches (TargetUser is gone!)

Scanning... (scroll 2/50)
  → Found: VictimX, Henry, Ivan
  → MATCH: VictimX -> VictimX
  → Removing... (2/40)

... (continues) ...

[After 4 removals]
Removed 4 users this cycle.
Checking WonderPick 3 times...
  WonderPick check 1/3 ✓
  WonderPick check 2/3 ✓
  WonderPick check 3/3 ✓

[Entering friend list...] (continue removals)
...

[COMPLETE]
Completed! Removed 40 users.
GP Test finished.
```

---

## Troubleshooting

### "remove_users.txt not found!"
Create the file in your PTCGPB root folder with usernames, one per line.

### "No more matching users in list"
All names in your remove list have been processed, or they don't exist in your friend list.

### OCR not reading names correctly
The OCR regions may need adjustment for your screen scale. Check `ScanFriendListNames()` for the Y positions.

### Script stuck at "Navigating to Social"
The game may be in an unexpected state. Try restarting the game and script.

---

## Files Modified

- `Scripts/Main.ahk` - Added the following functions:
  - `RemoveUsersFromListWithWonderPick()` - Main removal loop
  - `GoToSocialScreen()` - Navigate to Social screen
  - `ScanFriendListNames()` - OCR scan friend list
  - `ParseFriendListName()` - Extract name from screenshot region
  - `FuzzyNameMatch()` - Compare names with tolerance
  - `DoWonderPickCheck()` - Perform one WonderPick
  - `ScrollFriendList()` - Scroll the friend list

---

## Summary

| Step | Action |
|------|--------|
| 1 | Load names from `remove_users.txt` |
| 2 | Enter friend list (fresh) |
| 3 | Scan 3 visible names via OCR |
| 4 | If match found → remove friend → go to step 2 |
| 5 | If no match → scroll down → repeat step 3 |
| 6 | If end of list → done (no more matches) |
| 7 | After every 4 removals → do 3 WonderPick checks |
| 8 | Stop after 40 removals |
