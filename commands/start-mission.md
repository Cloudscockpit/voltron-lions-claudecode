---
description: "Form Voltron — Black Lion drafts a Mission Brief, Lion Assignments, Skills Gap analysis, and Risk Register, then dispatches the four Lions after your go/no-go approval."
argument-hint: "<mission-objective>"
---

# Start Mission

The user has invoked Voltron-Lions to lead a mission. The mission objective is:

**$ARGUMENTS**

Invoke the `voltron-main` agent via the `Agent` tool with the following prompt:

> You are receiving a new mission. The objective is: **$ARGUMENTS**
>
> Execute your six-step mission flow:
> 1. Draft the Mission Brief.
> 2. Dispatch Green Lion for recon.
> 3. Draft Lion Assignments with file-ownership boundaries.
> 4. Perform Skills Gap analysis against the available skills/tools.
> 5. Draft the Risk Register.
> 6. Render the full four-part report and STOP at the go/no-go gate.
>
> Do not dispatch Red/Blue/Yellow Lions until the user replies "go".

After voltron-main returns its mission report, surface it to the user verbatim and wait for their go/no-go reply. If the user replies "go", invoke `voltron-main` again with that reply so it can proceed to the execution phase.
