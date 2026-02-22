# Seedance 2.0 Prompt Analysis — Median-Length Prompts

**Dataset:** 102 prompts from `all_prompts.json`  
**Median length:** 369 characters  
**Analysis focus:** 300–500 char prompts (n=8 strict), expanded to 250–550 (n=26) for statistical reliability

---

## Length Distribution

| Range | Count | % |
|-------|-------|---|
| <200 chars | 30 | 29% |
| 200–300 | 21 | 21% |
| **300–500** | **8** | **8%** |
| 500–1000 | 19 | 19% |
| 1000–2000 | 14 | 14% |
| 2000+ | 10 | 10% |

The distribution is bimodal — many short prompts (<200) and many long ones (500+). The median sits in a relatively sparse zone.

---

## 1. Shots/Scenes Per Prompt (n=26, 250–550 range)

| Shot Count | # Prompts | % | Notes |
|---|---|---|---|
| **1 (single continuous shot)** | **15** | **58%** | Most common — one scene, one action |
| 2 shots | 5 | 19% | Usually a setup + payoff |
| 3+ shots | 4 | 15% | Montage/cut-based sequences |
| Explicit multi-shot structure | 2 | 8% | Timestamps `[00:00]` or `Cut 1/2/3` |

**Key finding:** The vast majority (58%) of median-length prompts describe a **single continuous shot** with no scene breaks. Multi-shot prompts at this length are brief per shot.

### Shot identification patterns found:
- `Cut 1 (2s):` / `Cut 2:` — 2 prompts
- `[00:00-00:05]` timestamps — 0 in median range (only in longer prompts)
- `Cut to` / `cuts to` — 3 prompts
- Implicit scene breaks (new subject/action) — 5 prompts
- `Panel` references — 1 prompt (storyboard-based)

---

## 2. Words Per Shot

| Type | Avg Words/Shot |
|---|---|
| Single-shot prompts | 50–70 words (entire prompt) |
| Multi-shot prompts (2 shots) | 25–35 words per shot |
| Multi-shot prompts (3+ shots) | 12–20 words per shot |

**Multi-shot prompts sacrifice detail per shot.** A 2-shot prompt at ~370 chars gives ~30 words per shot — enough for scene + one action. A 3+ shot prompt gives only ~15 words per shot — essentially shorthand.

---

## 3. Content Element Breakdown (n=26)

| Element | % of Prompts | Typical Word Allocation |
|---|---|---|
| **Action/event sequence** | **92%** | 40–60% of words |
| **Scene/environment** | **81%** | 15–25% of words |
| **Character/actor description** | **69%** | 10–20% of words |
| **Camera/cinematography** | **65%** | 10–15% of words |
| **Style tags** | **42%** | 5–10% of words |
| Sound/audio cues | **15%** | 1–5% of words |
| Dialogue/lip-sync | **19%** | 5–15% of words |

### Detail by category:

**Action/event (92%):** Nearly universal. Even "atmospheric" prompts include movement — stalking, walking, transforming, flooding. Prompts without action are rare style-guide fragments.

**Scene/environment (81%):** Usually 1–2 phrases: "dusty abandoned warehouse", "frozen exoplanet covered in endless glaciers", "empty desert". Time of day mentioned in ~40%.

**Character/actor (69%):** Often minimal — "a woman", "an astronaut", "a sailor". Detailed character descriptions (hair color, clothing, expression) appear in ~30% of prompts.

**Camera/cinematography (65%):** Very common at this length. Includes: "first-person POV", "slow cinematic shot", "handheld camera", "tracking shot", "crane shot", "fisheye lens", "whip-pan".

**Style tags (42%):** "Ultra-realistic", "cinematic", "4K", "live-action", "high-quality anime". Usually 2–5 words.

**Dialogue (19%):** Present mainly in character-interaction prompts. Usually short quoted speech.

**Sound/audio (15%):** Rare. When present: "mechanical gear sounds", "heavy breathing", "theme plays in background".

---

## 4. Primary Focus — What Gets the MOST Words?

| Primary Focus | % of Prompts |
|---|---|
| **Action/event sequence** | **58%** |
| Scene description | 19% |
| Actor/character description | 15% |
| Camera/style | 8% |

**Action dominates.** The median-length prompt is fundamentally an **event description** — what happens in the scene. Scene and character are supporting elements that establish context for the action.

---

## 5. Five Annotated Examples

### Example 1 (369 chars) — Action-dominant, multi-shot
```
Live-action cinematic sequence.                          [STYLE]
The woman finishes pumping gas.                          [ACTION]
The pump clicks off. Silence.                            [ACTION + SOUND]
She looks around at the empty desert.                    [ACTION + SCENE]
A car appears on the horizon. She squints.               [ACTION]
The car speeds past without stopping. Dust cloud.        [ACTION + SCENE]
She walks to her trunk and opens it.                     [ACTION]
Cut to inside: bags of cash and a gun.                   [CAMERA + SCENE]
She slams it shut and mutters:                           [ACTION]
"Not far enough yet".                                    [DIALOGUE]
She drives off.                                          [ACTION]
```
**Breakdown:** 70% action, 10% scene, 10% dialogue, 5% camera, 5% style

### Example 2 (395 chars) — Single continuous shot, action + scene
```
The camera plunges forward through a flooding            [CAMERA + SCENE]
submarine corridor, closely following a sailor           [ACTOR]
desperately sealing bulkhead doors amid torpedo          [ACTION]
impacts. Water pours violently through cracks,           [ACTION + SCENE]
electronics spark dangerously. The sailor navigates      [ACTION]
floating debris, leaps between slippery decks,           [ACTION]
and manually activates an emergency ballast valve        [ACTION]
just as the flooding stops inches from critical          [ACTION]
equipment.
```
**Breakdown:** 65% action, 20% scene, 10% camera, 5% actor

### Example 3 (503 chars) — Transformation sequence, actor-heavy
```
A lone astronaut in a standard spacesuit                 [ACTOR]
lands on a frozen exoplanet covered in endless           [SCENE]
glaciers under a dim blue sun.                           [SCENE]
Slow cinematic shot:                                     [CAMERA]
he removes his helmet, cold vapor fills the air,         [ACTION]
his skin gradually turns translucent and crystalline,    [ACTION - transformation]
veins glow icy blue, body develops thick insulating      [ACTION - transformation]
blubber and frost-like scales, eyes become               [ACTION - transformation]
multifaceted like ice crystals, until he becomes a       [ACTION - transformation]
graceful ice-adapted humanoid gliding across the         [ACTION]
surface. Ultra-realistic, slow transformation,           [STYLE]
dramatic lighting.                                       [STYLE]
```
**Breakdown:** 55% action (transformation), 20% scene, 10% actor, 10% style, 5% camera

### Example 4 (439 chars) — First-person, dialogue-heavy
```
First-person perspective,                                [CAMERA]
a girl is tying her shoelaces,                           [ACTOR + ACTION]
suddenly looks up at the camera, and says in shock:      [ACTION]
Wow, you're so handsome, are you a celebrity?            [DIALOGUE]
Then she reaches her hands towards the camera            [ACTION]
and leans in to give a kiss. Subsequently,               [ACTION]
the camera moves back, the girl stands up,               [CAMERA + ACTION]
takes out a phone from her sock, and asks the            [ACTION]
camera if they can exchange contact information.          [DIALOGUE]
The camera shakes its head and then turns and            [CAMERA + ACTION]
walks away.
```
**Breakdown:** 50% action, 20% dialogue, 20% camera, 10% actor

### Example 5 (258 chars) — Multi-shot, tightly structured
```
Cut 1 (2s): Wide crane shot from behind the couple,     [CAMERA + ACTOR]
castle glowing in the misty night.                       [SCENE]
Wind moves their cloaks.                                 [ACTION]
Cut 2 (2s): Slow push-in close-up on their              [CAMERA]
interlocked hands, subtle tremble.                       [ACTION]
Cut 3 (2s): Whip-pan to castle towers as shadows        [CAMERA + SCENE]
move behind lit [truncated]
```
**Breakdown:** 40% camera, 25% scene, 20% action, 15% actor

---

## Summary: The Typical ~369-Char Seedance 2.0 Prompt

> **One continuous shot** describing **an action sequence** in a **specific environment**, with **brief camera direction** and **minimal style tags**. Characters are described functionally (role/clothing), not elaborately. The prompt reads like a mini-screenplay paragraph.

**Formula:** `[Camera] + [Actor in Scene] + [Sequence of 2-4 actions] + [Style tags]`

**Word budget at 369 chars (~60 words):**
- Action: ~30 words
- Scene: ~12 words  
- Camera: ~8 words
- Actor: ~6 words
- Style: ~4 words
