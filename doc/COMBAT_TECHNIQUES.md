# Combat Techniques in Cataclysm: Dark Days Ahead

This document explains the mechanics behind combat techniques, both from martial arts and weapons.

## How Techniques are Chosen

When a character attacks, the game determines which special techniques are available for that specific attack. This process is handled primarily in the `Character::pick_technique` function (`src/melee.cpp`).

1.  **Gathering Potential Techniques:** The game first compiles a list of all possible techniques. This list is sourced from two places:
    *   The character's currently wielded weapon.
    *   The character's currently active martial arts style.

2.  **Evaluating Techniques:** Each technique in this initial list is then evaluated to see if it can be used in the current situation. This evaluation (`Character::evaluate_technique`) checks several criteria:
    *   **Basic Requirements:** Does the character meet the minimum skill levels (e.g., `unarmed` or `melee`)? Does the weapon belong to an allowed category?
    *   **Situational Triggers:** Is the technique only for critical hits (`crit_tec`)? Is it a counter-attack that requires a preceding dodge (`dodge_counter`) or block (`block_counter`)? The technique will only be considered if the situation matches its trigger.
    *   **Conditional Logic:** Many techniques have a `condition` block defined in their JSON files (`data/json/techniques.json`). This allows for complex rules, such as checking the target's size, species, or active status effects (like "downed" or "stunned").

## What Happens if Both Weapon and Martial Art Techniques Qualify?

There is no inherent priority given to weapon techniques over martial art techniques, or vice-versa. If a character is using a weapon that is compatible with their martial art style, techniques from both the weapon and the style are added to a single pool of potential moves for that attack.

The final selection from this pool is random, but it's weighted.

## What Determines the Likelihood a Technique Will Trigger?

The probability of a specific technique being chosen from the pool of valid moves is determined by its `weighting` value, found in `data/json/techniques.json`.

*   **Positive Weighting:** A technique with a higher `weighting` is more likely to be chosen. For example, a technique with a `weighting` of 3 will be added to the selection pool three times, while a technique with a `weighting` of 1 is added only once. This makes the former three times as likely to be picked.

*   **Negative Weighting:** A negative `weighting` of `-X` acts as a preliminary check. The technique only has a 1-in-X chance of being considered for use in the first place. If it passes this roll, it's added to the pool as if it had a weighting of 1.

*   **Default Weighting:** If no weighting is specified, it defaults to 1.

## What Happens if a Technique Does Not Have Its Criteria Satisfied?

If a technique fails any of its requirement checks during the evaluation phase, it is simply discarded from the pool of potential techniques for that specific attack. It will not be used.

If no special techniques qualify for the attack, the character will perform a standard melee attack.