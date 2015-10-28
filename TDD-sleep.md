
# Understanding sleep through imaginary real person API

## The task assignment

Let's say you were given access to APIs involving a real person. You had
various ways to influence its behavior. You also had some measurement
interfaces, though mostly badly documented and somewhat unreliable.  Let's also
assume that your task is to optimize the effectiveness of the person through
manipulating her sleep. For all this that, you have a testing framework that
takes samples of the state of the system every few minutes. 

## Understanding the system through testing

The first test we might write is to make sure that the person feels alert
during the day:

    @Test 
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

Alertness has been normalized to be a floating point number between 0 and 10,
where 10 means extremely alert and energized and 0 is being as close to asleep
as possible. For first iteration, I've picked an arbitrary threshold value, but
we'll get back to that.

Now, our person isn't awake around the clock. Let's pick a time when we assert
the alertness:

    @Test 
    @RunAt(2pm)
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

You might say that you do not feel particularly alert at 2pm. We run into a
first major complication in the system. If you observe the samples during the
whole awake time, you'll notice that it typically follows a pattern.

## Honing into the circadian cycle

After a normal night of sleep, the alertness right after waking up is not very
consistent. Especially if the person is woken up by an alarm clock. If we had a
more comprehensive API and testing framework we could write a check for the
early morning too:

    @Test
    @RunAfterWakingUp
    @RunIfSleepPhaseRightBeforeWaking({REM, NREM_1, NREM_2})
    shouldFeelOKIfWokenInLightSleep() {
        assertTrue(person.alertness() > 4);
    }

If the sleep phase right before waking up is {NREM_3, NREM_4}, we could not make
any guarantees of the alertness of the system. And since the
RunIfSleepPhaseRightBeforeWaking pre-processor doesn't currently exist, we
cannot and need not make any checks about the early morning feelings. If we
happen to wake the poor guy/girl on the proverbial wrong foot, it'll just take a
while to get her back on track. 

But if we continue prodding the system at different times of the waking hours, a
pattern like this will emerge:

    |             
    |            -
    |           / \          ^
    |    ---   /  |      alertness
    |   /   \_/   \          v
    |  /           |
    | /            |
    ------------------------------
       time of day

Right after waking up, the alertness quickly rises. Then we experience a
considerable drop in alertness typically during the afternoon, at "siesta time".
Then we continue to feel more and more awake late into the evening. Only until
our sleep hormones start to kick in and we crash into sleep again. 

The exact shape of the curve and its position relative to the clock depends
primarily on long-term sleep amount and the time the person woke up. The
afternoon dip in alertness typically occurs right around 6-7 hours after waking
up. So we rewrite the original test like this:

    @Test
    @RunAfterWakingUp(delta=6.5 hours)
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

The reason we gauge the alertness close to the afternoon dip is that it seems to
be the most reliable and quick indication of suboptimal amount of sleep. In
fact, one of the first methods to assess person's sleep debt was to measure the
time it took for the person to fall asleep during different times during the
day. Such measure is called the [multiple sleep latency
test](http://www.sleepeducation.com/disease-detection/multiple-sleep-latency-test/overview-and-facts)
(MSLT).  For an example, a person with zero sleep debt cannot fall asleep even
during the afternoon slump.

## Deeper look into the alertness() measure

A quick glance into our person API reveals that the alertness measurement is
actually only a kind of a proxy:

    float alertness() {
        return normalizeSubjectiveObservationToScale(this.observeAlertness(), 0..10);
    }

It's not, by definition, an objective measure and it's not even a very reliable
measure.  Unfortunately no API that could give an objective measure of the sleep
debt has been released yet. We could try to use the MSLT, but that would be a
huge inconvenience for our actual day to day operations. 

If we let the system run wild under this simple test, our monitoring system
would see constant test failures. In fact, typical running system would
monkey-patch the check itself to something like:

    @Test
    @RunAfterWakingUp(delta=6.5 hours)
    shouldFeelAlertDuringTheDay() {
        try {
            assertTrue(person.alertness() > 8);
        } catch AssertionError {
            log.info("Everything's fine. I'll just drink some coffee.");
        }
    }

(Notice the anti-pattern of try-catch-log.)

Since we're controlling the experiment we don't have to worry about such
unintended injections. 

But it leaves us with the problem of constant test failures. 

## How to avoid test failures for alertness()

The optimal solution would be to just declare:

    @Influence
    @RunAt(11pm)
    shouldGoToBedEarly() {
        // FIXME: implement some flexibility to the sleeping schedule.
        person.goToSleep(); 
    }


But unfortunately such API does not exist for the person. Well, the *method*
does *exist* but the actual implementation leaves much to desire. It might work
for the first few times, and even occasionally after that. But the method turns
into a no-op implementation after just few iterations. It doesn't do anything. 

The implementation of the goToSleep() looks something like this:

    void goToSleep() {
        if rewardFor(sleeping) > rewardFor(this.currentActivity()) {
            this.actuallyGoToSleep();
        }
    }

    float rewardFor(activity) {
        delay = whenRewardedFor(activity);
        return 1 / (1 + degree * delay);
    }

The rewardFor method is what psychologists call a time-inconsistent formula for
giving value to delayed rewards. That's yet another unfortunate feature of the
system. It means that system will disproportionally value rewards that are close
in time.  Activities that bring rewards in the distant future are heavily
*discounted*.  This feature is sometimes called [hyperbolic
discounting](https://en.wikipedia.org/wiki/Hyperbolic_discounting).

Of course, the reward for sleeping does eventually win over, but it's typically
not because we start to appreciate the feeling our future self will experience.
The *delay* in the whenRewardedFor() method actually starts giving extremely low
values, thus increasing the anticipated reward for sleeping. 

## Sounds like a reasonable implementation to me, eh?

Now, this is not necessarily a bad approximation of reasonable boundaries for
the running system. For hundreds of thousands of years, this is how we've
decided. But it's the realities of the modern life (combined with *yet another*
unfortunate feature of the system) that has made the situation more complicated. 

The unfortunate feature is the fact that our circadian cycle isn't exactly 24
hours. It varies individually, but typically it's around 24,5 hours. If we could
go a free-running sleeping cycle, our daily schedule would shift forward half an
hour a day. But because we live in a society with fixed schedules and on a
planet that has a 24 hour day, we have to force our sleeping schedule to the 24
hour day. 




























