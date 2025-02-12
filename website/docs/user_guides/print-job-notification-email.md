---
id: print-job-notification-email
title: What's in the email you receive when a print job is done
---

Unless you turned the notifications off, you will receive an email when a print job is done that looks like this:

![](/img/user_guides/print-notification-email-sample.png)

Let's take a look at what each of these paragraphs mean.

## 1. Failure detection

If The Detective found something during this print, you will see:

<p className="text--danger">The Detective found possible spaghetti for this print.</p>

Otherwise, you will see:

<p className="text--success">The Detective found nothing fishy for this print.</p>

## 2. Feedback

Should be quite self-explanatory. This is where you've a chance to [give The Detective feedback to help her get better](/docs/how-does-credits-work/).

:::note
Please note [some times there won't be the Focused Feedback link in the email](/docs/how-does-credits-work/#why-is-the-focused-feedback-button-missing-from-some-of-my-prints).
:::

## 3. Print job info and Detective Hours

:::info
Learn more about [how Detective Hour works](/docs/how-does-detective-hour-work).
:::

### 3.1 Print time

The duration between the time when the print job started and the time when it ended or was cancelled.

:::note
**This print time is NOT the amount of Detective Hours you spent on this print. See 3.2 for more details.**
:::

### 3.2 Time watched by The Detective

This is the amount of time that The Detective actually spent on watching your print. This is also the amount deducted from your Detective Hours balance.

Time watched by The Detective can be shorter than the print time because of a few reasons:

* You have [turned off "Watch for failures"](/docs/detective-not-watching/#2-you-turned-off-watch-for-failures-option) in the app.
* There were periods when your OctoPrint lost Internet connection and sent nothing to The Detective.
* The print was paused during the print. When a print is paused, The Detective will also pause watching and hence won't clock her time.
* You have chosen "Don't alert me again" in case of a false alarm.

### 3.3 Your remaining Detective Hour balance

The amount of Detective Hours left in your account *after the amount in 3.2 was deducted from the balance*.

:::note
Detective Hour balance may be a negative number. [Learn why this is the case](/docs/how-does-detective-hour-work/#hey-my-dh-balance-shows-a-negative-number-is-the-detective-out-of-her-mind).
:::

## FAQs

#### Why did I receive this email when I kicked off another print, not when this print was done?

This is because when your OctoPrint was not connected to the Internet at the time this print was done. This can happen when:

* The Internet connection happened to be down.
* You turned off the power before OctoPrint had performed all the "routines" to finish up a print.

When this happens, The Spaghetti Detective won't have a chance to send the "done signal" to the server. Hence the server wouldn't know this print is already done until it receives the signal that indicates "a new print has started". At that time, the server will determine the previous print is already done and send out this notification email.

#### Hey the print time is sometimes a lot longer than the actual print time! Why is it the case?

For the same reason as the previous question. When the "done signal" is missed and the server doesn't know a print is done until the next print starts, the server won't know the actual time when the previous print was done. So out of the lack of a better option, the server will use the time when the next print starts as the time when the previous print finishes.

Rest assured this will NOT cost you more Detective Hours. This is because The Detective will not clock her time when she is not hearing from your OctoPrint. When your OctoPrint is powered off or disconnected to the Internet, The Detective is out of job and hence won't bill you for Detective Hours.

#### Why is my DH balance a negative number?

Here is [why](/docs/how-does-detective-hour-work/#hey-my-dh-balance-shows-a-negative-number-is-the-detective-out-of-her-mind).