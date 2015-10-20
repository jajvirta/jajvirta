
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
consistent. Especially if the person is woken by an alarm clock.  But since our
testing framework comes with a lot of handy preprocessors, we can make sure that
if we wake up in the right sleep phase, we should feel pretty ok: 

    @Test
    @RunAfterWakingUp
    @RunIfSleepPhaseRightBeforeWaking({REM, NREM_1, NREM_2})
    shouldFeelOKIfWokenInLightSleep() {
        assertTrue(person.alertness() > 4);
    }

If it so happens that our person does wake after certain deep sleep stages, we
have to make sure 

    @Influence
    @RunAfterWakingUp
    @RunIfSleepPhaseRightBeforeWaking({NREM_3, NREM_4}) 
    shouldTakeSomeTimeToComeBacktoLife() {
        person.drink(strongCoffee);
        person.breathe(slowly);
    }


If we continue prodding the system like this, a pattern like this will emerge:

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
considerable drop in alertness typically during the afternoon --at "siesta
time"--. Then we continue to feel more awake late into the evening until our
sleeping hormones kick and we crash into sleep again.

The exact shape and position relative to the clock of the curve depends
primarily on long-term sleep amount and the time the person woke up. The
afternoon dip in alertness typically occurs right around 6-7 hours after waking
up. So we rewrite the original test more like this:

    @Test
    @RunAfterWakingUp(delta=6.5 hours)
    shouldFeelAlertDuringTheDay() {
        assertTrue(person.alertness() > 8);
    }




