# Ticket #670659: Can't log in (simulated)

A simulated help desk ticket worked through in the osTicket system I deployed. The ticket was created, triaged,
prioritized, and assigned in osTicket. The replies and resolution below are drafted as written examples,
because replying in the tool failed with an "unable to lock ticket" error and there is no mail server or Active 
Directory in this lab. The fix describes standard procedure, not steps I performed.

| Field | Value |
|---|---|
| Ticket number | 670659 |
| Subject | Can't log in |
| Priority | High |
| Due date | 2:00 PM |
| Assigned to | homelab bill |

## 1. Intake (done in osTicket)

Created from the client portal, written the way a non-technical user would describe it:

> Hi, I just got back from vacation and my computer won't let me log in. It says my password is wrong but I'm
> typing it the same way I always do. I have a client meeting at 2 and need my files.

## 2. Triage (done in osTicket)

- **Priority: High.** The user has a hard deadline and cannot work, so it should not sit at the default priority.
- **Not Emergency.** It affects one user and one account, with no outage affecting other people or systems.
- **Due date: 2:00 PM**, matching her meeting.
- **Assigned** to an agent in the staff control panel.

## 3. Clarifying reply (drafted)

> Hi Dana, sorry you're locked out right before your meeting. To get you back in quickly, could you tell me:
> 1. The exact message you see on screen
> 2. Whether you changed your password or got a password email before you left
> 3. Whether you can sign in to anything else, like email on your phone
>
> Thanks, IT Help Desk

Three short questions only, because long lists frustrate users who are in a hurry.

## 4. User response (simulated)

> It says "Your account has been locked out." I didn't change anything. My phone email stopped working too, yesterday.

## 5. Internal note (drafted, visible to agents only)

- **Symptoms:** User cannot log in after a week of vacation. The screen says the account is locked out.
- Phone email also stopped working yesterday.
- **Likely root cause:** Account lockout. Her phone email failing too points to the account itself, not her
-  computer. A phone repeatedly retrying an old or expired password while she was away would cause failed sign-ins
-  and a lockout. A password expiring during her leave is the other likely cause.
- **Fix (standard procedure in an Active Directory environment):**
  1. **Verify identity first.** Anyone could write in claiming to be Dana. Confirm with a callback to the
  2. number on file, an employee ID check, or manager approval, per policy.
  3. Unlock the account in Active Directory Users and Computers.
  4. Reset to a temporary password with "user must change password at next logon," delivered through a
  5.  secure channel and never written in the ticket.
  6. Tell her to update the saved password on her phone so it doesn't lock her out again.
- **Verification:** Dana signs in successfully and her phone email syncs.
- **Time spent:** about 10 minutes (estimated)

The internal note exists so other agents can see the troubleshooting trail without the user seeing it.
A reply, by contrast, is the message the user receives.

## 6. Resolution reply (drafted)

> Hi Dana, your account was locked after repeated sign-in attempts while you were away, most likely
>  from your phone still using your old password. I've unlocked it and set a temporary password,
>  which I sent to you separately. Please sign in, choose a new password when prompted,
> and then update the password on your phone. If anything still doesn't work, reply here or
> call the help desk and we'll sort it out right away.

## 7. Closure

Status changed to Resolved after confirming the user could sign in.

## What this ticket demonstrates

- Turning a vague complaint into a diagnosis with a few targeted questions
- Prioritizing based on impact and deadline, and explaining why
- Verifying identity before unlocking or resetting an account
- Never putting a password in a ticket
- Writing a technical internal note separately from a plain-language reply to the user
