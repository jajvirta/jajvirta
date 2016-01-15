

# A Day in the Life of a Typical (Solita) Developer

It is Thursday, around 9 am. I arrive at Solita's Tampere office. I feel a
slight relief after a busy morning of herding our kids to school.  I exchange
few quick words with my project team members and hurry to finish
what's left with the localization task I had pending from last day before our
daily status meeting starts.

To re-orientate where I was I run:

    git status
    git log

(Of course I have shortcuts, or git aliases, for these, but they wouldn't make
sense to anyone.)

I have couple of commits that I haven't pushed to the repository and I know
that just `git push`ing will result in rejection, because I haven't integrated
all the commits already in the repository. That's why I run these commands to
re-integrate the repository HEAD:

    git pull --ff-only
    git pull --rebase

After "rebasing", ie. putting my commits on the top of the HEAD, I re-run the
test suite just to make sure the rebase didn't break anything. Unfortunately
our UI test suite takes a bit longer so I'll leave the responsibility of
running the to the deployment pipeline we have on Jenkins CI server. If
anything were to fail, Jenkins will notify our project chat room.





- reintegrate

- csrf support
-- short summary of what it is
-- java implementation
--- understand the previous implementation
-- side-by-side implementation to the perftests
--> get it also done and have a reference impl for it

--> googling & stack overflow'ing



