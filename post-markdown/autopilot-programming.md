## How autopilot programming nearly brought down a build server

Welcome to the new year! I originally bought this domain to use for some personal projects instead of using hardcoded IPs. Eventually, I figured that now that I have it, I might as well chuck something up here to look at. This eventually evolved into me wanting to keep some sort of a blog or journal to write about whatever comes to mind. This all took place over the last few months. I've been struggling for a while to find a story worth posting about for my first blog post, but a recent experience left me with the need to write about it, so I thought this would make a fun initial entry.

After spending over a year working on a project at my current job, I was tired of looking at the same code day in, day out, and asked my manager for a bit of a change in scenery.

I got the opportunity to start on a new team working with something I hadn't worked with before.

I got started on my first task. I won't go into the details as it isn't relevant to the story. However, after finishing with the new feature, I was ready to start writing some tests.

If you're a subscriber to TDD, you'll know that that one should normally start with the test cases. In this case, I elected to make the tests after the fact.

In my previous project, I'd been working with XUnit as our testing suite, however this new (to me) project was using NUnit. Not that it is any sort of an excuse, as what happened next could have happened just as easily in XUnit, and the blame lies solely with me.

We are working with events, and I had defined a message with about 20 fields. At this stage, I was writing some tests for the event validation logic.

While this is a fairly trivial task, I had a quick look at how the existing tests were written just to ensure my own tests would remain consistent with the existing tests on the project.

I noticed that on the other validation tests, we were passing parameters into the tests decorated with "Value" attributes, which we would use to create the event being validated. This allowed us to write a single test, that would be executed multiple times with each combination of possible values on the fields, to ensure validation would fail correctly for any mix of invalid entries.

I'd like to say that I was programming on autopilot, and didn't seriously consider what this would mean performance-wise on my own validation test for the an event with 20 fields.

Abiding to the standard set by the previously existing tests, I wrote a single test using the "Value" attribute on each of the required fields in the message to ensure that validation would fail if any of the required fields were null or empty.

Like any good developer, I commit frequently. By force of habit I also push to remote, just in case a freak accident was to take out my system. I wouldn't want to lose all my progress, after all.

It was when I tried running my tests locally that I first noticed something was wrong.
For some reason, the tests weren't running. They weren't exactly failing, just sitting there thinking forever.

I started running my new tests one by one. They were all passing just fine, except for the test on validation.

At this point, this was enough to snap me out of autopilot. The issue became immediately obvious. My single validation test was checking every possible combination of my twenty required event fields, with two states each (null, empty).

In case it isn't obvious, the test was executing for every combination of null or empty on all 20 of the required fields. This would total 1,048,576 executions.

Because of course it would. _That's what I told it to do_. And it took a **long** time.

I facepalmed myself, and removed the problematic test to instead rewrite a better test for checking the event validation (which as of writing this, I still have not worked out the best way to handle this).

I re-ran the tests, all green, all is well.

Except it wasn't. See, as I previously mentioned, I commit often, and have a habit of also pushing my commits.

We have an automatic build pipeline on all of our projects at work that runs all the tests as part of the build process.

I quickly jumped onto the dashboard for our build server (we use TeamCity). Sure enough, there was my branch, a few minutes into the build process, happily chugging away on test number thirty-thousand-and-something.

It was just past 16:00 on a friday afternoon, and I was ready to sign off for the week. But I didn't want to leave this build running. I pressed stop on the build, hoping that would be enough to stop this de-railed train. "Build stopped by You", it now showed over the output log. But the tests didn't stop. I waited a few seconds. And then a few minutes. Nope, it was determined to continue.

In the end, I had to get someone else on the team to go in and kill the build process (again, I'm new to this team, so don't have direct access to the build servers).

By the time it finally came to a halt, it showed that 70,199 tests had passed... talk about test coverage!
