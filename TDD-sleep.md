
Let's say you were given access to APIs involving a real person. You had
various ways to influence, directly or indirectly, the person's behavior. You
also had a set of measurement interfaces, though mostly badly documented and
somewhat unreliable. On top of that, the system itself, --the person--, is a
complex legacy system that is for all practical purposes impossible to change.

Let's also assume that your task would be to optimize the effectiveness of the
person through manipulating her sleep. 

We have a testing harness that takes samples of the state of the system every
few minutes. 

The first test we might write is to make sure that the person feels alert
during the day:

    @Test 
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

Alertness has been normalized to be a floating point number between 0 and 10,
where 10 extremely alert and energized and 0 is being basically asleep. I've
picked an arbitrary threshold value, but we'll get back to that.

Now, our imaginary person isn't alert around the clock. Let's pick a time when
we assert the alertness:

    @Test 
    @RunOnlyAt(2pm)
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }

You might argue that you do not feel particularly awake just a 2pm. We run into
a first major complication in the system. If you observe the samples during the
whole awake time, you'll notice that it typically follows a recurring pattern.
After a normal night of sleep, the alertness right after waking up is not very
consistent. Especially if the person is woken by an alarm clock. 

But since our testing framework comes with a lot of handy preprocessors, we can
make sure that if the 

    @Test
    @RunAfterWakingUp
    @RunIfSleepPhaseRightBeforeWaking(phases=REM, NREM_1, NREM_2)
    shouldFeelOKIfWokenInLightSleep() {
        assertTrue(person.alertness() > 4);
    }

If we continue prodding the system like this, the pattern will emerge quite like
this:

    |             
    |            -
    |           / \          ^
    |    ---   /  |      alertness
    |   /   \_/   \          v
    |  /           |
    | /            |
    --------------------------------------------------
       time of day









