# **Module 10: Defensive Security**

_TryHackMe notes - SOC, DFIR, and logs_

Switched gears here which I liked, felt like a completely different brain being used compared to the offensive sections. This is honestly the side of security I actually want to end up working in.

## **Defensive Security Intro**

Basically the blue team version of the intro I'd already had for offensive stuff. Defenders are trying to stop, detect, and respond to the same attacks I'd just been practicing doing.

## **SOC Fundamentals**

Gave me an actual proper picture of what analysts do day to day rather than just a vague idea. Tiered structure:

- **Tier 1** - monitor alerts, triage, decide what's worth escalating
- **Tier 2** - deeper investigation on escalated stuff
- **Tier 3** - threat hunting, more advanced response

Made a lot more sense of the "alert fatigue" problem too, SOC analysts get absolutely buried in alerts and a lot of the job is filtering signal from noise.

## **Digital Forensics Fundamentals**

Really enjoyed this one. Covers the basics of investigating a machine after something's happened, preserving evidence properly, chain of custody, not touching things in a way that changes timestamps or data. Small stuff like this actually matters a lot, an investigation can fall apart if evidence isn't handled properly.

## **Incident Response Fundamentals**

The IR lifecycle stuck with me:

1. Preparation
2. Identification
3. Containment
4. Eradication
5. Recovery
6. Lessons learned

Simple structure but genuinely useful, gives you a checklist to think through instead of panicking when something actually goes wrong.

## **Logs Fundamentals**

A bit dry ngl but you genuinely can't do any of the above without understanding logs first. Windows Event Logs, Sysmon, that kind of thing. Basically the raw material everything else in this module depends on.

## **Stuff worth remembering**

- DFIR and SOC work is a lot more procedure and documentation heavy than offensive work, which I actually like
- Logs are boring until the one time they're the only thing that tells you what actually happened
- This module made me more sure DFIR is the direction I want to specialise in
