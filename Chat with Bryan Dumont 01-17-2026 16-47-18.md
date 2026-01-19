We’re in the middle of live fieldwork across multiple "PRISM" studies and have paused fielding briefly to avoid wasting sample. PRISM is segmentation model built for healthcare public affairs/policy/advocacy - it's a continuous research system built around a shared “typing tool” that assigns respondents into a fixed set of segments (~16). Those segments are stable across studies. Lots more we need to discuss about this after we get through the current chalenge.

Individual projects (currently studies on Medicare Advantage, employer sponsored insurance tax exclusion, GLP-1 compounding; with studies coming on line next week around phrama industry investment in American manufacturing, childhood vaccine schedule, the vaccine injury compensation program, and vaccination for new and pregnant parents...). All of these are not standalone surveys — they are instances that draw from the same typed population but prioritize different subsets of segments at different times.

As fielding progresses, segment priorities shift based on estimated "audience ROI" and analytic needs (based on how likely they are to be persuadable and activated on the issue addressed in each study). This means quota logic must adapt dynamically, and respondents who are not needed for one study are often valuable for another.

The Current problem: Studies are currently fielded separately, so respondents are terminated when over-quota in one study even though they could fill open quotas in another. With four studies running concurrently next week, this is causing major sample waste.  
Here's the ask: Design and implement (write the xml code) a minimal, safe routing structure so that all incoming sample passes through a single typing event and is then assigned to one of four studies based on open segment priorities, instead of being terminated. We are not building the final router. We need the simplest safe version that can be live next week.

I suspect we have two options: Option 1: A pre-survey “router” module that assigns a study and then branches to the other surveys, or, Option 2: One survey shell with four internal versions (messier but probably more realistic given time pressure.

=================================

Jeremy Schwab5:12 PM

Got it. This is very feasible to stabilize quickly.

Given time pressure and live risk, I recommend Option 2: a single survey shell with internal versions. It avoids cross-survey redirects, keeps panel flow intact, and lets us control quotas and priorities centrally in Decipher with minimal surface area for failure.

Proposed approach (safe/minimal):  
- One entry point -> one typing event (shared PRISM logic).  
- Compute segment + priority score once.  
- Lightweight allocator that assigns a `study_id` based on:  
- Open quotas by segment  
- Study-level priority weights (hard-coded for v1)  
- Deterministic tie-breaking.  
- Route internally to the correct study block/version.  
- If no valid study is open for that segment, hold or soft-term (not hard terminate).  
- Log segment, study assignment, and decision path for QA.

This can be implemented as a clean XML module that we can drop into the existing studies with minimal refactor and then evolve into the full router later.

I can start tonight. First step would be reviewing current typing outputs, segment quotas per study, and panel entry/exit constraints so I don't break live traffic.

=========================================

Bryan Dumont5:18 PM

I was going to find a way to share the xml for both of the studies in the field. Each one is about 5,000 lines of code (so bloated, not annotated at all). Is that too much to start with? I can send you the latest quotas, they are being manually adjusted on the fly - but are close to locked now.

==========================================

Jeremy Schwab5:21 PM

That's fine - I don't need to digest all of it up front.

To move fast and safely, I'll focus on:  
- The typing logic + outputs  
- Current quota variables / segment caps  
- Entry, termination, and completion logic

You can send the XMLs as-is. I'll treat them as reference, not something to refactor wholesale. The latest quota sheet will be very helpful, especially if priorities are now mostly stable.

Once I see those pieces, I can stub a clean router module tonight and we can wire it in without touching most of the existing code.

============================================

Bryan Dumont5:23 PM

would a sharebale Git gist work?

PRISM Segmentation Typing Tools.xlsx 

124 kB

2 files 

30 kB

38 kB

[https://gist.github.com/travlrdc1982/ef0117ee4207b1f2b6aa7415fa822fd0](https://gist.github.com/travlrdc1982/ef0117ee4207b1f2b6aa7415fa822fd0)  
[https://gist.github.com/travlrdc1982/ea144aa0be3738424f1c0b0994ca5e06](https://gist.github.com/travlrdc1982/ea144aa0be3738424f1c0b0994ca5e06)  
[https://gist.github.com/travlrdc1982/a0c226a11d1ce38e16e4212cf7f37651](https://gist.github.com/travlrdc1982/a0c226a11d1ce38e16e4212cf7f37651)

=============================================

Jeremy Schwab5:39 PM

Yes - that works perfectly.  
The gists plus the typing workbook and quota files are exactly what I need. I'll pull those now, review the typing outputs and quota structure, and start sketching a minimal router that can safely go live.  
I'll follow up shortly with a concrete recommendation on the router approach and what I'll implement first.

===============================================

This is a strong structure and absolutely workable for a minimal safe router. A few important clarifications and tweaks before I implement:  
- Step 2 -> Step 3 dependency: Insurance classification should happen after typing so routing can consider segment x insurance combinations (e.g., MA-eligible Progressives prioritized into MA). I'll make that dependency explicit in the router logic.

- Step 4 eligibility flags: These should be computed dynamically from segment + insurance + current study availability, not treated as static gates. A respondent can be technically eligible for MA but routed to ESI if MA is full and ESI has higher segment headroom. The flags are great for QA and logging; the router decision should override them.

- Step 5 router logic: We should formalize this as:  
1. Primary: segment quota headroom by study  
2. Tiebreaker: study priority weight (e.g., MA > ESI > GLP-1 this week)  
3. Secondary tiebreaker: deterministic rule (e.g., respondent ID mod N)  
4. Logging: Persist `STUDY_ASSIGNED`, evaluated eligibility flags, quotas checked, and decision path to hidden vars for QA/debugging.

One key clarification before I build: are segment definitions fully stable across studies, or do thresholds/logic shift by study context?

- If stable -> single `SEG_PRIMARY` works.  
- If not -> we'll need study-keyed segment outputs or a context-aware typing pass.

Once that's confirmed, I can move straight into the XML module.

============================================

Bryan Dumont9:40 PM

Hey sorry to keep you waiting... - on the insurance classification - you are correct. There are essentially two studies built into one with MA/ESI instrument and every respondent only gets one. We set quotas to prioritize those who actually have medicare advantage to get the MA questions/maxdiff and those who have insurance through employer to get the questions about employer-sponsored insurance.

Step 4 - you 've got it  
Step 5- sounds good  
Segment definitions are stable, but the actual variable is "XSEG_ASSIGNED'

==============================================

Jeremy Schwab9:44 PM

Got it - thanks for the clarification.

So the flow will be:  
- Typing assigns `XSEG_ASSIGNED` (stable across all studies)  
- Insurance classification then hard-routes respondents into either MA or ESI within the combined instrument  
- Segment quotas act as the gating and prioritization layer inside that decision  
- Other studies (GLP1 / Pulse) can be evaluated downstream once MA/ESI eligibility is resolved

That keeps the router clean and deterministic, which is exactly what we want under live-field pressure.

I'll proceed assuming:  
- MA/ESI is exclusive and evaluated first  
- Segment definitions do not vary by study  
- Routing fallbacks are allowed to send respondents to another active study rather than terminate, when segment priorities permit

I can start implementing the minimal XML router now and will focus on safety, logging, and reversibility so nothing burns sample.

================================================

Bryan Dumont
9:50 PM
and if all quotas are filled - they need to be systematically tagged for the segment they typed into so we can build up the panel and also for the modeling we're doing to score voter/media files

================================================

Jeremy Schwab
9:55 PM
Understood. That fits cleanly.

When everything is full, I'll tag, not burn:
- Persist XSEG_ASSIGNED
- Set a clear router status (e.g., ROUTER_STATUS=OVERQUOTA_TAGGED)
- Exit cleanly so the record is usable for panel growth and downstream modeling/scoring
- Log that they qualified but were held due to quota saturation

Before I wire it, three quick confirmations:
1. Preferred overquota exit behavior in Decipher (specific term code vs redirect w/ params)?
2. If MA and ESI are both full, should the respondent be tagged to the would-have-gone study (priority-based) or a neutral overquota bucket?
3. Should I read live quota state from Decipher quota sheets, or assume manually updated hard-coded values for now?

Once I have that, I'll lock the router logic and implement.

===============================================

Bryan Dumont
10:01 PM
1. not sure... except we should definitely send them off very politely!, 2. not sure again ... welcome your counsel; 3. reading live quota would be better. our goal moving forwda is not only reading them but changing them in real-time

===============================================

Jeremy Schwab
10:09 PM
Thanks - here's a concrete plan I recommend:
1. Overquota exit: Use a soft termination with a polite close. Persist XSEG_ASSIGNED and set ROUTER_STATUS=OVERQUOTA_TAGGED. If you want a panel redirect later, we can append params like status=overquota_tagged&seg=XSEG_ASSIGNED.
2. MA vs ESI when both are full: I suggest keeping a STUDY_INTENT variable (MA or ESI based on the same priority rule) plus ROUTER_STATUS=OVERQUOTA_TAGGED. That preserves which instrument would have taken them, while still giving you clean segment tagging for panel build and modeling.
3. Quotas: I'll read the Decipher quota sheet live. That gives us real-time gating now, and sets us up cleanly for phase two where quotas/weights can be adjusted programmatically in-field. For this week, manual quota-sheet edits are fine.

If you're good with that, I'll implement the router with these defaults and proceed.