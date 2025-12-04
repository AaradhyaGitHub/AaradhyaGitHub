# FlowBook
Digitizing BJJ Sparring Matches

---

## Why BJJ Is Different

Brazilian Jiu-Jitsu is a grappling martial art where a smaller person can beat a bigger one using leverage, technique, and timing. You're essentially solving a human puzzle while someone is actively trying to choke you unconscious.

It's also one of the hardest sports to improve at.

Unlike lifting (where you can track weight and reps) or running (where you have pace and distance), BJJ progress is maddeningly vague. You "feel" like you're getting better. Your coach says you're doing fine. But you can't point to a number that proves it.

Most people plateau for months without knowing why. They keep showing up, rolling hard, going home tired—but nothing changes. They're training, not practicing.

The difference? Feedback loops.

In other sports, feedback is built in. You see the weight go up. You see your mile time drop. In BJJ, you just... remember stuff wrong. You think you're aggressive when you're actually defensive. You think you're losing to submissions when you're actually getting swept first.

The sport desperately needs better self-tracking. But every solution so far has been designed by people who've never tried to journal while catching their breath between rounds.

---

## The Problem

You just finished sparring. You're exhausted, sweaty, probably got choked out once or twice. How do you capture this experience and maybe learn from it?
- Last thing you want to do is pull a piece of pen and paper 
- You can't have a coach watching your every sparring session and giving you detailed feedback.
- Wearable tech in BJJ can be hazard 

So people just... don't track. And then they wonder why they're stuck at the same level for months.

## What FlowBook Does

It's a mobile app that lets you log a sparring session in 30 seconds through a swipeable questionnaire (think Instagram stories but for tracking your rolls).

Here's the flow:
1. Tap the pencil icon
2. Pick: WON 🏆 / DRAW 🤝 / LOST ☠️
3. Swipe through 14 quick questions (mostly sliders and buttons)
4. Done. Your spar is logged.

The questions adapt based on whether you won, lost, or drew. If you can't finish because someone asks you to roll again, we auto-submit after the first 8 questions so you still get partial data.

Then you get:
- A dashboard showing your win streak, performance trends, and a FIFA-style spider graph of your strengths/weaknesses
- A "Pokémon card" profile of your grappling style (shareable on Instagram)
- Insights like "you've been in defensive positions 70% of the time—maybe work on your guard passing?"

## Why This Might Actually Work

**It's designed for fatigued athletes.** Every design decision asks: "would someone who just got armbarred 3 times actually use this?"

The top 4 design elements to reduce friction to log things:
- 30-second timer --> Gamifies the speed.  
- Progress bar --> it feel less daunting.  
- Auto-submit --> Respects your time if you can't finish.  
- Sliders instead of text --> Faster and easier than typing.

**It targets tech workers who do BJJ.** Silicon Valley is obsessed with BJJ right now (Zuckerberg, half of YC founders, etc.). These people actually care about data and are willing to pay for insights.

**It gives you a mirror, not advice.** BJJ is too vague and situational for an app to tell you "do this move." But it CAN show you patterns you're blind to. Like how you thought you were aggressive but actually you've been playing defense 80% of the time.

**It's built by someone who needs it.** I'm a blue belt on paper, high-level white belt in reality. I live this frustration.

## The Project

**Short-term goal: 75 active users.**

Just 75 people from gyms in Davis + Sacramento who log at least once a week for 2+ months.

If people will consistently log their training, then there's something here.

**Medium-term: Make the profile cards shareable.**

The real distribution play isn't ads—it's people posting their "grappler profile" on Instagram. It needs to look cool enough (Robinhood-style minimal UI, clean data viz), people will share it.

Think Spotify Wrapped but for BJJ. "This year I logged 240 spars, hit 47 submissions, and my guard game improved 32%."

**Long-term: Add some intelligence.**

If there are 75 people logging consistently, that's thousands of data points about what works and what doesn't in training.

At that point, possibly:
- Use GPT to generate personalized training suggestions based on your data
- Show you "people with similar stats focus on X and see improvement"
- Identify patterns like "you get swept more when you're tired—your round 4 defense drops 40%"

No need to build a ML model from scratch just yet. Just need clean data and thoughtful prompts to GPT.

**Eventually: Expand to other grappling sports.**

Once this works for BJJ, the same structure applies to wrestling, judo, MMA grappling. It's mostly just different question sets and databases. Not hard technically, just need someone who knows the sport well enough to build the right questions.

## Why I'm Building This Regardless

I could use this and I want to see if insight can be gathered from something as chaotic as two people rolling on mats.
BJJ is messy. The data is vague. But I think there's signal in the noise.


## What It's Not

This isn't  trying to disrupt the combat sports industry. It's not for everyone.

It's for the niche group of grapplers who are analytical about their training, who like data, who want to see if tracking actually helps them improve.

I'm building this to solve my own problem and see if anyone else cares. That's it.

---

## The Technical Stuff

For people who code:

**Stack:**
- React Native (JavaScript)
- Firebase Realtime Database
- Context API for state management
- React Navigation (Stack + Bottom Tabs)

**Current state:** 65% done. Core data collection works end-to-end. Users can log spars, data persists, cards display correctly. Dashboard has basic stats (win streak, trend arrows). No auth yet.

**Architecture:**

Questionnaire system: Instead of a boring form, it's a horizontal carousel where each question is a full-screen slide. Like onboarding flows but for data input.

Each question type (slider, MCQ, numerical input) is a different component rendered based on question metadata. The answer state bubbles up from `QuestionnaireItem` → `Questionnaire` → `SparForm` → Firebase.

Questions are stored in JSON files with this structure:
```javascript
{
  id: "win1",
  question: "How many takedowns did you score?",
  type: "slider",
  options: { min: 0, max: 100 },
  section: "takedown"
}
```

There are 3 question sets (WIN, DRAW, LOSS) that get loaded dynamically based on result selection. Plus 8 generic ROUNDINFO questions that always run first.

The win streak algorithm is simple: sort spars by date descending, count consecutive wins from the top, stop at first non-win. O(n log n) for sort + O(n) for scan.

The trend calculation compares last 7 days vs previous 7 days win percentage to show if you're improving.

**Design decisions:**

Removed 50+ complex questions that required users to search through databases of submissions, positions, and takedowns. Saved those in a `/legacy` folder for the search UI that will be built later.

Current version is just 14 questions per spar (8 ROUNDINFO + 6 result-specific), all simple inputs. Fast to answer, low friction.

Each of the 6 result-specific questions maps to one axis of the spider graph: takedowns, guard, submissions, control, defense, cardio. This keeps the data structure clean.

Dates are stored as ISO strings with timestamps (`2025-11-25T04:02:44.787Z`), not date-only, so sorting actually works properly. Learned this the hard way after spending 1.5 hours debugging why Recent tab showed 0 spars.

**What's left to build:**
- Section headers on slides ("ROUND INFO", "TAKEDOWNS", etc.)
- Progress counter (Question 5/14)
- Spider graph visualization (probably react-native-chart-kit or victory-native)
- Store raw answers in Firebase (for edit mode later)✅Implemented
- User authentication (have a separate login system in another repo, need to integrate) ✅Implemented
- Better rating algorithm (currently just averages 2 sliders, should use 10-15 data points)

**What I'm NOT doing:**
- Building my own ML model (will use GPT API when the time comes)
- Overcomplicated analytics (keep it simple until users ask for more)
- Social features yet (profile sharing comes after core product works)
- Perfecting everything (ship 80% solutions, iterate based on feedback)

The plan is: build the simplest thing that solves the problem, get it in people's hands, learn what actually matters, then iterate. 
I'm treating this like a learning project that might become something more. Not the other way around.

---

**Current status:** Building Session 3. About to add spider graphs and polish the dashboard.

**Timeline:** Finish v1 in next 4-6 weeks. Get 10-20 gym members using it. See if they come back without having to nag them. Then decide if it's worth continuing.

**Repo:** Private for now.
---

If you read this far and you're genuinely interested (not just being polite), hit me up. I'm not looking for feedbacks and helping hands. Just curious if this resonates with anyone else. And if you train BJJ and want to try it when it's ready, let me know. I need real users who will tell me when it sucks.

I appreciate your time. 
- Aaradhya

---
*Last updated: November 2025*
