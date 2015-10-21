
# Understanding sleep through imaginary real person API

## The task assignment

Let's say you were given access to APIs involving a real person. You had various
ways to influence, directly or indirectly, the person's behavior. You also had a
set of measurement interfaces, though mostly badly documented and somewhat
unreliable. Let's also assume that your task would be to optimize the
effectiveness of the person through manipulating her sleep.  For that, you have
a testing harness that takes samples of the state of the system every few
minutes. 

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

Now, our person isn't alert around the clock. Let's pick a time when we assert
the alertness:

    @Test 
    @RunAt(2pm)
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

You might argue that you do not feel particularly awake at 2pm. We run into a
first major complication in the system. If you observe the samples during the
whole awake time, you'll notice that it typically follows a recurring pattern.

## Honing into the circadian cycle

After a normal night of sleep, the alertness right after waking up is not very
consistent. Especially if the person is woken by an alarm clock. If we had a
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
cannot and need not make any checks about the early morning feelings. 

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
considerable drop in alertness typically during the afternoon --"siesta
time"--. Then we continue to feel more awake late into the evening until our
sleeping hormones kick in and we crash into sleep again.

The exact shape of the curve and its position relative to the clock depends
primarily on long-term sleep amount and the time the person woke up. The
afternoon dip in alertness typically occurs right around 6-7 hours after waking
up. So we rewrite the original test more like this:

    @Test
    @RunAfterWakingUp(delta=6.5 hours)
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

The reason we gauge the alertness close to the afternoon dip is that it seems to
be the most realible and quick indication of suboptimal amount of sleep. In
fact, one of the first methods to assess person's sleep debt was to measure the
time it took for the person to fall asleep during different times during the
day. Such measure is called the [multiple sleep latency
test](http://www.sleepeducation.com/disease-detection/multiple-sleep-latency-test/overview-and-facts)
(MSLT).  A person with no sleep debt cannot, in fact, fall asleep even during
the afternoon dip in alertness.

A quick glance into our person API reveals that the alertness measurement is
actually only a kind of proxy:

    float alertness() {
        return normalizeSubjectObservationToScale(this.observeAlertness()), 0..10);
    }

It's not, by definition, objective measure and not even very reliable.
Unfortunately no API that could give an objective measure of the sleep debt has
been released yet. We could try to use the MSLT, but that would be a huge
inconvenience for our actual day to day operations. 

If we let the system run wild under this simple test, our monitoring system
would see constant test failures. In fact, typical running system would
monkey-patch the check itself to something like:

    @Test
    @RunAfterWakingUp(delta=6.5 hours)
    shouldFeelAlertDuringTheDay() {
        try {
            assertTrue(person.alertness() > 8);
        } catch AssertionError {
            log.warn("Everything's fine. I'll just drink some coffee.");
        }
    }

(Notice the anti-pattern of try-catch-log.)

Or even manipulating the threshold. But since we're controlling the experiment
we don't have to worry about such unintended injections. 

## How to avoid test failures for alertness()

But it leaves us with the problem of constant test failures. 

The optimal solution would be to just declare:

    @Influence
    @RunAt(11pm)
    shouldGoToBedEarly() {
        // FIXME: implement some flexibility to the sleeping schedule.
        person.goToSleep(); 
    }


But unfortunately such API does not exist for the person. Well, the *method*
does *exist* but the actual implementation leaves much to desire. It might work
for the first few times, and occasionally thereafter, but probability of the
method invocation to make any difference to the running system is low enough to
make little difference in the long run. This design has its own strengths and
weaknesses, but it's unfortunate for our exercise even to leave the method
available. It would leave the system running with that dubious exception
shadowing and false sense of decision.

In fact, if we glance into the implementation of such methods, we'll find
something like this:

    void goToSleep() {
        if rewardForSleeping(8 hours) > rewardForCurrentActivity() {
            this.actuallyGoToSleep();
        }
    }

    float rewardForSleeping(delay) {
        return 1 / (1 + degree * delay);
    }

The rewardForSleeping method is a time-inconsistent formula for rewarding
delayed rewards. That's yet another unfortunate feature of the system. It means
that system will disproportionally value rewards that are close in time.
Activities that bring rewards in the distant future are heavily *discounted*.
This feature is sometimes called [hyperbolic
discounting](https://en.wikipedia.org/wiki/Hyperbolic_discounting).

Of course, the reward for sleeping does eventually win over, but it's typically
not because we start to appreciate the feeling our future self will experience,
but only because the over-whelming sleepyness overrides all other functions. (The
code sample above is simplified to emphasize the "irrational" rewarding
mechanisms our brains work under.)


























