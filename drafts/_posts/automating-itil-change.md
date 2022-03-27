---
title: Automating ITIL Change
---
ITIL is a standard set of IT service definitions and best practices that have been adopted by many organisations. How an organisation implements ITIL processes is up to them and this can vary significantly. ITIL is generally considered the antithesis  of Agile, which prefers speed above all else, but in theory (from my perspective currently) there are ways to continue to utilise the and conform with ITIL processes while working in an Agile way and permitting a culture of Continuous Deployment. The most significant process that we need to consider is how we manage Change.

Change Management is attractive because it offers to bring order to what is typically an anarchic IT organisation. Prior to implementing Change Management, many organisations suffer with rampant uncontrolled change which brings with it the risk of long periods of disruption. Change Management provides a somewhat solution to this by insisting all changes are tracked and reviewed. This has one excellent benefit if adhered to by all participants which is that all changes are at least recorded. As such when disruption occurs its at least simple to determine why and downtime as a result is generally reduced.

Unfortunately the downside to this is that changes generally take longer to occur. Even in its simplest form, the act of the process slows down the engineers ability to make changes. To the extreme, this can be by a factor of days, where changes may need to go via multiple individuals for approval or be presented as a CAB (Change Approval Board) meeting for review. This is sometimes suitable particularly for complex changes and as long as testing has been possible, not a huge inconvenience. However, if you're not able to fully and realistically test your changes, the change process (however small) delays implementation, particularly as each change failure results in more process.

So, ITIL isn't going way. Agile and Continuous Deployment shouldn't be ignored. What's the answer?

## Standard Changes

ITIL already considers a method for expediting certain changes through the definition of a "standard" change. This allows for a change to be recorded but pre-approved, so eliminates the delay that comes with change approvals. However, typically it's still a manual process for an engineer to record that they are performing the change (which standard change they are performing, to which system/customer/product, at what time etc.). And manual processes come with the usual risks:

- They take time and interrupt flow.
- They might not be followed completely (resulting in inaccuracy).
- They might not be followed at all.

**What if you could automate change?**

The act of recording Standard changes is just begging to be automated. There's options for how to achieve this:

- If the change itself is code driven, have the code record the change as a first step. This would require wherever the code is being executed to be able to communicate with the change system, but otherwise we have all the information we need: who's executing the change? (who's running the script) when is it occurring? (right now -- or at a scheduled time, which spawns a scheduled task).
- If the change is not code driven, then we could execute a separate script as part of the process. That would be particularly possible if the process is in an online system where clicking a link could cause code to be executed. At least that then doesn't break flow.