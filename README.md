[Ticket](https://issuetracker.google.com/issues/287882996)

Steps:

1. Build advanced/end
2. Run app
3. Search for "xxxxxxxxxxxxxxxxxxxxxxxx" or anything that gives **exactly** 1 page of results
4. Observe logs

The logs begin like this:

[Load state logs till buggy state](https://i.imgur.com/olYWmD2.png)

And end like this:

[Load state logs after buggy state](https://i.imgur.com/VbN3cAB.png)

As you'll notice, the load state goes from Loading -> NotLoading -> Loading. Adapter count, on the other hand, stays at 0 but goes on to become non-zero after this blip.

This causes 2 problems:

Seeing a NotLoading state, the progress view goes invisible but then shows up again.
An empty state, if it existed, would do the opposite and go from hidden to shown to hidden.

This puts the Ui in an intermediate incorrect state.
The Ui should be revealed to the user **after** the initial load is done and the adapter has been set.

In this case, however, we reveal it prematurely.

The culprit seems to be [this](https://i.imgur.com/DOkewuj.png) block of code in `RemoteMediatorAccessImpl#launchRefresh`.
Please note that for this particular query `isEndOfPaginationReached` flag in `GithubRemoteMediator` is true, the logic for which differs from the original logic provided in this codelab example.
Originally, the logic was to check `repos.isEmpty()` but the more appropriate check is `repos.size < pageSize`.

With that established, the codeblock in the screen shot above ends up setting all 3 LoadTypes to `COMPLETED` block state. This happens **before** paging source invalidation and therefore the paging data gets emitted **after** the load state update leading to the Ui state we saw above. 

