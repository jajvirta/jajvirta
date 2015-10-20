
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


